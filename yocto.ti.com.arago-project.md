
###Reference

http://arago-project.org/wiki/index.php/Setting_Up_Build_Environment#Available_Layer_Configurations


------------------------------------
### WWright Arago Project History background.

My understanding ,  

* Poky Linux is a build tool  
* OpenEmbedded requires Poky. OE is an all incompassing embedded Linux
* YoctoLinux requires Poky and OE-core Meta Layer. Yocto is limited focus of board support.
* Arago Project is a Yocto branch specifically for TI embedded boards.

So we should be able to checkout the arago project which will pull from yocto, and oe and poky as needed.  

**Anything** marked with "(ww)" is my addition to the doc from arago web.

-------------------------------------

####Cross-compile toolchain

While meta-arago can use different toolchains, the one currently recommended is the Linaro 2013.03. If you don't have it installed, download and unpack the binary distribution.
Install the Linaro toolchain  

```
$ wget --no-check-certificate \
   https://launchpad.net/linaro-toolchain-binaries/trunk/2013.03/+download/gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux.tar.bz2
$ tar -jxvf gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux.tar.bz2 -C $HOME
```

The toolchain will be installed in the gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux directory under your $HOME directory. You can also install it in a system-wide location, just make sure to update the location in your PATH environment variable.
Users of ARM 9 machines

For users who are interested in ARM 9 machines (ie AM18x) then the Linaro toolchain can not be used. Instead the Arago Toolchain must be used.
Install the Arago toolchain

```
$ wget http://downloads.ti.com/sdoemb/sdoemb_public_sw/arago_toolchain/2011_09/exports/arago-2011.09-armv5te-linux-gnueabi-sdk.tar.bz2
$ tar -jxvf arago-2011.09-armv5te-linux-gnueabi-sdk.tar.bz2 -C $HOME
```

The toolchain will be installed in the arago-2011.09 directory under your $HOME directory. You can also install it in a system-wide location, just make sure to update the location in your PATH environment variable.
Host tools

Some generic development tools are required on your host Linux machine. The below commands are specific to Ubuntu Linux distribution (tested on 10.04 and 12.04 versions), please adopt to your distribution of choice accordingly.
Install development tools

```
$ sudo apt-get install git build-essential diffstat texinfo gawk chrpath
```
*(ww)* **UBOOT tools**
```
$ sudo apt-get install u-boot-tools
```
*(ww)* **HOB Yocto project GUI**  
```
$ sudo apt-get install python-gtk2-dev
```
*(ww)* My ubuntu 14.4 does not recognize ia32lib  as a package
If you are using a 64-bit Linux, then you'd also need to install 32-bit support libraries, needed by the pre-built Linaro toolchain and other binary tools.
Install 32-bit support libraries

*(ww)* **Because:** Version 4.7 gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux.tar.bz2  I am down loading mutltilib 4.7

**x86_64 Ubuntu**  

```

$ sudo apt-get install gcc-multilib-4.7
$ sudo apt-get install zlib1g:i386
$ sudo apt-get install lib32ncurses5-dev

```

####Use Bash  
It is also recommended to switch your system shell from Ubuntu's standard dash to more universal bash.
Switch shell from dash to bash
```
$ sudo dpkg-reconfigure dash
```

Select no when prompted

####Quick Start

To quickly start making your own builds using meta-ti BSP layer and meta-arago Distribution layer, you can follow this short Quick Start section by entering below commands. For more expanded guide with each step detailed and sample output of the entered commands shown, please see the next Detailed Setup section.
Quick Start commands summary


---------------------------------------------
#####NOTE: Also see Customize board :  
- **http://processors.wiki.ti.com/index.php/Sitara_Linux_Training:_Getting_Started_with_Openembedded**  

---------------------------------------------


```
$ git clone git://arago-project.org/git/projects/oe-layersetup.git tisdk
$ cd tisdk

# If DCIM 1) Customize board u-boot dcim #
$ ./oe-layertool-setup.sh -f ./configs/amsdk/amsdk-06.00.00.00-config.txt

# Else 2)  -- Default stetup form arago wiki
$ ./oe-layertool-setup.sh -f configs/arago-daisy-config.txt
#

$ cd build
$ . conf/setenv
$ export PATH=$HOME/gcc-linaro-arm-linux-gnueabihf-4.7-2013.03-20130313_linux/bin:$PATH
$ MACHINE=am335x-evm bitbake core-image-minimal
```

**NOTE:** You can change the config above to the one matching the Yocto release you are using. See Available Layer Configurations section below.
Detailed Setup

Setting up the Build Environment for meta-ti/meta-arago is quite easy - most of the heavy lifting is done by the setup scripts. This section explains the required steps in more details.
Clone the Setup script

The setup script itself and a number of sample configuration files are stored in a public git repository on the arago-project.org server. Follow the below commands to clone the latest version of the oe-layersetup scripts repository.
Clone OE Layer Setup script
```
$ git clone git://arago-project.org/git/projects/oe-layersetup.git tisdk

Cloning into 'tisdk'...
remote: Counting objects: 318, done.
remote: Compressing objects: 100% (309/309), done.
remote: Total 318 (delta 201), reused 0 (delta 0)
Receiving objects: 100% (318/318), 42.86 KiB, done.
Resolving deltas: 100% (201/201), done.

$ cd tisdk
```
