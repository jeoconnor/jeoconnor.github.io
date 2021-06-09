# usb-live-2
Instructions for using [Hauppauge USB-Live-2](http://www.hauppauge.com/pages/products/data_usblive2.html) to convert VHS tapes to DVD 's

This describes just the specific steps that I used extracted from the very useful description at http://www.tedfelix.com/linux/hauppauge-usb-live-2-linux.html

The system I'm using is a Dell I5 desktop (circa 2013) with 8GB ram running LinuxMint 19.3.

1. Confirm that the USB-Live-2 device is present:
```
$ lsusb -t
/:  Bus 04.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 5000M
/:  Bus 03.Port 1: Dev 1, Class=root_hub, Driver=xhci_hcd/4p, 480M
    |__ Port 2: Dev 2, If 1, Class=Human Interface Device, Driver=usbhid, 1.5M
    |__ Port 2: Dev 2, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
    |__ Port 4: Dev 3, If 0, Class=Video, Driver=uvcvideo, 480M
    |__ Port 4: Dev 3, If 1, Class=Video, Driver=uvcvideo, 480M
    |__ Port 4: Dev 3, If 2, Class=Audio, Driver=snd-usb-audio, 480M
    |__ Port 4: Dev 3, If 3, Class=Audio, Driver=snd-usb-audio, 480M
/:  Bus 02.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
        |__ Port 6: Dev 3, If 0, Class=Human Interface Device, Driver=usbhid, 1.5M
/:  Bus 01.Port 1: Dev 1, Class=root_hub, Driver=ehci-pci/2p, 480M
    |__ Port 1: Dev 2, If 0, Class=Hub, Driver=hub/6p, 480M
        |__ Port 1: Dev 3, If 0, Class=Vendor Specific Class, Driver=, 480M
        |__ Port 1: Dev 3, If 1, Class=Vendor Specific Class, Driver=cx231xx, 480M
        |__ Port 1: Dev 3, If 2, Class=Vendor Specific Class, Driver=, 480M
        |__ Port 1: Dev 3, If 3, Class=Vendor Specific Class, Driver=, 480M
        |__ Port 1: Dev 3, If 4, Class=Vendor Specific Class, Driver=, 480M
        |__ Port 1: Dev 3, If 5, Class=Vendor Specific Class, Driver=, 480M
        |__ Port 4: Dev 5, If 0, Class=Mass Storage, Driver=ums-realtek, 480M

```
The driver is cx231xx on Bus 01.

I tried moving the USB-Live-2 to a different usb connection to put it on a different bus and the driver hung and I had to reboot.  Maybe there's a nice way to do this but I didn't bother to figure it out.

2. Next find the video device:
```
$ v4l2-ctl --list-devices
HD Pro Webcam C920 (usb-0000:00:14.0-4):
	/dev/video0
	/dev/video1

Hauppauge USB Live 2 (usb-0000:00:1a.0-1.1):
	/dev/video2
	/dev/vbi0
```
The video device here is `/dev/video2`

3. Check the current settings of the video device:
```
$ v4l2-ctl -d /dev/video2 --all
Driver Info (not using libv4l2):
	Driver name   : cx231xx
	Card type     : Hauppauge USB Live 2
	Bus info      : usb-0000:00:1a.0-1.1
	Driver version: 5.3.18
	Capabilities  : 0x85200011
		Video Capture
		VBI Capture
		Read/Write
		Streaming
		Extended Pix Format
		Device Capabilities
	Device Caps   : 0x05200001
		Video Capture
		Read/Write
		Streaming
		Extended Pix Format
Priority: 2
Video input : 0 (Composite1: ok)
Video Standard = 0x0000b000
	NTSC-M/M-JP/M-KR
Format Video Capture:
	Width/Height      : 720/480
	Pixel Format      : 'YUYV'
	Field             : Interlaced
	Bytes per Line    : 1440
	Size Image        : 691200
	Colorspace        : SMPTE 170M
	Transfer Function : Default (maps to Rec. 709)
	YCbCr/HSV Encoding: Default (maps to ITU-R 601)
	Quantization      : Default (maps to Limited Range)
	Flags             : 
Crop Capability Video Capture:
	Bounds      : Left 0, Top 0, Width 720, Height 480
	Default     : Left 0, Top 0, Width 720, Height 480
	Pixel Aspect: 11/10
Selection: crop_default, Left 0, Top 0, Width 720, Height 480
Selection: crop_bounds, Left 0, Top 0, Width 720, Height 480
Streaming Parameters Video Capture:
	Frames per second: 29.970 (30000/1001)
	Read buffers     : 2

User Controls

                     brightness 0x00980900 (int)    : min=0 max=255 step=1 default=128 value=128 flags=slider
                       contrast 0x00980901 (int)    : min=0 max=127 step=1 default=64 value=64 flags=slider
                     saturation 0x00980902 (int)    : min=0 max=127 step=1 default=64 value=64 flags=slider
                            hue 0x00980903 (int)    : min=-128 max=127 step=1 default=0 value=0 flags=slider
                         volume 0x00980905 (int)    : min=0 max=65535 step=655 default=60928 value=60928 flags=slider
                        balance 0x00980906 (int)    : min=0 max=65535 step=655 default=32768 value=32768 flags=slider
                           bass 0x00980907 (int)    : min=0 max=65535 step=655 default=32768 value=32768 flags=slider
                         treble 0x00980908 (int)    : min=0 max=65535 step=655 default=32768 value=32768 flags=slider
                           mute 0x00980909 (bool)   : default=0 value=0

```
The video device is set for PAL which is standard in Europe while NTSC is the USA standard.

4. Set video device to NTSC.
```
$ v4l2-ctl -d /dev/video2 -s ntsc
Standard set to 0000b000
```
5. To verify that the video device is working (no sound here):
```
ffplay -f video4linux2 /dev/video2
```
6. List audio devices:
```
$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: PCH [HDA Intel PCH], device 0: CX20641 Analog [CX20641 Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 0: PCH [HDA Intel PCH], device 2: CX20641 Alt Analog [CX20641 Alt Analog]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: C920 [HD Pro Webcam C920], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 3: Cx231xxAudio [Cx231xx Audio], device 0: Cx231xx Audio [Conexant cx231xx Capture]
  Subdevices: 0/1
  Subdevice #0: subdevice #0
```
The USB-Live-2 audio is on card 3.  In `ffmpeg` and `ffplay` commands it is specified as `-f alsa -i hw:3`

7. And here's the command I ended up using
```
sudo nice --20 ffmpeg -thread_queue_size 4096 -f alsa -i hw:3 -thread_queue_size 4096 -f video4linux2 -i /dev/video2 -t 02:00:00 -r:v ntsc -preset faster test.mp4
```

8. Turns out, it seems that worrying about the size of the file was unnecessary.  The program `devede`, a gui program, handles both converting the mp4 file to the appropriate format, mpeg-2, as well as resampling/resizing the file so that it will fit before burning the DVD.

When using devede, make sure that it is really using NTSC and not PAL format.  You can do this once it starts running like so: 
```
$ ps faxw | grep ffmpeg
 8798 ?        Rl     0:35          |           \_ ffmpeg -i /home/oconnor/the-love-bug.mp4 -i /home/oconnor/the-love-bug.mp4 -map 1:0 -map 0:1 -vf pad=720:540:0:30:0x000000,fifo,scale=720:480 -y -target ntsc-dvd -sn -g 12 -bf 2 -strict 1 -ac 2 -aspect 1.3333333333333333 -s 720x480 -trellis 1 -mbd 2 -b:a 224k -b:v 4158k /home/oconnor/movie/movies/movie_0.mpg
```

Notes:

*  It's a good idea to monitor cpu usage, at least initially, to make sure that the computer isn't being overtaxed.  For my case here, I was typically getting around 80% cpu usage on top.  Max is 400%.

*  At one point the log output started generating lots of messages like: 

```
   Past duration 0.604424 too large
   Past duration 0.600121 too large  764928kB time=00:52:09.59 bitrate=2002.3kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.600914 too large  764928kB time=00:52:10.09 bitrate=2001.9kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.604424 too large
   Past duration 0.602501 too large  765184kB time=00:52:11.62 bitrate=2001.6kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.600426 too large  765440kB time=00:52:13.13 bitrate=2001.3kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.600700 too large
   Past duration 0.603355 too large
   Past duration 0.604057 too large  765696kB time=00:52:15.14 bitrate=2000.7kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.602165 too large  765696kB time=00:52:16.65 bitrate=1999.8kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.603050 too large
   Past duration 0.603630 too large  765952kB time=00:52:18.17 bitrate=1999.5kbits/s dup=56 drop=0 speed=   1x    
   Past duration 0.601738 too large  766464kB time=00:52:20.19 bitrate=1999.5kbits/s dup=56 drop=0 speed=   1x    
``` 
   Which apparently can be ignored.  I read somewhere that the threshold is 0.6 and that in a more recent version of ffmpeg,   this message priority was changed from WARNING to VERBOSE.


