# auto install of esxi with iso and kickstart

To create a custom is i recommend to use linux and with this i am using RHEL8.

Become admin user or use sudo then get the esxi iso.


create a folder

```
mkdir -p /tmp/esxi/esxi_iso
```

Go to folder

````
cd /tmp/esxi
```

Then mount the iso 
```
mount /tmp/esxi/esxi_iso
```

copy files to the working dir
````
cp -au /tmp/esxi/esxi_iso/*  /mnt/esxi/
```

Add the example kickstart file found in the repo to the working dir and make 
your changes. 
```
vim ks.cfg
```

Then edit the boot menu in the working dir
```
vim isolinux.cfg

```
Then add this line after the first called standard installer
and you can edidt menu label to your standard.
```
LABEL kickstart
  KERNEL mboot.c32
  APPEND -c boot.cfg ks=cdrom:/KS.CFG
  MENU LABEL ESXi-7.0U1c-17325551-standard kickstart
```

The option `ks=cdrom:/KS.CFG` will tell the installer where the kistart files is.

Then create a bios boot iso from the working dir.
```
mkisofs -o /tmp/esxi.iso -b isolinux.bin -c boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -J -r /tmp/esxi/
```

