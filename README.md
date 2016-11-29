# proot-static-build
Build static variants of PRoot

PRoot static binaries were built using: 

    https://github.com/proot-me/proot-static-build 

   
## Docker build environment ##

Build the Docker container:

`docker build -t proot-buildenv .`

Run a build 
 `./build.sh make care proot -j4`

This creates a `target` folder with both static binaries for care and proot.

## Special variables

If needed you can override the mountpoints within the container by prefixing the call to build.sh with these variables.

### `VOLUMES`

  - Must be formatted as Docker expects them -> `/path_on_host:/mountpoint`
  - Specify multiple volumes using the `;` separator
  - The target in the containers should not be changed unless the container itself has been rebuilt with different paths
  - Example: `VOLUMES=/data/my_modified_proot.tar.gz:/opt/build/packages/proot-v5.1.1.tar.gz;/data/my_modified_care.tar.gz:/opt/build/packages/care-v2.2.2.tar.gz`

### `TARGET_DIR`

  - This is the destination folder for static binaries
  - Example: `TARGET_DIR=/tmp/mytarget`

### Example of modified command line

`VOLUMES=/data/my_modified_proot.tar.gz:/opt/build/packages/proot-v5.1.1.tar.gz;/data/my_modified_care.tar.gz:/opt/build/packages/care-v2.2.2.tar.gz TARGET_DIR=/tmp/mytarget ./build.sh make care proot -j4`

## Legacy builds (pre 5.1.1)

The ARM one was built on x86_64 using this command: 
   
    # Build PRoot/ARM statically: 
    cd ~/git/proot-static-build 
    mkdir build-arm 
    cd build-arm 
    proot -R ~/rootfs/slackwarearm-14.1 -b $(which cmake) -q qemu-arm make glibc-version=glibc-2.18 -f ../GNUmakefile proot -j4 

Where slackwarearm-14.1 was created this way:

    # Get Slackware/ARM packages: 
    wget -r -np http://ftp.arm.slackware.com/slackwarearm/slackwarearm-14.1/slackware/{a,ap,d,e,l,n,tcl}/ 
    mkdir ~/rootfs/slackwarearm-14.1 
    
    # Extract only a minimal subset (ignore errors): 
    ls ftp.arm.slackware.com/slackwarearm/slackwarearm-14.1/slackware/{a,l}/*.t?z | xargs -n 1 tar -C ~/rootfs/slackwarearm-14.1 -xf 

    # Do a minimal post-installation setup: 
    mv ~/rootfs/slackwarearm-14.1/lib/incoming/* ~/rootfs/slackwarearm-14.1/lib/ 
    mv ~/rootfs/slackwarearm-14.1/bin/bash4.new ~/rootfs/slackwarearm-14.1/bin/bash 
    proot -q qemu-arm -r ~/rootfs/slackwarearm-14.1 /sbin/ldconfig 
    proot -q qemu-arm -r ~/rootfs/slackwarearm-14.1 ln -s /bin/bash /bin/sh 

    # Install all package correcty (ignore warnings): 
    ls ftp.arm.slackware.com/slackwarearm/slackwarearm-14.1/slackware/*/*.t?z | xargs -n 1 proot -q qemu-arm -S ~/rootfs/slackwarearm-14.1 -b ftp.arm.slackware.com installpkg 

