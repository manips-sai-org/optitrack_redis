cmake_minimum_required(VERSION 3.1)

project(optitrack_redis_driver)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Set common source
set(OPTITRACK_COMMON_SOURCE
	${PROJECT_SOURCE_DIR}/RedisClient.cpp
	${PROJECT_SOURCE_DIR}/OptiTrackClient.cpp
)

# include Eigen
set(EIGEN3_INCLUDE_DIR $ENV{EIGEN3_INCLUDE_DIR})
if(NOT EIGEN3_INCLUDE_DIR)
	find_package(Eigen3 QUIET)
	# TODO: Find automatic way to find for Ubuntu 14.04
	if(NOT EIGEN3_INCLUDE_DIR)
		set(EIGEN3_INCLUDE_DIR /usr/include/eigen3)
	endif()
endif()
include_directories(${EIGEN3_INCLUDE_DIR})

# - hiredis
find_library(HIREDIS_LIBRARY hiredis)

# - jsoncpp
if(CMAKE_SYSTEM_NAME MATCHES Darwin) 
	find_library(JSONCPP_LIBRARY jsoncpp)
elseif(CMAKE_SYSTEM_NAME MATCHES Linux)
	find_package(PkgConfig REQUIRED)
	pkg_check_modules(JSONCPP jsoncpp)
	include_directories(${JSONCPP_INCLUDE_DIRS})
	link_directories(${JSONCPP_LIBRARY_DIRS})
	set(JSONCPP_LIBRARY ${JSONCPP_LIBRARIES})
endif()

# # - natnetlinux
find_package(Boost REQUIRED COMPONENTS system thread)
include_directories(${Boost_INCLUDE_DIRS})
link_libraries(${Boost_LIBRARIES})

set(OPTITRACK_COMMON_LIBRARIES
	${HIREDIS_LIBRARY}
	${JSONCPP_LIBRARY}
)


# create an executable
ADD_EXECUTABLE (optitrack_redis
	${OPTITRACK_COMMON_SOURCE}
	optitrack_main.cpp
)

# and link the library against the executable
TARGET_LINK_LIBRARIES (optitrack_redis
	${OPTITRACK_COMMON_LIBRARIES}
)
