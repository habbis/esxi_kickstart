# auto install of esxi with iso and kickstart

To create a custom is i recommend to use linux and with this i am using RHEL8.

Become admin user or use sudo then get the esxi iso.


create a folder

```
mkdir  /tmp/esxi/
mkdir  /tmp/custom_esxi
```

Go to folder

```
cd /tmp/esxi
```

Then mount the iso 
```
mount esxi.iso /tmp/esxi_iso
```

copy files to the working dir
```
cp -au /tmp/esxi/esxi_iso/*  /tmp/custom_esxi/
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
mkisofs -o /tmp/esxi.iso -b isolinux.bin -c boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -J -r /tmp/custom_esxi
```
To create uefi that also work with bios 
```
genisoimage -relaxed-filenames -J -R -o ~/custom_esxi.iso -b isolinux.bin -c boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e efiboot.img -no-emul-boot /tmp/custom_esxi
```

How to mask luns from esxi installer

The example below describes how to disable Fiber Chanel during the deployment process.

Prepare TGZ module with scripts injected

Create folders tree structure

```
mkdir -p test/etc/rc.local.d
cd test/etc/rc.local.d
```

Create mask.sh file with the following content

```
#!/bin/bash
localcli storage core claimrule add -r 2012 -P MASK_PATH -t transport -R fc
localcli storage core claimrule load
localcli storage core claiming unclaim -t plugin -P NMP
localcli storage core claimrule run
cat >> /etc/rc.local << __CLEANUP_MASKING__
localcli storage core claimrule remove -r 2012
__CLEANUP_MASKING__
cat > /etc/init.d/maskcleanup << __CLEANUP_MASKING__
sed -i 's/localcli.*//g' /etc/rc.local
rm -f /etc/init.d/maskcleanup
__CLEANUP_MASKING__
chmod +x /etc/init.d/maskcleanup

```


Make the script file executable

```
chmod +x mask.sh
```

Package the module

```
cd ../..
tar cvf mask.tgz *
```

Upload/Mount ESXi image to your Linux machine and copy ISO content locally
For uploaded ISO image:

```
mkdir esxi_source
sudo mount -o loop VMware-VMvisor-Installer-6.0.0.update02-3620759.x86_64.iso /mnt
cp -r /mnt esxi_source
sudo umount /mnt
```

Copy mask.tgz module to the esxi_source folder 

Append boot.cfg file in esxi_source folder to load our module

Add ' --- /mask.tgz' to the end of the string with modules
examples for uefi boot and bios boot '

```
sed -i '/^modules/ s/$/ --- \/mask.tgz/' efi/boot/boot.cfg

sed -i '/^modules/ s/$/ --- \/mask.tgz/' ./boot.cfg
```

Then go the section on how to generate the custom iso


### working with Tar xz

compress

```
$ tar cJvf xzcompressed.tar.xz directory3
```
Peek
```
$ tar tJvf xzcompressed.tar.xz
```
Decompress
```
$ tar xJvf xzcompressed.tar.xz
```
To sha256sum hash to the password for kickstart 
```
openssl passwd -5 "yourpassword"
```
Links: 


https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.install.doc/GUID-C03EADEA-A192-4AB4-9B71-9256A9CB1F9C.html

http://www.softpanorama.org/Commercial_linuxes/RHEL/Installation/Kickstart/creating_boot_image_that_points_to_remote_kickstart_file.shtml

http://www.softpanorama.org/Commercial_linuxes/RHEL/Installation/Kickstart/modifing_iso_image_to_include_kickstart_file.shtml

https://docs.vmware.com/en/VMware-vSphere/7.0/com.vmware.esxi.upgrade.doc/GUID-61A14EBB-5CF3-43EE-87EF-DB8EC6D83698.html
