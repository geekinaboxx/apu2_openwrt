# apu2_openwrt
This is my customised OpenWrt image for apu2 boards. It extends the generic x86_64 OpenWrt builds by:

* Specific kernel modules for specific APU2 features
* Two ports are active by default: eth0 set to PPPoE WAN; eth1 for LAN
* Adding LuCI with HTTPS SSL support; along with the LuCI Material theme in addition to the default bootstrap
* Replacing dnsmasq with dnsmasq-full
* IPv6: tunnel support for 6in4, 6to4, 6rd
* SQM scripts
* Stubby, Unbound
* Adblock package
* banIP
* vpn-policy-routing package
* Wireguard support
* USB storage support
* wget, curl
* usbtools
* Limited support for 3G/4G USB modems via the ncm and 3g protocols
* Wake-on-LAN LuCI module

The images can be found in the bin/ directory of this repo. Note that _no warranty whatsoever is provided with these_.
 
# Recreating my build environment

I use the image builder rather than building from source. To create the builds, I use a Debian VM running in VirtualBox on MacOS. If you'd like to recreate my build environment, you can follow these instructions:

### Setting up a VM
1. Download & verify the Debian netinst CD Image for amd64 from https://www.debian.org/CD/netinst/. Later, this image can also be used to create a bootable USB which can be used to apply the OpenWrt image to the APU2's mSATA drive
2. Launch VirtualBox and create a new VM
3. Name the VM (e.g. Debian 9.9.0). Set the type as "Linux" and Version as "Debian (64-bit)"
4. Choose a memory size (RAM) for your VM. Note that the minimum memory requirement for Debian is 512MB
5. Choose ‘Create a virtual hard disk now’ option and click 'create'
6. Choose VDI, 'Dynamically allocated' options.
7. Provide a Hard Disk size (8GB fine for a minimal install: no graphical environment - just SSH server + base system, and the ImageBuilder deps)
8. Now click on ‘[Optical Drive] Empty’. Select ‘Choose disk image…’ and select the Debian netinst image that you downloaded
9. Start the VM and follow the installation instructions

During installation, you'll create a root user and non-root user. It's a good idea to grant sudo access to the non-root user:

```
su root
apt-get install sudo
adduser <username> sudo
exit
```

### Installing Guest Additions

This can be useful for sharing folders between the host and the VM. As root, do:

```
apt-get update
apt-get upgrade
apt-get install build-essential module-assistant
m-a prepare

```

Choose Devices > Insert Guest Additions from the VitualBox menu, and then:

```
mount /media/cdrom
sh /media/cdrom/VBoxLinuxAdditions.run
```

Note that shared folders can be found in /media/*

### Building an customising an image

Clone this repo, and `cd` into `buildEnv`. Make `apu2BuildRoot.sh` executable by:

```
chmod 755 apu2BuildRoot.sh
```

modify the script if you wish: if you want wireless support, you'll need to install hostapd along with kmod-ath9k & ath9k-htc-firmware for the wle200nx; or ath10k-firmware-qca988x and kmod-ath10k for the wle600vx or wle900vx.

Now, go ahead and run it!

```
./apu2BuildRoot.sh
```

You'll be given the choice to build from Snapshot, 18.06.4 or 19.07

### Testing the images
The built images will be located in the `bin` directory. These can be tested in VirtualBox:

1. Copy the image to the host machine and unzip it `gzip -d openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.img.gz`
2. Run `VBoxManage convertfromraw --format VDI openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.img openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.vdi`

If you see an error like:

`VBoxManage: error: VD: The given disk size 23499980 is not aligned on a sector boundary (512 bytes)` 

then run:

`dd if=openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.img of=openwrt.img bs=128000 conv=sync` first

Finally, enlarge the image:

```VBoxManage modifymedium openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.vdi --resize 128```

3. Launch VirtualBox and create a new VM
4. Name the VM (e.g. OpenWRT). Set the type as "Linux" and Version as "Other Linux (64-bit)"
5. Accept the defaults, and then choose "Use an existing virtual hard disk file" selecting the VDI you created in the previous steps
6. Launch!

### Flash it!

1. Flash the Debian netinst amd64 iso to a USB drive using Etcher (https://www.balena.io/etcher/)
2. 
3. Get the image `https://raw.githubusercontent.com/geekinaboxx/apu2_openwrt/master/bin/openwrt-18.06.4-apu2-2nic-geekinaboxx-x86-64-combined-squashfs.img.gz`

