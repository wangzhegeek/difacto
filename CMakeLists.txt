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

# 添加测试目标
file(GLOB_RECURSE TEST_SRCS
  tests/cpp/main.cc
  tests/cpp/spmv_test.cc
  tests/cpp/spmm_test.cc
  tests/cpp/spmt_test.cc
  tests/cpp/compressed_row_block_test.cc
  tests/cpp/find_position_test.cc
  tests/cpp/localizer_test.cc
  tests/cpp/kv_match_test.cc
  tests/cpp/kv_union_test.cc
  tests/cpp/sgd_learner_test.cc
  tests/cpp/bcd_learner_test.cc
  tests/cpp/lbfgs_learner_test.cc
  tests/cpp/lbfgs_twoloop_test.cc
  tests/cpp/batch_reader_test.cc
  tests/cpp/fm_loss_test.cc
  tests/cpp/logit_loss_delta_test.cc
  tests/cpp/data_store_test.cc
)

add_executable(difacto_tests ${TEST_SRCS})
target_link_libraries(difacto_tests difacto dmlc pslite gtest gtest_main pthread ${CMAKE_THREAD_LIBS_INIT})

# 如果需要单元测试，在这里添加
# add_subdirectory(tests) 