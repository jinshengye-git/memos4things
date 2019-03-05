# compile and install Rtabmap / rtabmap_ros

```bash
echo 'install dependencies'

sudo apt-get install ros-melodic-rtabmap ros-melodic-rtabmap-ros
sudo apt-get remove  ros-melodic-rtabmap ros-melodic-rtabmap-ros



cd ~
git clone https://bitbucket.org/gtborg/gtsam.git
cd gtsam/
git checkout 4.0.0-alpha2
mkdir build
cd build
cmake GTSAM_USE_SYSTEM_EIGEN=ON ..
make
sudo make install

cd ~
git clone https://github.com/RainerKuemmerle/g2o.git
cd g2o/
git checkout dynamic_vertex
sudo apt install libeigen3-dev
sudo apt install libsuitesparse-dev qtdeclarative5-dev qt5-qmake
sudo apt install libqglviewer-headers
mkdir build
cd build
cmake -DBUILD_WITH_MARCH_NATIVE=OFF ..
make
sudo makeinstall
```


after `sudo make install` g2o

run

```bash
cd /opt/ros/melodic/lib 
sudo rm libg2o_cli.so; sudo ln /usr/local/lib/libg2o_cli.so ;
sudo rm libg2o_core.so; sudo ln /usr/local/lib/libg2o_core.so ;
sudo rm libg2o_csparse_extension.so; sudo ln /usr/local/lib/libg2o_csparse_extension.so ;
sudo rm libg2o_ext_freeglut_minimal.so; sudo ln /usr/local/lib/libg2o_ext_freeglut_minimal.so ;
sudo rm libg2o_incremental.so; sudo ln /usr/local/lib/libg2o_incremental.so ;
sudo rm libg2o_interactive.so; sudo ln /usr/local/lib/libg2o_interactive.so ;
sudo rm libg2o_interface.so; sudo ln /usr/local/lib/libg2o_interface.so ;
sudo rm libg2o_opengl_helper.so; sudo ln /usr/local/lib/libg2o_opengl_helper.so ;
sudo rm libg2o_parser.so; sudo ln /usr/local/lib/libg2o_parser.so ;
sudo rm libg2o_simulator.so; sudo ln /usr/local/lib/libg2o_simulator.so ;
sudo rm libg2o_solver_cholmod.so; sudo ln /usr/local/lib/libg2o_solver_cholmod.so ;
sudo rm libg2o_solver_csparse.so; sudo ln /usr/local/lib/libg2o_solver_csparse.so ;
sudo rm libg2o_solver_dense.so; sudo ln /usr/local/lib/libg2o_solver_dense.so ;
sudo rm libg2o_solver_eigen.so; sudo ln /usr/local/lib/libg2o_solver_eigen.so ;
sudo rm libg2o_solver_pcg.so; sudo ln /usr/local/lib/libg2o_solver_pcg.so ;
sudo rm libg2o_solver_slam2d_linear.so; sudo ln /usr/local/lib/libg2o_solver_slam2d_linear.so ;
sudo rm libg2o_solver_structure_only.so; sudo ln /usr/local/lib/libg2o_solver_structure_only.so ;
sudo rm libg2o_stuff.so; sudo ln /usr/local/lib/libg2o_stuff.so ;
sudo rm libg2o_types_data.so; sudo ln /usr/local/lib/libg2o_types_data.so ;
sudo rm libg2o_types_icp.so; sudo ln /usr/local/lib/libg2o_types_icp.so ;
sudo rm libg2o_types_sba.so; sudo ln /usr/local/lib/libg2o_types_sba.so ;
sudo rm libg2o_types_sclam2d.so; sudo ln /usr/local/lib/libg2o_types_sclam2d.so ;
sudo rm libg2o_types_sim3.so; sudo ln /usr/local/lib/libg2o_types_sim3.so;
sudo rm libg2o_types_slam2d_addons.so; sudo ln /usr/local/lib/libg2o_types_slam2d_addons.so ;
sudo rm libg2o_types_slam2d.so; sudo ln /usr/local/lib/libg2o_types_slam2d.so ;
sudo rm libg2o_types_slam3d_addons.so; sudo ln /usr/local/lib/libg2o_types_slam3d_addons.so ;
sudo rm libg2o_types_slam3d.so; sudo ln /usr/local/lib/libg2o_types_slam3d.so ;
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


