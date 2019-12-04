# Configure Jetson TX2

## Terminology
workstation PC: basically it means your PC at hand.  
TX2 PC / TX2: it means the mini PC on TX2.  
eMMC: hardisk on TX2 about 30GB...  

## Flash Jetson TX2 / TX1 
- download and run **JetPack-L4T-3.3-linux-x64_b39.run**
 Link :  [https://developer.nvidia.com/embedded/dlc/jetpack-l4t-3_3](https://developer.nvidia.com/embedded/dlc/jetpack-l4t-3_3)

Connect JetSon and your workstation PC by USB cable (Jetson developer package includes the cable), 
which means the Ubuntu OS will be install on JetSon by runing the *JetPack-L4T-3.3-linux-x64_b39.run* on your 
workstation PC, Then run command:

```
chmod +x JetPack-L4T-3.3-linux-x64_b39.run
./JetPack-L4T-3.3-linux-x64_b39.run
```

User will see some gui like the following 

(https://docs.nvidia.com/jetson/jetpack/content/jetpack/4.1.1/img/jetpack_1_welcome.01_700x515.png)

*WARNING!!!  JETSON MUST BE LAN CABLED WITH THE NET， OTHERWISE， CUDA etc WILL NOT BE INSTALLED......*

*HOWEVER*, If you forget the lan cable, it also could be fixed by runing  

```
./JetPack-L4T-3.3-linux-x64_b39.run
```  

and you can sellect not to flash Jetson only to install CUDA kit. During the installation , in my case I chose *wireless network* on my HOST PC, and a pop-up Xterms window will instruct you to put your Jetson into Force USB Recovery Mode
so you can flash your OS by doing what it requests.

*WARNING!!! You must set the same net work with host and TX2. ( because we used mesh wifi (white router) and access point (black router), we currently use the white router for Host PC so lan cable must be connect with whiter router. )*

1. Power down the device. remove the AC adaptor from Jetson. Jetson must be Power off.
2. Connect the Micro-B plug on the USB cable on the Jetson and your workstation PC.
3. Connect the adaptor on Jetson
4. One click the POWER button. Press and hold the FORCE RESET button; wait two seconds,
   and release the FORCE RECOVERY button;
5. When jetson is in recovery mode, lsusb command on workstation PC will list a line of
   "NVidia Corp"

After a while, ubuntu 16.04 will be installed on Jetson TX2. If everything set well, CUDA9.0, cuDNN, and OpenCV 3.3.1-dev will be installed on TX2 as well.

*in order to ssh connect Jetson easier, a static IP setting is highly recommended*

- Set about the Static IP address:
on TX2 
  Edit Connections-->IPV4 Settings-->Method:manual-->Address:192.168.xx.xxx-->Netmask:24-->GateWay:192.168.11.1-->DNS servers:8.8.8.8


## Do the tx2UsbFix  
#############################  
    Compile the kernel  
#############################  
Use the scripts provided by jetsonhacks  
1). get the git  
```
$ cd ~/  
$ git clone http://github.com/jetsonhacks/buildJetsonTX2Kernel.git 
```
2). Enter the repository folder  
```
$ cd buildJetsonTX2Kernel
```
3). Get the kernel sources  
```
$ ./getKernelSources.sh
```
the process will stop when it goes to scripts/kconfig/qconf Kconfig; click " F " like button then click " I I "like button , then the window will be devided into 2 columns.  

```(left column) USB support [✓] --> (left column) USB Serial Converter support[・] --> (right column) USB Generic Serial Driver [✓] --> (right column) USB Serial Simple Driver [・] --> Save```



4). Edit the tegra18_defconfig file located in /usr/src/kernel/kernel-4.4/arch/arm64/configs/tegra18_defconfig  
   Add 'CONFIG_SPI_SPIDEV=m' bellow 'CONFIG_SPI_TEGRA114=y' eg:
