How to fix teb_local_planner install on HostPC


```
sudo apt install libsuitesparse-dev
```

make sure your g2o is from official, and cmake g2o with proper settings.



go to teb_local_planner/

edit CMakeLists.txt

comment out following parts:

```
add_executable(test_optim_node src/test_optim_node.cpp)

target_link_libraries(test_optim_node
   teb_local_planner
   ${EXTERNAL_LIBS}
   ${catkin_LIBRARIES}
)
```

and 

```
install(TARGETS test_optim_node
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

then

```
catkin_make
```
