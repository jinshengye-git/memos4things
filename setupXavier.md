# compile and install Rtabmap / rtabmap_ros  

```bash
echo 'install dependencies'

sudo apt-get install ros-melodic-rtabmap ros-melodic-rtabmap-ros
sudo apt-get remove  ros-melodic-rtabmap ros-melodic-rtabmap-ros

#install OpenCV 3.4.2 version..
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

cd ~
git clone https://bitbucket.org/gtborg/gtsam.git
cd gtsam/
git checkout 4.0.0-alpha2
mkdir build
cd build
cmake -DGTSAM_USE_SYSTEM_EIGEN=ON ..
make
sudo make install

cd ~
git clone https://github.com/RainerKuemmerle/g2o.git
cd g2o/
git checkout master
sudo apt install libeigen3-dev
sudo apt install libsuitesparse-dev qtdeclarative5-dev qt5-qmake
sudo apt install libqglviewer-headers
mkdir build
cd build
cmake -DBUILD_WITH_MARCH_NATIVE=OFF ..
make
sudo make install
```


after `sudo make install` g2o  

run  

```bash
cd /opt/ros/melodic/lib 
sudo rm libg2o_cli.so; sudo ln -s /usr/local/lib/libg2o_cli.so libg2o_cli.so;
sudo rm libg2o_core.so; sudo ln -s /usr/local/lib/libg2o_core.so libg2o_core.so;
sudo rm libg2o_csparse_extension.so; sudo ln -s /usr/local/lib/libg2o_csparse_extension.so libg2o_csparse_extension.so;
sudo rm libg2o_ext_freeglut_minimal.so; sudo ln -s /usr/local/lib/libg2o_ext_freeglut_minimal.so libg2o_ext_freeglut_minimal.so;
sudo rm libg2o_incremental.so; sudo ln -s /usr/local/lib/libg2o_incremental.so libg2o_incremental.so;
sudo rm libg2o_interactive.so; sudo ln -s /usr/local/lib/libg2o_interactive.so libg2o_interactive.so;
sudo rm libg2o_interface.so; sudo ln -s /usr/local/lib/libg2o_interface.so libg2o_interface.so;
sudo rm libg2o_opengl_helper.so; sudo ln -s /usr/local/lib/libg2o_opengl_helper.so libg2o_opengl_helper.so;
sudo rm libg2o_parser.so; sudo ln -s /usr/local/lib/libg2o_parser.so libg2o_parser.so;
sudo rm libg2o_simulator.so; sudo ln -s /usr/local/lib/libg2o_simulator.so libg2o_simulator.so;
sudo rm libg2o_solver_cholmod.so; sudo ln -s /usr/local/lib/libg2o_solver_cholmod.so libg2o_solver_cholmod.so;
sudo rm libg2o_solver_csparse.so; sudo ln -s /usr/local/lib/libg2o_solver_csparse.so libg2o_solver_csparse.so;
sudo rm libg2o_solver_dense.so; sudo ln -s /usr/local/lib/libg2o_solver_dense.so libg2o_solver_dense.so;
sudo rm libg2o_solver_eigen.so; sudo ln -s /usr/local/lib/libg2o_solver_eigen.so libg2o_solver_eigen.so;
sudo rm libg2o_solver_pcg.so; sudo ln -s /usr/local/lib/libg2o_solver_pcg.so libg2o_solver_pcg.so;
sudo rm libg2o_solver_slam2d_linear.so; sudo ln -s /usr/local/lib/libg2o_solver_slam2d_linear.so libg2o_solver_slam2d_linear.so;
sudo rm libg2o_solver_structure_only.so; sudo ln -s /usr/local/lib/libg2o_solver_structure_only.so libg2o_solver_structure_only.so;
sudo rm libg2o_stuff.so; sudo ln -s /usr/local/lib/libg2o_stuff.so libg2o_stuff.so;
sudo rm libg2o_types_data.so; sudo ln -s /usr/local/lib/libg2o_types_data.so libg2o_types_data.so;
sudo rm libg2o_types_icp.so; sudo ln -s /usr/local/lib/libg2o_types_icp.so libg2o_types_icp.so;
sudo rm libg2o_types_sba.so; sudo ln -s /usr/local/lib/libg2o_types_sba.so libg2o_types_sba.so;
sudo rm libg2o_types_sclam2d.so; sudo ln -s /usr/local/lib/libg2o_types_sclam2d.so libg2o_types_sclam2d.so;
sudo rm libg2o_types_sim3.so; sudo ln -s /usr/local/lib/libg2o_types_sim3.so libg2o_types_sim3.so;
sudo rm libg2o_types_slam2d_addons.so; sudo ln -s /usr/local/lib/libg2o_types_slam2d_addons.so libg2o_types_slam2d_addons.so;
sudo rm libg2o_types_slam2d.so; sudo ln -s /usr/local/lib/libg2o_types_slam2d.so libg2o_types_slam2d.so;
sudo rm libg2o_types_slam3d_addons.so; sudo ln -s /usr/local/lib/libg2o_types_slam3d_addons.so libg2o_types_slam3d_addons.so;
sudo rm libg2o_types_slam3d.so; sudo ln -s /usr/local/lib/libg2o_types_slam3d.so libg2o_types_slam3d.so;
```