```
      CONFIG_SPI=y
      CONFIG_SPI_TEGRA114=y
   --> CONFIG_SPI_SPIDEV=m <--
      CONFIG_QSPI_TEGRA186=y
```
5). Build the Kernel
```
$ cd /usr/src/kernel/kernel-4.4
$ sudo make tegra18_defconfig
```
6). Modify the file /usr/src/kernel/t18x/drivers/pinctrl/pinctrl-tegra186-padctl.c  
   Remove the following lines from the ```tegra186_enable_vbus_oc(struct phy *phy)``` function
```
reg = padctl_readl(padctl, XUSB_PADCTL_VBUS_OC_MAP);
reg |= VBUS_ENABLE(pin);
padctl_writel(padctl, reg, XUSB_PADCTL_VBUS_OC_MAP);
```
7). Build the kernel using the jetsonhacks repository  
```
$ cd ~/buildJetsonTX2Kernel
$ ./makeKernel.sh
```
8). Ensure the SPIDev Kernel module is copied to /lib/modules  
```
$ sudo cp /usr/src/kernel/kernel-4.4/drivers/spi/spidev.ko /lib/modules/$(uname -r)/kernel/drivers/
```
9). Update module dependencies and kernel image  
```
$ sudo depmod
$ ./copyImage.sh
```
10). Reboot  
11). verify the SPIDev kernel  
```
$ cd /lib/modules/$(uname -r)
$ cat modules.dep | grep spidev
```

eg. kernel/drivers/spi/spidev.ko  

#############################  
 Modifying the Device Tree  
#############################  
1. Install DTC Tool
```
$ sudo apt update
$ sudo apt install device-tree-compiler
```
2. Decompile Device Tree  
```
$ cd /boot/dtb/
$ sudo dtc -I fs -O dts -o extracted_proc.dts /proc/device-tree
```
3. check _extracted_proc.dts_, Ensure the following patch is present  
```
   spi@3240000{
       compatible = "nvidia,tegra186-spi";
       reg = <0x0 0x3240000 0x0 0x10000>;
       ....
       ....
       ....
       linux,phandle = <0x7d>;
   -->  spi@0 {
         reg = <0x0>;
         nvidia,tx-clk-tap-delau = <0x0>;
         compatible = "spidev";
         nvidia,cs-setup-clk-count = <0x1e>;
         nvidia,cs-hold-clk-count = <0x1e>;
         spi-max-frequency = <0x1312d00>;
         nvidia,enable-hw-based-cs;
         nvidia,rx-clk-tap-delay = <0x1f>;
      }; <--
    };
 ```
4. Remove the regulator @5  
```regulator@5 {...};```
3. Recompile the Device Tree  
```
$ cd /boot/dtb/
$ sudo dtc -I dts -O dtb -o tegra186-quill-p3310-1000-c03-00-base.dtb extracted_proc.dts
```
4. enable FDT in /boot/extlinux/extlinux.conf  
```
      LINUX /boot/Image
    --> FDT /boot/dtb/tegra186-quill-p3310-1000-c03-00-base.dtb <--
      APPEND ${cbootargs} root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4
```
5. Reboot  
6. Verify SPIDev Device  
```$ ls /dev/spi*```

eg. /dev/spidev3.0  

#############################  
  Restart usb on bootup  
#############################  

1. Create the script to restart the USB on /usr/local/bin/usb_restart.sh  
```$ sudo vi /usr/local/bin/usb_restart.sh```
```
-->
#!/bin/bash
sleep 10
echo "Exporting gpio"
echo 349 > /sys/class/gpio/export
echo "GPIO 349 exported"
sleep 1
echo "turning OFF USB"
echo out > /sys/class/gpio/gpio349/direction && echo 0  > /sys/class/gpio/gpio349/value
echo "USB OFF"
sleep 5
echo "turning ON USB"
echo out > /sys/class/gpio/gpio349/direction && echo 1  > /sys/class/gpio/gpio349/value
echo "USB ON"
exit
<--
```
2. Add the right permissions  
```$ sudo chmod 744 /usr/local/bin/usb_restart.sh```
3. Create the service file on /etc/systemd/system/usbRestart.service  
```$ sudo vi /etc/systemd/system/usbRestart.service```
```
-->
[Service]
ExecStart=/usr/local/bin/usb_restart.sh

[Install]
WantedBy=default.target
<--
```
4. Add the right permissions to the service file  
```$ sudo chmod 664 /etc/systemd/system/usbRestart.service```
5. Restart the service daemon  
```$ sudo systemctl daemon-reload```
6. Enable the new service  
```$ sudo systemctl enable usbRestart.service```




