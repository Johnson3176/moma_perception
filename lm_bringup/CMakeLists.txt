cmake_minimum_required(VERSION 2.8.3)
project(lm_bringup)
find_package(catkin REQUIRED)
catkin_package()
install(DIRECTORY launch config rviz
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(FILES README.md
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
