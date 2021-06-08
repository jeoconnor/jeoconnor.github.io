# Converting VHS to DVD

Found good reference at http://www.tedfelix.com/linux/hauppauge-usb-live-2-linux.html

Working with a LinuxMint 19.1 system and found it the package version of ffmpeg is not compiled with all the features needed.

Clone ffmpeg from https://git.ffmpeg.org/ffmpeg.git

ffplay wasn't getting built and found the missing dependencies to build ffplay with
```
$ grep ffplay_dep config.log
ffplay_deps='avcodec avformat swscale swresample sdl2' 
```  

The `--preset OPTION` described in the above document wasn't available in the default configuration until built with the configure options, `--enable-libx264 --enable-gpl`.  

It was also necessary to install the library `libx264-dev`

The basic command for generating an mp4 file from the usb-live2 output stream is ```
sudo nice --20 ffmpeg -f alsa -i hw:2 -f v4l2 -i /dev/video0 -t 01:30:00 -r:v ntsc -preset ultrafast output.mp4```

The correct devices are found from instructions in the above reference.

Write the DVD using the program `devede`.  I have two DVD players, both SONY.  Initially, the test DVD that I wrote would only play on the newer one.  The older one refused to play due to region restriction.  I then set the region of the DVD write to 1 using `regionset 1` and then it worked on the older player as well.

