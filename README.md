# To stream video from a Raspberry Pi + Pi Camera using UV4L

## Install the drive onto the Pi

To install the uv4l driver, open the terminal and run the following commands:
```bash
wget http://www.linux-projects.org/listing/uv4l_repo/lrkey.asc && sudo apt-key add ./lrkey.asc
```
Add the following line to the file */etc/apt/sources.list*:

```bash
sudo nano /etc/apt/sources.list
```
Enter the following;
```bash
deb http://www.linux-projects.org/listing/uv4l_repo/raspbian/ wheezy main
```
```bash
sudo apt-get update upgrade
```
```bash
sudo apt-get install uv4l-raspicam-extras uv4l-server uv4l-uvc uv4l-xscreen uv4l-mjpegstream
sudo reboot
```

## Run the camera

_(Optional)_
```bash
sudo pkill uv4l 
```
```bash
sudo uv4l -nopreview --auto-video_nr --driver raspicam --encoding mjpeg --width 640 --height 480 --rotation 90 --framerate 2 --server-option '--port=9090' --server-option '--max-queued-connections=10' --server-option '--max-streams=2' --server-option '--max-threads=10'
```

*Notes:*

* --port=9090 is the local IP port. You can use any port you like.
* --max-streams=25 is the maximum simultaneous streams.

## Ensure the service is run on Pi startup (Jessie uses Systemd!)
```bash
/etc/init.d/isaaccam
```

### init.d
```bash
#!/bin/bash

NAME=uv4l
UV4L=/usr/bin/uv4l
FIND_PID="ps -eo pid,args | grep uv4l | grep '\--device-id $2' | awk '{print \$1}'"

RET=0

kill_pid () {
    pkill uv4l
    PID=$(eval $FIND_PID)
    if [ ! -z $PID ]; then
        kill $PID
        sleep 3
        PID=$(eval $FIND_PID)
        if [ ! -z $PID ]; then
            kill -9 $PID
        fi
    fi
}

case "$1" in
  start|add)
    kill_pid
    $UV4L -nopreview --auto-video_nr --driver raspicam --encoding mjpeg --width 340 --height 420 --rotation 90 --framerate 5 --server-option '--port=9090' --server-option '--max-queued-connections=5' --server-option '--max-streams=5' --server-option '--max-threads=10'    
    RET=$?
    ;;
  stop|remove)
    kill_pid
    RET=$?
    ;;
  *)
    echo "Usage: /etc/init.d/$NAME {add|remove} vid:pid"
    RET=1
    ;;
esac
exit $RET
```

[Debian init.d scripts howto](https://www.debian-administration.org/article/28/Making_scripts_run_at_boot_time_with_Debian)

```bash
update-rc.d isaaccam defaults
update-rc.d -f isaaccam remove
```

### systemd install

```bash
sudo nano /etc/systemd/system/isaaccam.service
```

```bash
[Unit]
Description=uv4l remote server
After=sshd.service

[Service]
Type=dbus
ExecStart=/usr/bin/uv4l -nopreview --auto-video_nr --driver raspicam --encoding mjpeg --width 340 --height 420 --rotation 90 --framerate 5 --server-option '--port=9090' --server-option '--max-queued-connections=5' --server-option '--max-streams=5' --server-option '--max-threads=10'
User=pi
Type=forking

[Install]
WantedBy=multi-user.target
```

```bash
sudo chown root:root /etc/systemd/system/isaaccam.service
sudo chmod 755 /etc/systemd/system/isaaccam.service
sudo systemctl start isaaccam.service
```