## Set the SSD disk as the root disk

1. To format the SSD disk to EXT4 format. 
Use ubuntu OS application: *Disks*


2. Copy eMMC to SSD
Copy exactly the eMMC hard disk contents to the SSD you going to use on the TX2.
before copying, make sure the SSD path, in my case it is: */media/nvidia/ssd*

```
sudo cp -ax / /media/nvidia/ssd
```

3. Configure boot: from eMMC to SSD
[install Samsung SSD on JetSon](https://www.jetsonhacks.com/2017/01/28/install-samsung-ssd-on-nvidia-jetson-tx1/)

We must edit the eMMC's **/boot/extlinux/extlinux.conf** file.

origin **extlinux.conf** file is like:

```
TIMEOUT 30
DEFAULT primary

MENU TITLE p2771-0000 eMMC boot options

LABEL primary
      MENU LABEL primary kernel
      LINUX /boot/Image
      APPEND ${cbootargs} root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4

```
We would like to backup the origin file:
```
sudo cp /boot/extlinux/extlinux.conf /boot/extlinux/extlinux.conf.bkup
```
assume your SSD disk is named as: *ssd*
and the disk on root is: /dev/sda <------- *This should be carefully confirmed by the user*.
write your */boot/extlinux/extlinux.conf* file like this:

``` 
TIMEOUT 30
DEFAULT ssd
MENU TITLE p2771-0000 eMMC boot options
LABEL primary
      MENU LABEL primary kernel
      LINUX /boot/Image
      FDT /boot/dtb/tegra186-quill-p3310-1000-c03-00-base.dtb
      APPEND ${cbootargs} root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4
LABEL ssd
      MENU LABEL primary kernel
      LINUX /boot/Image
      FDT /boot/dtb/tegra186-quill-p3310-1000-c03-00-base.dtb
      APPEND ${cbootargs} root=/dev/sda rw rootwait rootfstype=ext4
```

Then reboot your TX2

```
sudo shutdown -r now
```

And the TX2 will boot up with the ssd disk not the origin internal SD card on Jetson TX2.


## [Install OPenCV 3.4 for TX2](https://www.jetsonhacks.com/2017/04/05/build-opencv-nvidia-jetson-tx2/)

- Install OpenCV 3.4 for TX2
First download opencv_contrib
If you want to enable markdetection of apriltags2, you have to install opencv 3.4.2 
```
cd ~
#get opencv source code
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.4.2 #for enable markdetection.
mkdir build
cd ..
cd ~
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.4.2 #for enable markdetection.
cd ..
```
and you should uninstall / remove the Opencv 3.3.1 installed by Jetson Pack 3.3

```
sudo apt install mlocate
sudo updatedb
sudo locate opencv_
```
there will be a lot of opencv_* and carefully remove them....


First, user should run following codes on your bash terminal:
``` 
cd ~
git clone https://github.com/jetsonhacks/buildOpenCVTX2.git
cd buildOpenCVTX2
```
find out the cmake setting parts and replace with:

```
time cmake \
-D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_CUDA=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules  \
-D BUILD_SHARED_LIBS=ON \
-D WITH_GTK=ON \
-D BUILD_EXAMPLES=OFF ..  
```

previous version I set as ```-D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \```
however, there will be errors like ```C++11 is not supported......```

Then run the script: (it may request the root passwd)

```
./buildOpenCV.sh
cd ..
```

**OR**

If you want to manually compile the source do like this: open a terminal and we suppose you got the s


```bash
cd ~
#get opencv source code
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.4.1 # or 3.4.2  you decide
mkdir build
cd ..
#get opencv_contrib
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.4.1 # or 3.4.2  you decide
cd ..

cd ~/opencv/build

# Jetson TX2
ARCH_BIN=6.2
# Jetson TX1
# ARCH_BIN=5.3
INSTALL_DIR=/usr/local
# Download the opencv_extras repository
# If you are installing the opencv testdata, ie
#  OPENCV_TEST_DATA_PATH=../opencv_extra/testdata
# Make sure that you set this to YES
# Value should be YES or NO
DOWNLOAD_OPENCV_EXTRAS=NO
# Source code directory
OPENCV_SOURCE_DIR=$HOME
WHEREAMI=$PWD
CLEANUP=true
CMAKE_INSTALL_PREFIX=$INSTALL_DIR


cmake \
-D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D WITH_CUDA=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D INSTALL_C_EXAMPLES=ON \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules  \
-D BUILD_SHARED_LIBS=ON \
-D WITH_GTK=ON \
-D BUILD_EXAMPLES=OFF ..  

make
sudo make install
```

## Install ZED camera SDK

Make Sure your CUDA version must be *CUDA9.0*. Login your TX2 Ubuntu OS, plugin your ZED camera on TX2.

Open terminal and run:

```
mkdir ~/ZED_SDK
cd ~/ZED_SDK
wget https://www.stereolabs.com/developers/downloads/ZED_SDK_JTX2_JP3.2_v2.7.1.run
chmod +x ZED_SDK_JTX2_JP3.2_v2.7.1.run
./ZED_SDK_JTX2_JP3.2_v2.7.1.run
```

after this user will find folder like */usr/local/zed* 

and to testify, user should run:

```
cd /usr/local/zed/tools
./ZED\ Explorer
```



## [install ROS and logiler packages on the Jetson Machine's SSD](https://www.jetsonhacks.com/2017/03/27/robot-operating-system-ros-nvidia-jetson-tx2/)
The SSD will be the primary disk from now on.

- install ROS
run the following codes on your bash terminal:

```
cd ~
git clone https://github.com/jetsonhacks/installROSTX2.git
cd installROSTX2
./installROS.sh
./setupCatkinWorkspace.sh ros/catkin_ws 
cd ..
```

- install logiler packages on TX2

*the version of logiler should be exactly taken*, user might be asked to input your github account to check whether it was authorized or not
assume you have the catkin workspace named */home/nvidia/catkin_ws*
open bash terminal and run:


```
cd ~/catkin_ws/src
git clone -b feature/updatedSdk --single-branch https://github.com/SeaosRobotics/logiler_bringup.git
git clone -b feature/extImu --single-branch https://github.com/SeaosRobotics/key_cart.git
git clone https://github.com/SeaosRobotics/pipeline_planner.git
git clone -b feature/short --single-branch https://github.com/SeaosRobotics/logiler_navigation.git
git clone -b develop --single-branch https://github.com/SeaosRobotics/zed-ros-wrapper.git
git clone -b feature/short --single-branch https://github.com/SeaosRobotics/logiler_description.git
cd ~/ros/catkin_ws
rosdep install --from-paths src --ignore-src --rosdistro kinetic -y
sudo apt install ros-kinetic-kobuki-msgs
```

for the zed-ros-wrapper folder, it must copy from other keycart like (222, 223, 224), current github version has bugs.  


Here we do not need to catkin_make, because ```rosdep install --from-paths src --ignore-src --rosdistro kinetic -y``` will install ros-kinetic-rtabmap, this should be removed, we need compile rtabmap in next step by ourself.
so you can continue to next step.  



- install Rtabmap / Rtabmap_ros on TX2  
BEFORE INSTALL ```git checkout 0.18.1``` is necessary.
Please follow this https://github.com/introlab/rtabmap_ros/blob/master/README.md

fininally, make sure you did:
```
roscd
cd ..
catkin_make -j1
```
