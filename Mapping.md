# Record and replay ros bags for rtabmap

## Recording a bag on Velodyne kobuki (logiler_bringup: branch=develop_velodyne_kobuki)

The following example shows how to record a bag using the kobuki base with the Zed camera and Velodyne Punk.

#### Prepare the base

First start the base with minimal functionality, for that use the record launcher included on the logiler package.

```roslaunch logiler_bringup record_velodyne.launch```

#### Record the bag
On another terminal navigate to the location where the bag is going to be stored (Note: bags can be very big, even more than 200 Gb so make sure to have enough storage).

Decide the topics you want to record, the more topics you use the bigger the bag, too many topics can also affect the framerate. For this example we will use the following:

```
- /camera/depth/camera_info
- /camera/rgb/image_rect_color
- /camera/depth/depth_registered
- /camera/rgb/camera_info
- /joint_states
- /mobile_base/sensors/imu_data
- /mobile_base/odom
- /tf
- /tf_static
- /robot_pose
- /velodyne_points
- /scan
- /zed_scan
```

Once the topics are decided use the following command to start recording the bag.

```
rosbag record /camera/depth/camera_info /camera/rgb/image_rect_color /camera/depth/depth_registered /camera/rgb/camera_info /joint_states /mobile_base/sensors/imu_data /mobile_base/odom /tf /tf_static /robot_pose /velodyne_points /scan /zed_scan
```

or

```
roslaunch logiler_bringup bag.launch
```

Navigate the robot around the area you wish to record, altough Rviz is not required you may want to monitor the camera output, for that you can use Rviz or even rqt_image_view.

## Making a map using a recorded bag

#### Replay the bag
On one terminal run the `roscore`, on another terminal play the bag, you may want to start the bag in paused state so that you control when to start publishing, for that use pause flag, also you can use a higher replay rate for faster mapping.

- `--pause` Start in paused mode.
- `-r FACTOR`, `--rate=FACTOR` Multiply the publish rate by FACTOR.

For this example the bag name is "2019-02-14-02-01-50.bag" and it will be played at 10x rate.

```
rosparam set use_sim_time true;rosbag play --pause --clock 2019-11-12-16-03-23.bag -r 0.5
```

Next on another terminal launch the data recoder from logiler_bringup
```
roslaunch logiler_bringup data_recorder_velodyne.launch
```
Once rtabmap is ready you can "un-pause" the bag by pressing the space bar on the bag's terminal.

To see the map creation you can use rviz.


NOTICE:

once the bag playing is finished, you will `Ctrl+c` to save the rtabmap.db to /home/nvidia/.ros.  Before doing this, open a terminal to run:


```
rosservice call /rtabmap/set_mode_localization
```

this is for safely save the rtabmap.db,  otherwise the database file will cause Dictionary Errors. after the command finished execution you can `Ctrl+c` the `data_recorder_velodyne.launch`.



## "Navigating" a map using a recorded bag
On one terminal run the `roscore`, on another terminal play the bag, you may want to start the bag in paused state so that you control when to start publishing, for that use pause flag, also you can use a higher replay rate for faster mapping.

- `--pause` Start in paused mode.
- `-r FACTOR`, `--rate=FACTOR` Multiply the publish rate by FACTOR.

For this example the bag name is "2019-02-14-02-01-50.bag" and it will be played at 10x rate.

```
rosparam set use_sim_time true
rosbag play --clock --pause 2019-02-14-02-01-50.bag
```

Next on another terminal launch the data recoder from logiler_bringup

```
roslaunch logiler_bringup data_recorder_velodyne.launch localization:=true
```

Once rtabmap is ready you can "un-pause" the bag by pressing the space bar on the bag's terminal. If you want the Yaml file and pgm file, you can run:

```
rosrun map_server map_saver -f mymap
```


