#Ubuntu and Debian

The essential and graphical support packages you need for a supported Ubuntu or Debian distribution are shown in the following command:   

```
sudo apt-get install sed wget cvs subversion git-core coreutils \
     unzip texi2html texinfo libsdl1.2-dev docbook-utils gawk \
     python-pysqlite2 diffstat help2man make gcc build-essential \
     g++ desktop-file-utils chrpath libgl1-mesa-dev libglu1-mesa-dev \
     mercurial autoconf automake groff libtool xterm

git clone http://git.yoctoproject.org/git/poky
cd poky
git checkout -b dizzy origin/dizzy
source oe-init-build-env
```

#Tip

To help conserve disk space during builds, you can add the following statement to your project's configuration file, which for this example is poky/build/conf/local.conf. Adding this statement deletes the work directory used for building a package once the package is built.  
```
 INHERIT += "rm_work"
```

# Configure

1)default architecture qemux86  
Intel32-bit based architecture  

2) you should increase these settings to twice the number of cores used. Doing so can significantly shorten your build time.


```
# This sets the default machine to be qemux86 if no other machine is selected:
#1)
MACHINE ??= "qemux86"

#2)
PARALLEL_MAKE ?= "-j 8"
BB_NUMBER_THREADS ?= "8"


PACKAGE_CLASSES
```

Another consideration before you build is the package manager used when creating the image. By default, the OpenEmbedded build system uses the RPM package manager. You can control this configuration by using the PACKAGE_CLASSES variable. For additional package manager selection information, see the "package*.bbclass" section in the Yocto Project Reference Manual.

Continue with the following command to build an OS image for the target, which is core-image-sato in this example. For information on the -k option use the bitbake --help command, see the "BitBake" section in the Yocto Project Reference Manual, or see the "BitBake Command" section in the BitBake User Manual.


# Build

```
bitbake -k core-image-sato 

```

# Run
```
runqemu qemux86
```





