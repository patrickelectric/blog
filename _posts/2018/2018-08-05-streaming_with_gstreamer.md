---
title: "Streaming with gstreamer"
header:
    teaser: assets/gstreamer/gst-launch.bmp
tags:
    - gstreamer
    - video
---

Gtreamer is a great tool for everything that need to deal with video transmission, some things you can do with it:
- Add a subtitle while the video is streaming
- Get video from file, udp, or v4l and store or transmit it
- Get two webcam videos, mix both together in the the same stream (Creating a stereo image)
- A RTSP server
- And etc..

![stereo](/assets/gstreamer/stereo.gif)

# Testing 1 2 4, Testing !

This pipeline will create a H264 video test source, encode it, and send via udp, creating the video with `videotestsrc`, selecting the video input in `video/`, encoding the video to transmit in `x264enc` and `rtph264pay`, and transmitting it with `udpsink`.
```
gst-launch-1.0 -v videotestsrc ! video/x-raw,width=800,height=600,codec=h264,type=video ! videoscale ! videoconvert \
! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600
```

![testsrc](/assets/gstreamer/testsrc-stream.png)

This one will get the video via udp with `udpsrc`, `rtpjitterbuffer` will create a buffer and remove any duplicate packets (removing unnecessary processing), `rtph264depay` will remove any unnecessary data in the packet and return only the stream, `avdec_h264` is the H264 decoder by [libav](https://www.libav.org/), and in the end we shows the output in fpsdisplaysink.

```
gst-launch-1.0 -vc udpsrc port=5600 close-socket=false multicast-iface=false auto-multicast=true \
! application/x-rtp, payload=96 ! rtpjitterbuffer ! rtph264depay ! avdec_h264 \
! fpsdisplaysink  sync=false async=false --verbose
```

![udpsrc](/assets/gstreamer/testudpsrc-stream.png)

![fpssink](/assets/gstreamer/gst-launch.bmp)

# Get the video from a real device

Ok, now, to get the output from your webcam, it's necessary to talk see what kind of resolution and encode the hardware can provide us.

This can be done by running:
```
v4l2-ctl --list-formats-ext
```

Or, if you want to get from a specif device:
```
v4l2-ctl --list-formats-ext --device=0
```

In my webcam, it returns:
```
ioctl: VIDIOC_ENUM_FMT
	Index       : 0
	Type        : Video Capture
	Pixel Format: 'MJPG' (compressed)
	Name        : Motion-JPEG
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 848x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 960x540
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.033s (30.000 fps)

	Index       : 1
	Type        : Video Capture
	Pixel Format: 'YUYV'
	Name        : YUYV 4:2:2
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 160x120
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 320x180
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 320x240
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 424x240
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x360
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 1280x720
			Interval: Discrete 0.100s (10.000 fps)
```

Ok, nice ! Now, this information is pretty important for ~~me~~ us, the `video/` width and height need to be compatible with the ones in this output, and doing so we can create our pipeline !
```
gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,width=1280,height=720,type=video \
! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 --verbose
```

![udpsrc](/assets/gstreamer/v4lsrc-stream.png)

The receive done in **Testing** will show the webcam video.

But how gstreamer know which encode we want to use ?
To discover this info, we can enable the debug level, with:

```
export GST_DEBUG="*:5"
```
From [developer.ridgerun](https://developer.ridgerun.com/wiki/index.php/GStreamer_Debugging), you can find this table that shows what each level mean:

| Debug level | Type of message | Description                                                                                                                                                                                                        |
|-------------|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0           |       none      | No debug information is output.                                                                                                                                                                                    |
| 1           |      ERROR      | Logs all fatal errors. These are errors that do not allow the core or elements to perform the requested action. The application can still recover if programmed to handle the conditions that triggered the error. |
| 2           |     WARNING     | Logs all warnings. Typically these are non-fatal, but user-visible problems are expected to happen.                                                                                                                |
| 3           |       INFO      | Logs all informational messages. These are typically used for events in the system that only happen once, or are important and rare enough to be logged at this level.                                             |
| 4           |      DEBUG      | Logs all debug messages. These are general debug messages for events that happen only a limited number of times during an object's lifetime; these include setup, teardown, change of parameters, ...              |
| 5           |       LOG       | Logs all log messages. These are messages for events that happen repeatedly during an object's lifetime; these include streaming and steady-state conditions.                                                      |
| 9           |    bufferdump   | Hex dump of buffer contents.                                                                                                                                                                                       |

Running:
```
gst-launch-1.0 v4l2src device=/dev/video0 ! videoconvert ! autovideosink
```

Will open our device and show the output for us, it's possible to see in debug level 4 all available video and image formats !

```
0:00:00.102958865 16444 0x5646d1572140 INFO        GST_ELEMENT_PADS gstelement.c:917:gst_element_get_static_pad: no such pad 'sink' in element "v4l2src0"
0:00:00.103172072 16444 0x5646d1572140 INFO                    v4l2 gstv4l2object.c:1196:gst_v4l2_object_fill_format_list:<v4l2src0:src> got 2 format(s):
0:00:00.103214415 16444 0x5646d1572140 INFO                    v4l2 gstv4l2object.c:1202:gst_v4l2_object_fill_format_list:<v4l2src0:src>   YUYV
0:00:00.103243944 16444 0x5646d1572140 INFO                    v4l2 gstv4l2object.c:1202:gst_v4l2_object_fill_format_list:<v4l2src0:src>   MJPG
Setting pipeline to PLAYING ...
0:00:00.103625400 16444 0x5646d1572140 INFO                    v4l2 gstv4l2object.c:4136:gst_v4l2_object_probe_caps:<v4l2src0:src> probed caps:
video/x-raw, format=(string)YUY2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1;
video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 };
video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 };
video/x-raw, format=(string)YUY2, width=(int)640, height=(int)360, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
video/x-raw, format=(string)YUY2, width=(int)424, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
video/x-raw, format=(string)YUY2, width=(int)320, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
video/x-raw, format=(string)YUY2, width=(int)320, height=(int)180, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
video/x-raw, format=(string)YUY2, width=(int)160, height=(int)120, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
image/jpeg, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 };
image/jpeg, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 };
image/jpeg, width=(int)960, height=(int)540, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1;
image/jpeg, width=(int)848, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1

```

And, when running our `v4l2src` pipeline:
```
0:00:00.036525158 18676 0x55cdad7e4e00 INFO            GST_PIPELINE gstparse.c:337:gst_parse_launch_full: parsing pipeline description 'v4l2src device=/dev/video0 ! video/x-raw,width=1280,height=720,type=video ! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 '
0:00:00.043026669 18676 0x55cdad7e4e00 INFO            GST_PIPELINE grammar.y:652:gst_parse_perform_link: linking some pad of GstV4l2Src named v4l2src0 to some pad of GstVideoScale named videoscale0 (0/0) with caps "video/x-raw, width=(int)1280, height=(int)720, type=(string)video"
0:00:00.168796760 18676 0x55cdada61c50 INFO                    v4l2 gstv4l2object.c:4136:gst_v4l2_object_probe_caps:<v4l2src0:src> probed caps: video/x-raw, format=(string)YUY2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1; video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 }; video/x-raw, format=(string)YUY2, width=(int)640, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 }; video/x-raw, format=(string)YUY2, width=(int)640, height=(int)360, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; video/x-raw, format=(string)YUY2, width=(int)424, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; video/x-raw, format=(string)YUY2, width=(int)320, height=(int)240, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; video/x-raw, format=(string)YUY2, width=(int)320, height=(int)180, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; video/x-raw, format=(string)YUY2, width=(int)160, height=(int)120, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; image/jpeg, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 }; image/jpeg, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction){ 30/1, 30/1 }; image/jpeg, width=(int)960, height=(int)540, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1; image/jpeg, width=(int)848, height=(int)480, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)30/1
0:00:00.192946256 18676 0x55cdada61c50 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstV4l2Src:v4l2src0.GstPad:src: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
0:00:00.193154229 18676 0x55cdada61c50 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstCapsFilter:capsfilter0.GstPad:src: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
0:00:00.193861576 18676 0x55cdada61c50 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstVideoScale:videoscale0.GstPad:src: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
0:00:00.194599343 18676 0x55cdada61c50 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-raw, width=(int)1280, height=(int)720, framerate=(fraction)10/1, format=(string)Y42B, type=(string)video, pixel-aspect-ratio=(fraction)1/1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstVideoConvert:videoconvert0.GstPad:src: caps = video/x-raw, width=(int)1280, height=(int)720, framerate=(fraction)10/1, format=(string)Y42B, type=(string)video, pixel-aspect-ratio=(fraction)1/1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstX264Enc:x264enc0.GstPad:sink: caps = video/x-raw, width=(int)1280, height=(int)720, framerate=(fraction)10/1, format=(string)Y42B, type=(string)video, pixel-aspect-ratio=(fraction)1/1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstVideoConvert:videoconvert0.GstPad:sink: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstVideoScale:videoscale0.GstPad:sink: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
/GstPipeline:pipeline0/GstCapsFilter:capsfilter0.GstPad:sink: caps = video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)10/1, format=(string)YUY2, pixel-aspect-ratio=(fraction)1/1, colorimetry=(string)2:4:7:1, interlace-mode=(string)progressive
0:00:00.614072982 18676 0x55cdada61c50 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-h264, codec_data=(buffer)017a001fffe1001d677a001fbcb200a00b7602d4040405000003000100000300148f18324801000568ebccb22c, stream-format=(string)avc, alignment=(string)au, level=(string)3.1, profile=(string)high-4:2:2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1, interlace-mode=(string)progressive, colorimetry=(string)bt709, chroma-site=(string)mpeg2
/GstPipeline:pipeline0/GstX264Enc:x264enc0.GstPad:src: caps = video/x-h264, codec_data=(buffer)017a001fffe1001d677a001fbcb200a00b7602d4040405000003000100000300148f18324801000568ebccb22c, stream-format=(string)avc, alignment=(string)au, level=(string)3.1, profile=(string)high-4:2:2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1, interlace-mode=(string)progressive, colorimetry=(string)bt709, chroma-site=(string)mpeg2
/GstPipeline:pipeline0/GstRtpH264Pay:rtph264pay0.GstPad:sink: caps = video/x-h264, codec_data=(buffer)017a001fffe1001d677a001fbcb200a00b7602d4040405000003000100000300148f18324801000568ebccb22c, stream-format=(string)avc, alignment=(string)au, level=(string)3.1, profile=(string)high-4:2:2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1, interlace-mode=(string)progressive, colorimetry=(string)bt709, chroma-site=(string)mpeg2
```

And as we can see here:

```
017a001fffe1001d677a001fbcb200a00b7602d4040405000003000100000300148f18324801000568ebccb22c, stream-format=(string)avc, alignment=(string)au, level=(string)3.1, profile=(string)high-4:2:2, width=(int)1280, height=(int)720, pixel-aspect-ratio=(fraction)1/1, framerate=(fraction)10/1, interlace-mode=(string)progressive, colorimetry=(string)bt709, chroma-site=(string)mpeg2
```
Gstreamer appears to be using our YUV webcam output (that is 4:2:2) to stream, and using a fps of 10 !

Checking the `udpsrc` pipeline, we can check the same thing !

```
0:00:02.181566030 19432 0x55be0166ead0 INFO               GST_EVENT gstevent.c:814:gst_event_new_caps: creating caps event video/x-raw, format=(string)Y42B, width=(int)1280, height=(int)720, interlace-mode=(string)progressive, multiview-mode=(string)mono, multiview-flags=(GstVideoMultiviewFlagsSet)0:ffffffff:/right-view-first/left-flipped/left-flopped/right-flipped/right-flopped/half-aspect/mixed-mono, pixel-aspect-ratio=(fraction)1/1, chroma-site=(string)mpeg2, colorimetry=(string)bt709, framerate=(fraction)10/1
/GstPipeline:pipeline0/avdec_h264:avdec_h264-0.GstPad:src: caps = video/x-raw, format=(string)Y42B, width=(int)1280, height=(int)720, interlace-mode=(string)progressive, multiview-mode=(string)mono, multiview-flags=(GstVideoMultiviewFlagsSet)0:ffffffff:/right-view-first/left-flipped/left-flopped/right-flipped/right-flopped/half-aspect/mixed-mono, pixel-aspect-ratio=(fraction)1/1, chroma-site=(string)mpeg2, colorimetry=(string)bt709, framerate=(fraction)10/1
```

Pretty interesting right ?! Gstreamer did all the dirty job for us.
Now you can play with it, like changing the fps to 30 !

```
gst-launch-1.0 -v v4l2src device=/dev/video0 ! video/x-raw,width=1280,height=720,type=video,framerate=30 ! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 --verbose 2>&1 | grep video/
0:00:00.011889942 20140 0x55783db05e00 INFO            GST_PIPELINE gstparse.c:337:gst_parse_launch_full: parsing pipeline description 'v4l2src device=/dev/video0 ! video/x-raw,width=1280,height=720,type=video,framerate=30 ! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 '
0:00:00.016560183 20140 0x55783db05e00 INFO            GST_PIPELINE grammar.y:652:gst_parse_perform_link: linking some pad of GstV4l2Src named v4l2src0 to some pad of GstVideoScale named videoscale0 (0/0) with caps "video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(int)30"
0:00:00.018836082 20140 0x55783db05e00 ERROR           GST_PIPELINE grammar.y:730:gst_parse_perform_link: could not link v4l2src0 to videoscale0, neither element can handle caps video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(int)30
WARNING: erroneous pipeline: could not link v4l2src0 to videoscale0, neither element can handle caps video/x-raw, width=(int)1280, height=(int)720, type=(string)video, framerate=(int)30
```

Oh, but wait, YUV can't handle this FPS, so we should move to MJPG.

```
gst-launch-1.0 -v v4l2src device=/dev/video0 ! image/jpeg,width=1280,height=720,type=video,framerate=30/1 ! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 --verbose
WARNING: erroneous pipeline: could not link v4l2src0 to videoscale0, videoscale0 can't handle caps image/jpeg, width=(int)1280, height=(int)720, type=(string)video, framerate=(fraction)30/1
```

Effing pipeline ! Oh wait, jpeg is a encode format, to do anything with it (like videoscale), we should decode it first.

```
gst-launch-1.0 -v v4l2src device=/dev/video0 ! image/jpeg,width=1280,height=720,type=video,framerate=30/1 ! jpegdec ! videoscale ! videoconvert ! x264enc tune=zerolatency ! rtph264pay ! udpsink host=127.0.0.1 port=5600 --verbose
```

Nice, now we can check the `udpsrc` pipeline output:

```
videodecoder gstvideodecoder.c:3718:gst_video_decoder_negotiate_pool:<avdec_h264-0> ALLOCATION (1) params: allocation query: 0x7f4e00008680, GstQueryAllocation, caps=(GstCaps)"video/x-raw\,\ format\=\(string\)I420\,\ width\=\(int\)1280\,\ height\=\(int\)720\,\ interlace-mode\=\(string\)progressive\,\ multiview-mode\=\(string\)mono\,\ multiview-flags\=\(GstVideoMultiviewFlagsSet\)0:ffffffff:/right-view-first/left-flipped/left-flopped/right-flipped/right-flopped/half-aspect/mixed-mono\,\ pixel-aspect-ratio\=\(fraction\)1/1\,\ chroma-site\=\(string\)mpeg2\,\ colorimetry\=\(string\)1:4:5:1\,\ framerate\=\(fraction\)30/1", need-pool=(boolean)true, pool=(GArray)NULL, meta=(GArray)NULL, allocator=(GArray)NULL;
```

Awesome ! 30 FPS !

And that is it, I'm writing this for my future me. Yes, gstreamer is hard, but maybe this step-by-step guide of _tunning_ a gstreamer pipeline can help !

# Bonus !
![bonus](/assets/gstreamer/simpsonsbonus.jpg)

This is my stereo camera pipeline:
```
gst-launch-1.0 videomixer name=m sink_1::xpos=320 ! videoconvert ! x264enc tune=zerolatency ! rtph264pay \
! udpsink host=127.0.0.1 port=5600 \
v4l2src device=/dev/video14 ! image/jpeg, width=320, height=240 ! jpegdec ! queue ! videoconvert \
! m. v4l2src device=/dev/video2 ! image/jpeg, width=320, height=240 ! jpegdec ! queue ! videoconvert ! m.
```

This is an example code in C++ that gets udp video from gstreamer and show it with OpenCV.
<script src="https://gist.github.com/patrickelectric/5dca1cb7cef4ffa7fbb6fb70dd9f9edc.js"></script>

And same example, but with python !

<script src="https://gist.github.com/patrickelectric/443645bb0fd6e71b34c504d20d475d5a.js"></script>

