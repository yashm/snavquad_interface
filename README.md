# snavquad_interface

[ATLFlight README](https://github.com/ATLFlight/ATLFlightDocs/blob/master/README.md)

[Get Latest Platform BSP](https://support.intrinsyc.com/projects/snapdragon-flight/wiki/Get_and_install_the_latest_platform_BSP)

```
unzip Flight_3.1.3.1_JFlash.zip
cd Flight_3.1.3_JFlash
./jflash.sh
```

Update DSP firmware
```
unzip Flight_3.1.3.1_qcom_flight_controller_hexagon_sdk_add_on.zip
cd Flight_3.1.3.1_qcom_flight_controller_hexagon_sdk_add_on
./installfcaddon.sh
```

Change the hostname based on vehicle id
```
adb shell
sudo hostname dragon65
vi /etc/hostname
```

Copy snav setup folder onto the board
```
adb push snav_setup /home/linaro/snav_setup

```

Modify /etc/network/interfaces and /etc/wpa_supplicant/wpa_supplicant.conf to connect to WiFi network
```
cd ~/snav_setup
cp interface /etc/network/interface
cp wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant.conf
```

[Put the board in station mode](https://docs.px4.io/en/flight_controller/snapdragon_flight_advanced.html#wifi-settings)
```
/usr/local/qr-linux/wificonfig.sh -s station
```

Reeboot and try ssh 
```
ssh linaro@dragon65
```

[ROS Installation](https://github.com/ATLFlight/ATLFlightDocs/blob/master/SnapdragonROSInstallation.md)
```
sudo apt-get install -f
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu trusty main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
sudo apt-get update
```

ros-indigo-opencv3 has a conflict
```
sudo apt purge --auto-remove libxcb1 libfreetype6
```

```
sudo apt-get install ros-indigo-opencv3 ros-indigo-ros-base
sudo apt-get -f -o Dpkg::Options::="--force-overwrite" install
cd /home
sudo chown -R linaro:linaro linaro
cd ~/
echo "export HOME=/home/linaro/" >> /home/linaro/.bashrc
echo "source /opt/ros/indigo/setup.bash" >> /home/linaro/.bashrc
sudo rosdep init
rosdep update
sudo dpkg -r libhwloc-plugins
sudo dpkg -r ocl-icd-libopencl1:armhf
sudo apt-get install ros-indigo-tf2-ros python-catkin-tools
mkdir ws_ros
cd ws_ros
mkdir src
catkin init
catkin config -j1 -DCMAKE_BUILD_TYPE=RelWithDebInfo -DCMAKE_CXX_FLAGS=-std=c++11 -DCMAKE_C_FLAGS=-std=gnu99
```

Setup Machine Vision SDK and Snapdragon Navigator SDK
```
cd ~/snav_setup
sudo dpkg -i mv_1.1.8_8x74.deb
sudo dpkg -i snav_1.2.58_8x74.deb
sudo dpkg -i snav-esc_1.2.0.deb
sudo cp snapdragon-flight-license.bin /usr/lib
sudo cp snav_params.xml /usr/share/data/adsp/
```

If ESC Update is needed `force_firmware_update` and `force_config_update` have to be set in /usr/share/data/adsp/snav_params.xml and rebooted. On reboot boards updates ESC
Reset params to 0 to disable update on every boot

```
cd /etc/snav
sudo ./configure_vio.sh
```

Static and dynamic calibration (Vehicle will takeoff!!!)
```
sudo stop snav
sudo start snav
sudo snav_calibration_manager -s
sudo snav_calibration_manager -d
```

To check vio and other status
```
snav_inspector
```

Setup other snav ros packages
```
cd ~/ws_ros
mkdir src
cd src
mkdir snav
git clone https://github.com/ATLFlight/ros-examples
git clone https://github.com/ATLFlight/snav_ros
git clone https://github.com/ATLFlight/snav_msgs
git clone https://github.com/ATLFlight/snap_vio
git clone https://github.com/ATLFlight/snap_imu
git clone https://github.com/ATLFlight/snap_cpa
git clone https://github.com/ATLFlight/snap_cam_ros
git clone https://github.com/ATLFlight/qflight_descriptions
git clone https://github.com/ATLFlight/snap_msgs
git clone https://github.com/ATLFlight/dfs-ros-example
```


```
cd ~/ws_ros/src
git clone https://github.com/ros-perception/vision_opencv.git
cd vision_opencv
git checkout origin/indigo
cd ~/ws_ros/src
git clone https://github.com/tdinesh/apriltag.git
git checkout -t origin/devel_snav

```

OpenCV3 deb file (Not needed)
```
cd ~/Downloads
wget http://px4-tools.s3.amazonaws.com/opencv3_20160222-1_armhf.deb
```

```
roslaunch ddk_launch snav_interface.launch
```
