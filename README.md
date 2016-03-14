To install the uv4l driver, open the terminal and run the following commands:

wget http://www.linux-projects.org/listing/uv4l_repo/lrkey.asc && sudo apt-key add ./lrkey.asc

Add the following line to the file /etc/apt/sources.list :

sudo nano /etc/apt/sources.list

deb http://www.linux-projects.org/listing/uv4l_repo/raspbian/ wheezy main

sudo apt-get update upgrade

sudo apt-get install uv4l-raspicam-extras uv4l-server uv4l-uvc uv4l-xscreen uv4l-mjpegstream

OR

$ sudo apt-get install uv4l uv4l-raspicam
$ sudo apt-get install uv4l-raspicam-extras
$ sudo apt-get install uv4l-server
$ sudo apt-get install uv4l-uvc
$ sudo apt-get install uv4l-xscreen
$ sudo apt-get install uv4l-mjpegstream

$ sudo reboot

```bash
#### Run the camera
(Optional)
sudo pkill uv4l 

sudo uv4l -nopreview --auto-video_nr --driver raspicam --encoding mjpeg --width 640 --height 480 --rotation 90 --framerate 2 --server-option '--port=9090' --server-option '--max-queued-connections=10' --server-option '--max-streams=2' --server-option '--max-threads=10'

Notes:

The --port=9090 is the local IP port. You can use any port you like.

The --max-streams=25 is the maximum simultaneous streams.

/etc/init.d/isaaccam

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
    $UV4L -nopreview --auto-video_nr --driver raspicam --encoding mjpeg --width 340 --height 420 --rotation 90 --framerate 5 --server-option '--port=9090' --server-option '--max-queued-connections=5' --server-option '--max-streams=5' --server-option '--max-threads=10'    RET=$?
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

https://www.debian-administration.org/article/28/Making_scripts_run_at_boot_time_with_Debian

update-rc.d isaaccam defaults
update-rc.d -f isaaccam remove

