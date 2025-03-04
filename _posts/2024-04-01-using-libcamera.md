 
# Options

Enable debug logs:
```
LIBCAMERA_LOG_FILE=<path-to-log-file> LIBCAMERA_LOG_LEVELS=V4L2Compat:DEBUG ...
```

## Install

```shell
$ sudo apt install rpicam-apps libcamera-v4l2 libcamera-tools
```

## Using

Basic testing:
```shell
$ LIBCAMERA_LOG_LEVELS=*:DEBUG cam -l
```

List available cameras:
```shell
$ libcamera-hello --list-cameras
Available cameras
-----------------
0 : imx708 [4608x2592 10-bit RGGB] (/base/soc/i2c0mux/i2c@1/imx708@1a)
    Modes: 'SRGGB10_CSI2P' : 1536x864 [120.13 fps - (768, 432)/3072x1728 crop]
                             2304x1296 [56.03 fps - (0, 0)/4608x2592 crop]
                             4608x2592 [14.35 fps - (0, 0)/4608x2592 crop]

              S <Bayer order> <Bit-depth>_<Optional packing> : <Resolution list>
SRGGB10_CSI2P S     RGGB          10            CSI2P                 ...
```

Load an application with libcamera V4L2 compatibility layer preload:
```shell
# List devices
$ libcamerify v4l2-ctl --list-devices
# List formats
$ libcamerify v4l2-ctl --list-formats
```

Getting simple JPEG images:
```shell
$ libcamera-jpeg --immediate -o test.jpg
```

Getting JPEG images with exposure compensation (using internal AEC/AGC algorithms):
```shell
$ libcamera-jpeg --immediate --ev -0.5 -o darker.jpg
$ libcamera-jpeg --immediate --ev 0 -o normal.jpg
$ libcamera-jpeg --immediate --ev 0.5 -o brighter.jpg
```

Getting JPEG image with digital gain:
```shell
$ libcamera-jpeg --immediate -o gain.jpg --gain 1.5
```

Getting RAW images:
```shell
$ libcamera-still --immediate -r -o raw.jpg
```

Getting unpackaged video stream with 3 seconds duration (H.264 encoder):
```shell
$ libcamera-vid -n -t 3000 -o test.h264
$ vlc test.h264
```

Getting unpackaged video stream with timestamps for fither encoding:
```shell
$ libcamera-vid -n -t 3000 -o test.h264 --save-pts
$ mkvmerge -o test.mkv --timecodes 0:timestamps.txt test.h264
```

Getting video stream in motion JPEG format:
```shell
$ libcamera-vid -n -t 3000 --codec mjpeg -o test.mjpeg
```

Breaks output video into segment size (given in millis):
```shell
$ libcamera-vid -n -t 3000 --codec mjpeg --segment 100 -o test%05d.jpeg
```

Streaming video to a window (V4L2 compatibility layer):
```shell
# Get video stream (planar 4:2:0 YUV)
$ libcamerify gst-launch-1.0 v4l2src device=/dev/video0 ! "video/x-raw,format=(string)I420,width=(int)800,height=(int)600" ! videoconvert ! xvimagesink sync=false
# Get video stream (reverse RGB packed into 24 bits without padding)
$ libcamerify gst-launch-1.0 v4l2src device=/dev/video0 ! "video/x-raw,format=(string)BGR,width=(int)800,height=(int)600" ! videoconvert ! xvimagesink sync=false
```

Streaming video to a window (using libcamera plugin):
```shell
$ gst-launch-1.0 libcamerasrc ! videoconvert ! 'video/x-raw,format=I420,width=1280,height=720' ! queue ! autovideosink sync=false
```

Streaming video (UDP+libcamera):
```shell
# on server side (RPi3)
$ libcamera-vid -n -t 0 --inline -o udp://<dst-ip-addr>:<port>
# on client side (host)
$ ffplay udp://192.168.1.36:<port> -fflags nobuffer -flags low_delay -framedrop

or

# Server side
$ gst-launch-1.0 libcamerasrc ! "video/x-raw,width=1280,height=720,format=NV12" ! v4l2convert ! v4l2h264enc extra-controls="controls,repeat_sequence_header=1" ! "video/x-h264,profile=(string)main,level=(string)4" ! h264parse ! rtph264pay ! udpsink host=<ip-address> port=5000
# Client side
$ gst-launch-1.0 udpsrc port=5000 caps=application/x-rtp ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink

or

# Server side
$ libcamera-vid -t 0 --width 1280 --height 720 -n --inline -o - | gst-launch-1.0 fdsrc fd=0 ! h264parse ! rtph264pay ! udpsink host=<ip-address> port=5000
# Cliend side
$ gst-launch-1.0 udpsrc port=5000 caps=application/x-rtp ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink
```

Streaming video (TCP+gstreamer):
```shell
# Sender
$ gst-launch-1.0 libcamerasrc ! \
     video/x-raw,colorimetry=bt709,format=NV12,width=1280,height=720,framerate=30/1 ! \
     queue ! jpegenc ! multipartmux ! \
     tcpserversink host=0.0.0.0 port=5000
# Sender (multi stream)
$ gst-launch-1.0 libcamerasrc name=cs src::stream-role=view-finder src_0::stream-role=video-recording \
    cs.src ! queue ! video/x-raw,width=640,height=480 ! videoconvert ! autovideosink \
    cs.src_0 ! queue ! video/x-raw,width=800,height=600 ! videoconvert ! \
    jpegenc ! multipartmux ! tcpserversink host=0.0.0.0 port=5000
# Receiver
$ gst-launch-1.0 tcpclientsrc host=$DEVICE_IP port=5000 ! \
     multipartdemux ! jpegdec ! autovideosink
```

