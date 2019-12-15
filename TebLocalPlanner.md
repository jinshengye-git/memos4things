How to fix teb_local_planner install on HostPC

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
