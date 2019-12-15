# Rtabmap-install Manual

## Dependencies
```
sudo apt-get update
sudo apt-get install libsqlite3-dev libpcl-dev git cmake libproj-dev libqt5svg5-dev
# google-glog + gflags
sudo apt-get install libgoogle-glog-dev
# BLAS & LAPACK
sudo apt-get install libatlas-base-dev
# Eigen3
sudo apt-get install libeigen3-dev
# SuiteSparse and CXSparse (optional)
# - If you want to build Ceres as a *static* library (the default)
#   you can use the SuiteSparse package in the main Ubuntu package
#   repository:
sudo apt-get install libsuitesparse-dev
```

### OpenCV
```
cd ~
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd ~/opencv && git checkout 3.4.6
cd ~/opencv_contrib && git checkout 3.4.6
cd ~/opencv
mkdir build && cd build
cmake-gui ..
```

Make sure you have done:

```
 - WITH_CUDA  ON
 - OPENCV_ENABLE_NON_FREE  ON
 - OPENCV_EXTRA_MODULES_PATH   set to opencv_contrib/modules/
 - WITH_QT  ON
 - BUILD_opencv_cudacodec    OFF
```
Make sure on the outputs of CMake-gui you can find:

OpenCV modules:
- To be built:                 aruco bgsegm bioinspired calib3d ccalib core cudaarithm cudabgsegm cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev **cvv** datasets dnn dnn_objdetect dpm face features2d flann freetype fuzzy hdf hfs highgui img_hash imgcodecs imgproc line_descriptor ml objdetect optflow phase_unwrapping photo plot python2 python3 reg rgbd saliency **sfm** shape stereo stitching structured_light superres surface_matching text tracking ts video videoio videostab viz xfeatures2d ximgproc xobjdetect xphoto
- Disabled:                    cudacodec world
- Disabled by dependency:      -
- Unavailable:                 cnn_3dobj java js matlab ovis
- Applications:                tests perf_tests apps
- Documentation:               NO
- Non-free algorithms:         NO




### install cvsba
download from https://sourceforge.net/projects/cvsba/files/

```
sudo apt-get install liblapack-dev libf2c2-dev
tar -zxvf cvsba-1.0.0.tgz 
mkdir cvsba-1.0.0.tgz/build
cd cvsba-1.0.0.tgz/build
cmake ..
make
make install
sudo mkdir /usr/local/lib/cmake/cvsba 
sudo mv /usr/local/lib/cmake/Findcvsba.cmake /usr/local/lib/cmake/cvsba/cvsbaConfig.cmake
```

### install g2o

```
cd ~
git clone https://github.com/RainerKuemmerle/g2o.git 
cd ~/g2o
mkdir build
cd build
cmake -DBUILD_WITH_MARCH_NATIVE=OFF -DBUILD_CSPARSE=ON -DG2O_BUILD_APPS=OFF -DG2O_BUILD_EXAMPLES=ON -DG2O_USE_OPENGL=OFF ..
make -j12
sudo make install
```

### install GTSAM

```
cd ~
git clone --branch 4.0.0-alpha2 https://github.com/borglab/gtsam.git gtsam-4.0.0-alpha2
cd ~/gtsam-4.0.0-alpha2
mkdir build
cd build
cmake -DGTSAM_USE_SYSTEM_EIGEN=ON -DGTSAM_BUILD_EXAMPLES_ALWAYS=OFF -DGTSAM_BUILD_TESTS=OFF -DGTSAM_BUILD_UNSTABLE=OFF ..
make -j12
sudo make install
```

### install octomap
```
cd ~
git clone 	https://github.com/OctoMap/octomap.git
cd /octomap
git checkout qt5-support
mkdir build
cd build
cmake ..
make -j12
sudo make install
```

### install libpointmatcher

```
sudo apt-get install libboost-all-dev
sudo apt-get install libeigen3-dev
sudo apt-get install libyaml-cpp-dev
sudo apt-get install doxygen

cd ~
git clone https://github.com/ethz-asl/libnabo.git
cd ~/libnabo/
mkdir build
cd build/
cmake -DCMAKE_BUILD_TYPE=RelWithDebInfo ..
make -j7
sudo make install


cd ~
git clone https://github.com/ethz-asl/libpointmatcher.git
cd ~/libpointmatcher/
mkdir build
cd build/
cmake ..
make -j7
sudo make install
```

### install rtabmap

```
cd ~
git clone https://github.com/introlab/rtabmap.git
cd ~/rtabmap/build
cmake ..
make -j12
sudo make install
```