Streaming video (TCP+libcamera):
```shell
# on server side (RPi3)
$ libcamera-vid -n -t 0 --listen -o tcp://0.0.0.0:<port>
# on client side (host)
$ ffplay tcp://<ip-addr>:<port> -vf "setpts=N/30" -fflags nobuffer -flags low_delay -framedrop
```

Streaming video+audio (TCP/UDP+libcamera+libav):
```shell
$ libcamera-vid -t 0 --codec libav --libav-format mpegts --libav-audio -o "tcp://0.0.0.0:<port>?listen=1"
$ libcamera-vid -t 0 --codec libav --libav-format mpegts --libav-audio  -o "udp://<dst-ip-addr>:<port>"
```

Streaming video (gstreamer):
```shell
$ gst-launch-1.0 libcamerasrc ! "video/x-raw,width=1280,height=720,format=NV12" ! v4l2convert ! v4l2h264enc extra-controls="controls,repeat_sequence_header=1" ! "video/x-h264,level=(string)4" ! h264parse ! fakesink
```

Streaming video (SRT):
```shell
# Server side
$ libcamera-vid -t 0 --width 1280 --height 720 -n --inline -o - | gst-launch-1.0 fdsrc fd=0 ! h264parse ! rtph264pay aggregate-mode=zero-latency config-interval=-1 ! gdppay crc-header=false crc-payload=false ! srtsink uri=srt://:<port>
# Client side
$ gst-launch-1.0 -v srtsrc uri="srt://<remote-ip-address>:<port>" ! gdpdepay ! rtph264depay ! h264parse ! avdec_h264 ! autovideosink sync=false
```

Dumping RAW frames:
```shell
$ libcamera-raw -t 2000 --width 640 --height 480 --segment 1 -o test%05d.raw
```

Detect objects using TFLite:
```shell
$ libcamera-detect -t 0 -o cat%04d.jpg --lores-width 400 --lores-height 300 --post-process-file object_detect_tf.json --object cat
```

## Options

Common options:
```
--width                 Capture image width <width>
--height                Capture image height <height>
--roi <x,y,w,h>         Capture only paritcular region of tereset (e.g. 0.25,0.25,0.5,0.5)
--hdr                   Enables HDR mode (only Camera Module 3 or Pi 5 supports thism ode)
--output                Sets the name of the output file (or -, or udp:// or tcp://, a string with %d directive or any standard C format directive)
--flush                 Causes output files to be flushed to disk immediately
```

Still command line:
```
--quality <number>      Sets the JPEG quality (100 - max, 93 - default)
--exif <string>         Adds extra exif tags (e.g. "--exif IDO0.Artist=Someone")
--timelapse <millis>    Captures repeated images according to given interval
--timeout 
--timestamp
--thumb <w:h:q>         Sets the dimensions and quality parameter of the associated thumbnail image
--encoding <jpg|png|bmp|rgb|yuv420> Select the still image encoding
--raw                   Save raw file
--autofocus-on-capture  Whether to run an autofocus cycle before capture
```

Video Command Line:
```
--quality <number>      Sets the JPEG quality when saving in MJPEG format (100 - max, 50 - default)
--bitrate <number>      [H.264] Set the target bitrate (bits per second, e.g. 10000000)
--intra <number>        [H.264] Set the frequency of I (intra) frames (60 by default)
--profile <baseline|main|high> [H.264] Set the profile
--level <4 | 4.1 | 4.2> [H.264] Set the level
--inline                [H.264] Enables writing sequence headers into every I (Intra) frame. Allows client to decode the video sequence from any I frame (not just from beginning of the stream). Usefull for cases of segmented writing (e.g. --segment or --circular) and for streaming over the network.
--codec <h264|mjpeg|yuv420|libav> Set encoder
--segment <number>      Enables writing the video recording into multiple segments (1 - means writing each frame to a separate output file)
--circular <size>       Enables the video to a circular buffer which is written to disk
--listen                Wait for an incoming TCP connection
--frames <number>       Record given number of frames
```

Control options:
```
--sharpness <0 (disable) |1.0 (default) | 2.0 (extra) | ...>
--contrast <0 (minimum)> | 1.0 (default) | 2.0 (extra) | ...>
--brightness <-1.0 (almost black) | 1.0 (almost white) | 0.0 (standard)>
--saturation <0 (grey scale) | 1.0 (default) | 2.0 (extra) | ...>
--ev <from -10 to 10, default is 0> - raise or low the target values the AEC/AGC algorithm to match
--shutter <number> - sets the exposure time in ms (e.g. 112sec is a maximum for Camera Module 3, see https://www.raspberrypi.com/documentation/accessories/camera.html#hardware-specification)
--exposure <normal|short|long>
--gain <number>
--metering <centre (default) | spot | average | custom (use tuning file)>
--awb <auto|incandescent|tungsten|fluorescent|indoor|daylight|cloudy|custom (use tuning file)> - change colour temperature
--denoise <auto|off|cdn_off,cdn_fast|cdn_hq> - specify denoise mode
--tuning-file - specify the camera tuning file to use
--autofocus-mode <default (continuous with respect to lens-position + autofocus-on-capture options) | manual | auto | continuous>
--autofocus-range <normal (default) | macro (focuses only on close objects) | full (focus on entire range)>
--autofocus-speed <normal | macro | full>
--autofocus-window <x,y,width,height (e.g. 0.25,0.25,0.5,0.5 - centering in the middle)>
--lens-position <0.0 (infinity position) | number>
```