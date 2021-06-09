# Yocto Package Server 

For this example chose the opkg package manager option by added `opkg`
and `opkg-utils` to `EXTRA_IMAGE_FEATURES` and setting
`PACKAGE_CLASSES` to `"package_ipk"' in `conf/local.conf`.

Once a build is complete, ie. `bitbake core-image-minimal` has
completed, generate the package indexes with `bitbake package-index`.

The switch to the `tmp/deploy/ipk` directory and start up the http server with
```
$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```
With the corresponding boot image start up the raspberry pi and create the file `/etc/opkg/base-feeds.conf` with the content:
```
src/gz all http://192.168.1.138:8000/all                                        
src/gz cortexa7t2hf-neon-vfpv4 http://192.168.1.138:8000/cortexa7t2hf-neon-vfpv4
src/gz raspberrypi3 http://192.168.1.138:8000/raspberrypi3                      
```
using the correct IP address for the host where the http server is running.

Confirm that it's working:
```
# opkg list | wc -l
6055
```
Prior to adding `base-feeds.conf` and starting the http server on the host, the same query came up empty.
