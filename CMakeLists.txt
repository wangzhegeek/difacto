cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

project(difacto VERSION 0.1 LANGUAGES C CXX)

# 设置C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 定义依赖路径
set(DEPS_PATH ${CMAKE_CURRENT_SOURCE_DIR}/deps)

# 配置选项
option(USE_CITY "Build with CITY HASH support" ON)
option(USE_LZ4 "Build with LZ4 support" ON)
option(NO_REVERSE_ID "Disable reverse feature ID" OFF)

# 添加路径定义
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/dmlc-core/include
  ${CMAKE_CURRENT_SOURCE_DIR}/ps-lite/include
  ${CMAKE_CURRENT_SOURCE_DIR}/dmlc-core/src
  ${DEPS_PATH}/include
)

# 定义全局编译选项
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fopenmp -fPIC -O3 -ggdb -Wall -finline-functions -DDMLC_LOG_FATAL_THROW=0")

if(NO_REVERSE_ID)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DREVERSE_FEATURE_ID=0")
endif()

# 添加dmlc-core子项目
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/dmlc-core)

# 添加ps-lite子项目
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/ps-lite)

# 编译difacto库
file(GLOB_RECURSE DIFACTO_SRCS
  src/loss/loss.cc
  src/updater.cc
  src/sgd/sgd_updater.cc
  src/sgd/sgd_learner.cc
  src/learner.cc
  src/bcd/bcd_learner.cc
  src/lbfgs/lbfgs_learner.cc
  src/store/store.cc
  src/tracker/tracker.cc
  src/reporter/reporter.cc
  src/data/localizer.cc
  src/reader/batch_reader.cc
)

add_library(difacto STATIC ${DIFACTO_SRCS})

# City Hash 依赖处理
if(USE_CITY)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDIFACTO_USE_CITY=1")
  target_link_libraries(difacto ${DEPS_PATH}/lib/libcityhash.a)
endif()

# LZ4 依赖处理
if(USE_LZ4)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDIFACTO_USE_LZ4=1")
  target_link_libraries(difacto ${DEPS_PATH}/lib/liblz4.a)
endif()

# 添加可执行文件
add_executable(difacto_main src/main.cc)
set_target_properties(difacto_main PROPERTIES OUTPUT_NAME "difacto")
target_link_libraries(difacto_main difacto dmlc pslite ${CMAKE_THREAD_LIBS_INIT})

# 如果需要单元测试，在这里添加
# add_subdirectory(tests) 