when `make` the rtabmap it might be shown some ERROR saids: can not find `....g2o/g2o/solvers/linear_solver_eigen.h`  
try to find it from g2o source code folder (from github)  
```bash
sudo cp -r /ssd1tb/home/nvidia/app/g2o/g2o/solvers/eigen /usr/local/include/g2o/solvers  
```


try to compile and install rtabmap:  
```
cd ~
git clone https://github.com/introlab/rtabmap.git rtabmap
cd rtabmap/build
cmake ..  [<---double dots included]
make
sudo make install


roscd; cd ../src;
git clone https://github.com/introlab/rtabmap_ros.git src/rtabmap_ros
catkin_make -j1
```


### about ROS `catkin_make`  

when I tried to `catkin_make`  
it saids   

```bash
CMake Error at /opt/ros/melodic/share/cv_bridge/cmake/cv_bridgeConfig.cmake:113 (message):
  Project 'cv_bridge' specifies '/usr/include/opencv' as an include dir,
  which is not found.

.... ....

and

.... ....

*** No rule to make target '/usr/lib/aarch64-linux-gnu/libopencv_core.so.3.2.0', needed by '/home/nvidia/catkin_ws/devel/lib/rtabmap_ros/rtabmap'.  Stop.
 
```

run  

```bash
sudo ln -s /usr/local/include/opencv /usr/include/opencv
sudo ln -s /usr/local/lib/libopencv_core.so.3.4.1 /usr/lib/aarch64-linux-gnu/libopencv_core.so.3.2.0
```

then `catkin_make` again it goes fine.  

### about logiler packages:  

first run:  

```
roscd
cd ..
rosdep install -r -y --from-paths src --ignore-src
```

Then do `catkin_make` you will find _not found_ packages. we have to find them mainly from github.  


```bash
roscd
cd ../src
git clone https://github.com/yujinrobot/yujin_ocs.git
cd yujin_ocs
git checkout release/0.8-melodic
roscd
rosdep install -r -y --from-paths src --ignore-src
sudo apt install ros-melodic-kobuki-*

sudo apt install ros-melodic-yujin-ocs
sudo apt install ros-melodic-tf*
git clone https://github.com/ros-planning/navigation.git
cd navigation
git reset --hard 8665d81a20a1ecd45811ac6195a9c0d1a472080b    #<--according to https://github.com/paulbovbel/frontier_exploration/issues/38-->
cd ..

### teb_local_planner I installed from `sudo apt install ros-melodic-teb-local-planner` 
### git clone https://github.com/rst-tu-dortmund/teb_local_planner.git
### cd teb_local_planner
### git checkout melodic-devel
### cd ..
### 
catkin_make   # Xavier no need to -j1
```
