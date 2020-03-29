# OpenRC On NanoBSD
Instructions to set up OpenRC services on NanoBSD

## How to build and install GhostBSD-based NanoBSD
[NanoBSD](https://www.freebsd.org/doc/en_US.ISO8859-1/articles/nanobsd/index.html) is a variant of FreeBSD for embedded systems suitable to run on devices such as Beaglebone, Raspberry PI etc. In my latest experience I was playing with GhostBSD to be run on the Beaglebone board. So the below examples will be based on such the setup. The NanoBSD tool makes it pretty simple to build an image with the following command line

1. Check out the source tree of FreeBSD system, in this case GhostBSD
```
git clone https://github.com/ghostbsd/ghostbsd
```
2. Go to the following subdirectory
```
cd ghostbsd/tools/tools/nanobsd/embedded
```
3. Run the build command
```
../nanobsd.sh -c beaglebone.cfg
```
If you would like to skip re-building world and kernel, the option **-n** for **NOCLEAN** can be used as follows:
```
../nanobsd.sh -n -c beaglebone.cfg
```
4. Afterwards the image located at the following path **|source directory prefix|/images/_.disk.image.beaglebone** can copied to flash device as follows:
```
sudo dd if=|source directory prefix|/images/_.disk.image.beaglebon of=/dev/da0 bs=16m
sudo sync
```
## How to set up OpenRC services on NanoBSD
After GhostBSD is booted up on Beaglebone you will probably find that a couple of more things needs to be setup such as networking, mounting of filesystem, time synchronization etc. GhostBSD is using OpenRC as init and run control system. Since NanoBSD is minimalistic (at its name suggests) the root partition is mounted in read-only mode. So system configurations are done as follows.
1. Mount a special configuration partition **/cfg**
```
mount /cfg
```
2. Copy system configuration file which you would like to tune
```
cp /etc/rc.conf /cfg/rc.conf
```
3. Add your configuration changes
```
echo "hostname=nanobeagle" >> /cfg/rc.conf
```
4. Reboot the device

Many of the functionality described in the beginning of the paragraph is managed by corresponding OpenRC services in GhostBSD. Due to the nature of NanoBSD enabling service via the following command will not work in the sense the service will not be persisted to the runlevel of the OpenRC system
```
rc-update add network
```
So the following steps will be needed to achieve this:
1. Copy **runlevels** directory to the configuration partition
```
cp -r /etc/runlevels /cfg/
```
2. Enter the directory of the runlevel of the service being set up
```
cd /cfg/runlevels/default
```
3. Create symlink to the OpenRC  script that corresponds the given service
```
ln -s /etc/init.d/network network
```
To wrap this up I made a few changes to NanoBSD build scripts in order to incorporate essential services to make the out-of the-box usage experience a bit better. You can find the proposed changes [here](https://github.com/olehzdubna/ghostbsd-emb/pull/1/files).
