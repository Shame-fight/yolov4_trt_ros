cmake_minimum_required(VERSION 3.0.2)
project(yolov4_trt)

find_package(OpenCV 3.0 REQUIRED)
find_package(jetson-utils REQUIRED)
find_package(jetson-inference REQUIRED)
find_package(CUDA REQUIRED)

catkin_python_setup()

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/package.xml ${CMAKE_CURRENT_SOURCE_DIR}/package.xml COPYONLY)

find_package(catkin REQUIRED COMPONENTS
  image_transport
  rospy
  sensor_msgs
  vision_msgs
  cv_bridge
  message_generation
)

add_message_files(
  FILES
  Detector2D.msg
  Detector2DArray.msg
  BoundingBox2D.msg
  ObjectHypothesis.msg
)

generate_messages(DEPENDENCIES std_msgs vision_msgs)
	 
catkin_package(
  LIBRARIES ros_deep_learning_nodelets
  CATKIN_DEPENDS nodelet rospy image_transport sensor_msgs vision_msgs cv_bridge message_runtime
)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")	# enable c++11 (TensorRT requirement)
include_directories(${catkin_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS})
add_definitions(-DROS1)

set(common_src src/image_converter.cpp src/ros_compat.cpp)

add_executable(video_source src/node_video_source.cpp ${common_src})
target_link_libraries(video_source ${catkin_LIBRARIES} jetson-inference)

if(catkin_EXPORTED_LIBRARIES)
  add_dependencies(ros_deep_learning_nodelets ${catkin_EXPORTED_LIBRARIES})
endif()

catkin_install_python(PROGRAMS trt_yolo_v4.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)