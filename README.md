# JAVA work self
Raushan Java **code** (VMTCA) tool for image building.  is released every few months with contains tool enhancements, new rpm version for RHEL/CentOS Base image and new security hardening.



## Setting up JAVA
There are 2 ways to setup

### Using l
If you have access to a Linux system 

* Create an empty disk image  `javac javafile`
* Create a xml file. d as 

```xml
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/home/'/>
      <backingStore/>
      <target dev='vda' bus='virtio'/>
      <alias name='virtio-disk0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </disk>
  
```
* Run command `virsh create <domain xml file>` to launch the 

### Using OpenStack (CBIS) to bring up VMTCA VM in cloud
If you have access to a OpenStack cloud, you can launch VMTCA VM there. Here are the steps -


* Make sure there is a external network that has intranet access and **DHCP enabled**


## Building VNFC image using VMTCA
VMTCA works on layered customization approach. It comes with a basic cloud image as `base+cloud` layer. Any VNF can customize on top of `base+cloud` layer. For OPS VNFC the layer structure would look like -



`nothing` is a special layer that has no customization code.

 tool requires us to write image customization code in bash/korn shell scripts. The script naming is predefined. The order of script execution is also predefined. Here's the order -
1. Run customize script
2. Set parameters
3. Run prepare script
4. Run install script
5. Run finish script
5. Run package script

Each script mentioned above is executed (if present) for all the layers starting from `default` layer, before running the next script in the mentioned order.

Build will generate qcow2 image file in /tmp/ directory

## FAQ

**Is there any customised **  
Yes, it is there underand script name is build_image.sh

* Example usage:

This script is used for creation of the It will copy the latest stable image into your /tmp directory.



**How can I build a base image without any SDL related code?**  
In VMTCA VM run `use_vendor centos; crvmtemplate base+cloud`

**How can I build VNFC image with a custom built component rpm (without using one from CI/CIC stable repo)?**  
Assuming the custom rpm is `nokia-sdl-realtimedatabase-19.0.0.0-1.x86_64.rpm`, we can edit `image_building_tools/vmtca/sdl-layers/db/data/db_nokia_rpm_list` file and replace `nokia-sdl-realtimedatabase` with `nokia-sdl-realtimedatabase-19.0.0.0-1.x86_64`. Then we need to copy `nokia-sdl-realtimedatabase-19.0.0.0-1.x86_64.rpm` to `/var/crvmtemplate/rpms/` directory. Finally we can run `crvmtemplate db` command to create a DB image with the custom rpm.

**How can I modify an existing VNFC image?**  
At times we may need to modify a CI cleared image (e.g. update rpm, change some file permission, add some group/user). This can be achieved by the following steps -
* Attach the qcow2 as `/dev/vdb` of VMTCA VM. This requires different steps in libvirt and OpenStack environment.

For libvirt  
  * Provide the pre-built qcow2 path in VM domain xml file `<source file='/home/sarbajit/vnf-image.qcow2'/>`
  * Launch VMTCA VM

For OpenStack  
  * Upload the qcow2 image to glance
  * Create a cinder volume from the previously uploaded image as source
  * Attach the cinder volume to VMTCA VM as /dev/vdb

Once the image is mounted as /dev/vdb, create a `dummy` layer from *nothing* layer  
* `create_layer dummy nothing`
* Create a new file named `prepare` in `/lib/crvmtemplate/dummy/` path. Give executable permission. Add following content  

	```bash
	#!/bin/ksh
	set -e
	mount_existing
	exit 0
	```

* Add any customization code in `install` or `finish` script of dummy directory
* Run `crvmtemplate dummy` to build the image


**Where can I see the image building log?**
`crvmtemplate` command prints output to console and log file. The log file is present at `/var/crvmtemplate/log/crvmtemplate.out` path.


