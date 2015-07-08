  
###Bitbake Environment

#####Contents

#####External links  
-  <a href=http://www.yoctoproject.org/docs/1.6.1/dev-manual/dev-manual.html#configuring-the-kernel>configure the Kernel</a>
-  <a href=http://git.yoctoproject.org/cgit.cgi/poky/plain/meta/conf/bitbake.conf?h=blinky>oe-core/meta/conf/bitbake.conf : Bitbake variables. see also under each Layer ./conf/bitbake.conf</a>

### Bitbake vars  
-  <a href=http://git.yoctoproject.org/cgit.cgi/poky/plain/meta/conf/bitbake.conf?h=blinky>oe-core/meta/conf/bitbake.conf : Bitbake variables. see also under each Layer ./conf/bitbake.conf</a>


### Important Vars.  

PREFERRED\_PROVIDER =  
PREFERRED\_VERSION  =  
DEFAULT_PREFERENCE  =  [ -1 | 0 | 1]  default is "0", "-1" recipe unlikely, "1" likely the recipe is used PREFERRED\_VERSION overrides any DEFAULT\_PREFERENCE


###Task Stages:  

- fetch 
- unpack 
- patch
- configure
- compile
- install
- package
- package_write
- build 



example:  
  given dir structure note u-boot-am33x_%.bbappend :  

tisdk/sources/**meta-ti**/recipes-bsp/u-boot/u-boot-am33x_2013.01.01.bb  
tisdk/sources/**meta-arago**/meta-arago-distro/recipes-bsp/u-boot/u-boot-am33x_2013.01.01.bbappend  
tisdk/sources/**meta-methode**/recipes-boot/u-boot-am33x/u-boot-am33x_%.bbappend  
tisdk/sources/meta-methode/recipes-boot/u-boot-am33x/u-boot-am33x/0001-u-boot-2013.01.01-cti-dcim.patch  


Note the three layers:  

- meta-ti    .bb  
- meta-arago .bbappend  
- meta-methode .bbappend  

###Trouble shooting.

Break up the process by executing task seperately. 

```
commands: 
 bitbake -c fetch u-boot-am33x
 bitbake -c unpack u-boot-am33x 
 bitbake -c patch u-boot-am33x 
 bitbake -c configure u-boot-am33x 
 bitbake -c compile u-boot-am33x 

```



-----------------------------------

#####Trouble: 


-----------------------------------

#####Trouble: Port  3.2 board-dcim.c to 3.14 board-dcim.c

Track down Undefines and #defines that changed.


-----------------------------------

#####Trouble: Dependencies 

```
bitbake -g -u depexp u-boot-ti-staging

``` 
This brings up a x-windows app to show dependencies.

-----------------------------------

#####Trouble: linux-ti-staging for am335x-evm Over writes kernel config values

Low and behold.
TI - created a dir ./ti_config_fragments/ and a file defconfig which gets interpreted and creates .config
the meta-ti/recipes-kernel/linux/linux-ti-staging_3.14.bb Appends ./ti_config_fragments/am33xx_only.cfg to the mix and can overwrite your configs if you are unaware.  So, not only has TI obfuscated the config with the "defconfig_fragment" they then backdoored the fragments with a BB recipe variable.

It would have been simple for the user to create his own fragments and take over the defconfig but TI comes back through the recipe VARS and appends configs again, so approach 2 is my choice, the alternative is to remove any appends in the linux-ti-staging-3.14.bb recipe from your own bbappend recipe.


1) I created my on defconfig and defconfig_fragment trying to override the TI defconfig  and got land-mined from this and from the ./git/../configs/systest  config.

2) I backed out my changes, now use the TI defconfig_fragment and use there variable in a bbappend file to append my configs.

...So, the philosophy here is let TI do there Config.  Compare against what My target Configs look like and then override with methode.cfg by appending to VAR to override config flags.  

Note time saver:  
bitbake -c configure  linux-ti-staging  
 to just configure your build and check the flags before compiling.

```
S = "${WORKDIR}/git"
#KERNEL_CONFIG_DIR = "${S}/ti_config_fragments"
#KERNEL_CONFIG_FRAGMENTS_remove = " ${KERNEL_CONFIG_DIR}/am33xx_only.cfg"

KERNEL_CONFIG_FRAGMENTS += " ${S}/../methode.cfg"
```

-----------------------------------

#####Trouble: Adding A Package.  
http://blogs.mentor.com/chrishallinan/blog/2012/04/27/more-on-yocto-terminology-recipes-and-packages/  

Images, Recipes and Packages

This is most confusing when trying to add new packages to your root file system image.  A recent example of this was encountered when one of our customers asked how to incorporate the NET-SNMP package into his image.  It seemed obvious to him (and to this author) that because there is a nice, tidy recipe called net-snmp_5.7.1.bb, all he needed to do was add this to his image.  He tried the obvious, based on general instructions in the various manuals which describe adding packages to images:

IMAGE_INSTALL += "net-snmp"

However, that failed because while there is a recipe called net-snmp, there are no packages by that name.  Root file system images are created from packages, and not recipes.  Looking at the packages produced by the net-snmp recipe, using BitBake’s -e switch, we have:

