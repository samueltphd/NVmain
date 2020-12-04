## Running C/C++ Scripts on Non-Volatile Memory Using gem5 and NVmain

**author: Sam Thomas, Brown University**
**Contact samuel_thomas@brown.edu to report errors**

0. _Workspace Setup_
This document assumes that the host system is running Ubuntu 20.04.1 on an x86_64 machine and will run an ARM configuration of gem5.
`~ $ sudo apt-get install build-essential git scons device-tree-compiler bzip2 python-dev mount`
* build-essential: general-purpose Ubuntu command-line tools
* git: used to download
* scons: used to build gem5 executables
* bzip2: used to unzip image file for Ubuntu 18.04
* python-dev: used to mount disk image onto directory
* mount: used to unmount disk image from directory

`~ $ mkdir workspace`

`~ $ cd workspace`
1. _Installing gem5_

`workspace $ git clone https://gem5.googlesource.com/public/gem5`

`workspace $ cd gem5`
2. _Compile gem5_
`gem5 $ scons build/ARM/gem5.opt`
(note: scons will create the "build" repository and its children by running this command - this might take some time)
3. _Download gem5 Disk Images and Kernel Binaries_
(note: binaries and images to be downloaded from gem5 recommended Guest Binaries - see https://www.gem5.org/documentation/general_docs/fullsystem/guest_binaries for more details)

`gem5 $ mkdir dist`

`gem5 $ cd dist`

`dist $ wget http://dist.gem5.org/dist/current/arm/aarch-system-201901106.tar.bz2`

`dist $ tar xvf aarch-system-201901106.tar.bz2`
(this creates two folders: disks and binaries)

`dist $ cd disks`

`disks $ wget http://dist.gem5.org/dist/current/arm/disks/ubuntu-18.04-arm64-docker.img.bz2`

`disks $ bzip2 -d ubuntu-18.04-arm64-docker.img.bz2`

`disks $ cd ../../../`

`workspace $ echo "gem5 installed, configured and ready to roll!"`
4. _Download NVmain_

`workspace $ git clone https://github.com/samueltphd/NVmain`
5. _Apply NVmain Patches to gem5 and recompile gem5_

`workspace $ cd gem5`

`gem5 $ git apply ../NVmain/patches/gem5/nvmain2-gem5-11688+`

`gem5 $ scons build/ARM/gem5.opt EXTRAS=../NVmain`

`workspace $ echo "Built NVmain and applied to gem5!"`
6. _Write C/C++ Script_

`workspace $ mkdir src`

`workspace $ cd src`

`src $ echo "int main(int argc, char* argv[]) { return 0; }" > hello_world.c`

`src $ cd ..`

`workspace $ echo "Example C script written, any script can be written here!"`
7. _Compile C/C++ Script for ARM Target_

`workspace $ aarch-linux-gnu-gcc ./src/hello_world.c -o hello_world.arm`

`workspace $ echo "ARM executable produced!"`
8. _Mount Executable onto Disk Image_

`workspace $ mkdir mount-directory`

`workspace $ sudo python gem5/util/gem5img.py mount gem5/dist/disks/ubuntu-18.04-arm64-docker.img mount-directory`

`workspace $ sudo cp hello_world.arm mount-directory`

`workspace $ sudo umount mount-directory`

`workspace $ rmdir mount-directory`

`workspace $ echo "Executable file mounted onto disk image!"`
9. _Run C/C++ Script on gem5 with Non-Volatile Main Memory_
`workspace $ gem5/build/ARM/gem5.opt gem5/configs/example/fs.py --disk-image=gem5/dist/disks/ubuntu-18.04-arm64-docker.img --kernel=gem5/dist/binaries/vmlinux.arm64 --bootloader gem5/dist/binaries/boot.arm64 --mem-type=NVMainMemory --nvmain-config=NVmain/Configs/PCM_ISSCC_2012_4GB.config`
(gem5 system setup output will be visible with several warnings, this is okay)

In a new terminal window

`workspace $ telnet localhost 3456`
(More system setup output will appear -- eventually a command prompt "#" will appear, and then a normal Ubuntu terminal can be used!)

`# ./hello_world.arm`