$ bitbake -e net-snmp | grep ^PACKAGES=
PACKAGES="net-snmp-dbg net-snmp-doc net-snmp-dev net-snmp-staticdev net-snmp-static net-snmp-libs net-snmp-mibs net-snmp-server net-snmp-client"

Looking at this output, it becomes clear that the packages desired for installation on the root file system are net-snmp-server, net-snmp-client, and possibly the supporting packages such as the -libs and -mibs. So your image must be modified to have these packages installed.  In summary, the correct way to specify this is as follows:

IMAGE_INSTALL += "net-snmp-server net-snmp-client net-snmp-libs net-snmp-mibs"

BitBake -e is your friend.  Use it often – it can really help you understand what’s going on when things don’t work out as you expect.




-----------------------------------

#####Trouble: kernel config over written

bitbake -c configure linux-ti-staging  
"works"  git/../defconfig is ok, and .config is ok.

but :  
bitbake -c compile linux-ti-staging 

over writes the ".config" file and git/../defconfig


git/.config is being over written by git/../configs/systest 
git/../defconfig is over written and a file git/../defconfig.save shows up.

Solution:  
create a file : 
in sources/meta-methode/recipes-kernel/linux/files/configs/empty

This prevents the meta-ti recipe from using the default git/../configs/systest config file.


-----------------------------------

#####Trouble: Failed w/ message patch\_do\_patch (enforce with -f) :    
bitbake u-boot-am33x 

```
Patch 0001-u-boot-2013.01.01-cti-dcim.patch does not apply (enforce with -f)
ERROR: Function failed: patch_do_patch
ERROR: Logfile of failure stored in: /home/wwright/dev/dcim3.x/tisdk/build/arago-tmp-external-linaro-toolchain/work/am335x_evm-linux-gnueabi/u-boot-am33x/2013.01.01-r6+gitrAUTOINC+540aa6fbb0-arago1/temp/log.do_patch.18173
ERROR: Task 1 (/home/wwright/dev/dcim3.x/tisdk/sources/meta-ti/recipes-bsp/u-boot/u-boot-am33x_2013.01.01.bb, do_patch) failed with exit code '1'
```

Created a patch:  
using a DCIM_Uboot tar.gz file  **u-boot-2013.01.01.dcim.tar.gz**

1)  first download and upack the target.
```
bitbake -c unpack u-boot-am33x
```
You should find it here:
```
../build/arago-tmp-external-linaro-toolchain/work/am335x_evm-linux-gnueabi/u-boot-ti-staging/2013.01.01-r11+gitrAUTOINC+540aa6fbb0-arago7

cd build/arago-tmp-external-linaro-toolchain/work/am335x_evm-linux-gnueabi/u-boot-ti-staging/2013.01.01-r11+gitrAUTOINC+540aa6fbb0-arago7

tar -C git -xzvf u-boot-2013.01.01.dcim.tar.gz 
git status
git add .
git commit -m "u-boot 2013.01.01 dcim"
git format-patch -1 -o /tmp
cd ~/dev/dcim3.x/tisdk/sources/meta-methode/recipes-boot/u-boot-ti-staging

cp /tmp/0001-u-boot-2013.01.01-cti-dcim.patch ./u-boot-ti-staging

cd ~/dev/dcim3.x/tisdk/build
bitbake -v -cleansstate u-boot-am33x
bitbake -c unpack u-boot-am33x
cd 
 patch -p1 -f  --ignore-whitespace < ../0001-u-boot-2013.01.01-cti-dcim.patch
...repeat creating patch...
git add .
git commit -m "u-boot 2013.01.01 dcim"
git format-patch -1 -o /tmp

cd ~/dev/dcim3.x/tisdk/sources/meta-methode/recipes-boot/u-boot-ti-staging

cp /tmp/0001-u-boot-2013.01.01-cti-dcim.patch ./u-boot-am33x
-------------------------------------------------
user@box:~/dev/dcim3.x/tisdk/sources/meta-methode/recipes-boot$ cat ./u-boot-ti-staging/u-boot-ti-staging_%.bbappend
FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"
UBOOT_MACHINE = "dcim_config"
SRC_URI += "file://0001-u-boot-2013.01.01-cti-dcim.patch"
LIC_FILES_CHKSUM = "file://README;md5=35cbc54e665b3ffb33ffb26ef322afb8"
PV = "2013.01.01"

# This version of u-boot is meant for 3.2 kernel which doesn't support device tree.
BRANCH = "ti-u-boot-2013.01.01-amsdk-06.00.00.00"
SRCREV = "540aa6fbb0c9274bda598f7e8819ed28259cad6b"

# Set the name of the SPL that will built so that it is also packaged with u-boot.
SPL_BINARY = "MLO"

-----------------------------------------------

cd ~/dev/dcim3.x/tisdk/build
bitbake -v -cleansstate u-boot-ti-staging
bitbake -v -k u-boot-ti-staging 
 
```
The above 1) untar the files on top of the git from the am33x target. 2)get status to view the changes. 3) git add to stage the changes. 4) commit the changes 5) create the patch file. 6) clean and fetch/unpack the target again
7) cd back to work dir. 8) and do a patch by hand using the --ignore-whitespace 9) create patch again and copy back to meta-methode layer 10) Now you can cleansstate and build with out the errormessage. enforce -f   

  


------------------------------------------  


#####Trouble: Tribbles  




