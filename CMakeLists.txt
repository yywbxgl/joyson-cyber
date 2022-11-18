cmake_minimum_required(VERSION 3.14)

project(cyber VERSION 6.0.0 LANGUAGES CXX ASM)

set(INSTALL_PATH ${CMAKE_SOURCE_DIR}/out/target/install/aarch64le)
set(TOOLCHAIN_FILE ${CMAKE_SOURCE_DIR}/toolchain/linux_aarch64.toolchain.cmake)

# Options: Things you can set via commandline options to cmake (e.g. -DENABLE_LTO=[ON|OFF])
#option(ENABLE_WARNINGS_SETTINGS "Allow target_set_warnings to add flags and defines.
#                                 Set this to OFF if you want to provide your own warning parameters." ON)

# Include stuff. No change needed.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")
#include(gflags)
#include(glog)
#include(uuid)
#include(protobuf-3.21.5)
#include(fastcdr)
#include(foonathan_memory)
#include(fastdds)

find_package(Glog REQUIRED)

# --------------------------------------------------------------------------------
#                         Locate files (change as needed).
# --------------------------------------------------------------------------------
set(TASK_SRC
    cyber/task/task_manager.cc
)
set(SCHEDULER_SRC
    cyber/scheduler/scheduler.cc
    cyber/scheduler/common/pin_thread.cc
    cyber/scheduler/processor_context.cc
    cyber/scheduler/policy/scheduler_classic.cc
    cyber/scheduler/policy/choreography_context.cc
    cyber/scheduler/policy/classic_context.cc
    cyber/scheduler/policy/scheduler_choreography.cc
    cyber/scheduler/processor.cc
    cyber/scheduler/scheduler_factory.cc
)
set(COMPONENT_SRC
    cyber/component/timer_component.cc
)
set(MESSAGE_SRC
    cyber/message/protobuf_factory.cc
)
set(TIME_SRC
#    cyber/time/rate.cc
#    cyber/time/duration.cc
    cyber/time/time.cc
#    cyber/time/clock.cc
)
set(TIMER_SRC
    cyber/timer/timing_wheel.cc
    cyber/timer/timer.cc
)
set(LOGGER_SRC
    cyber/logger/async_logger.cc
    cyber/logger/log_file_object.cc
    cyber/logger/logger.cc
    cyber/logger/logger_util.cc
)
set(COMMON_SRC
    cyber/common/global_data.cc
    cyber/common/file.cc
)
set(CROUTINE_SRC
    cyber/croutine/croutine.cc
    cyber/croutine/detail/routine_context.cc
    cyber/croutine/detail/swap_aarch64.S
)
set(EVENT_SRC
    cyber/event/perf_event_cache.cc
)
set(BLOCKER_SRC
    cyber/blocker/blocker_manager.cc
)
set(TRANSPORT_SRC
    cyber/transport/transport.cc
    cyber/transport/message/message_info.cc
    cyber/transport/common/endpoint.cc
    cyber/transport/common/identity.cc
    cyber/transport/shm/readable_info.cc
    cyber/transport/shm/state.cc
    cyber/transport/shm/notifier_factory.cc
#    cyber/transport/shm/xsi_segment.cc
    cyber/transport/shm/segment.cc
    cyber/transport/shm/condition_notifier.cc
    cyber/transport/shm/posix_segment.cc
    cyber/transport/shm/multicast_notifier.cc
    cyber/transport/shm/segment_factory.cc
    cyber/transport/shm/shm_conf.cc
    cyber/transport/shm/block.cc
    cyber/transport/dispatcher/dispatcher.cc
    cyber/transport/dispatcher/intra_dispatcher.cc
    cyber/transport/dispatcher/shm_dispatcher.cc
    cyber/transport/dispatcher/rtps_dispatcher.cc
    cyber/transport/qos/qos_profile_conf.cc
    cyber/transport/rtps/sub_listener.cc
    cyber/transport/rtps/participant.cc
    cyber/transport/rtps/underlay_message.cxx
    cyber/transport/rtps/underlay_messagePubSubTypes.cxx
    cyber/transport/rtps/attributes_filler.cc
)
set(SYSMO_SRC
    cyber/sysmo/sysmo.cc
)
set(PARAMETER_SRC
    cyber/parameter/parameter.cc
    cyber/parameter/parameter_client.cc
    cyber/parameter/parameter_server.cc
)
set(NODE_SRC
    cyber/node/node.cc
)
set(CLASS_LOADER_SRC
    cyber/class_loader/class_loader_manager.cc
    cyber/class_loader/utility/class_factory.cc
    cyber/class_loader/utility/class_loader_utility.cc
    cyber/class_loader/shared_library/sample.cc
    cyber/class_loader/shared_library/shared_library.cc
    cyber/class_loader/class_loader.cc
)
set(RECORD_SRC
    cyber/record/record_writer.cc
    cyber/record/file/record_file_writer.cc
    cyber/record/file/record_file_reader.cc
    cyber/record/file/record_file_base.cc
    cyber/record/record_viewer.cc
    cyber/record/record_reader.cc
    cyber/record/header_builder.cc
)
set(IO_SRC
    cyber/io/poller.cc
    cyber/io/poll_handler.cc
    cyber/io/session.cc
)
set(SERVICE_DISCOVERY_SRC
    cyber/service_discovery/container/single_value_warehouse.cc
    cyber/service_discovery/container/multi_value_warehouse.cc
    cyber/service_discovery/container/graph.cc
    cyber/service_discovery/communication/participant_listener.cc
    cyber/service_discovery/communication/subscriber_listener.cc
    cyber/service_discovery/role/role.cc
    cyber/service_discovery/topology_manager.cc
    cyber/service_discovery/specific_manager/manager.cc
    cyber/service_discovery/specific_manager/channel_manager.cc
    cyber/service_discovery/specific_manager/service_manager.cc
    cyber/service_discovery/specific_manager/node_manager.cc
)
set(SOURCES
   ${TASK_SRC}
   ${SCHEDULER_SRC}
#   ${COMPONENT_SRC}
#   ${MESSAGE_SRC}
   ${TIME_SRC}
#   ${TIMER_SRC}
   ${LOGGER_SRC}
   ${COMMON_SRC}
   ${CROUTINE_SRC}
#   ${EVENT_SRC}
#   ${BLOCKER_SRC}
#   ${TRANSPORT_SRC}
   ${SYSMO_SRC}
#   ${PARAMETER_SRC}
#   ${NODE_SRC}
#   ${CLASS_LOADER_SRC}
#   ${RECORD_SRC}
#    ${IO_SRC}
#    ${SERVICE_DISCOVERY_SRC}
    cyber/binary.cc
#    cyber/cyber.cc
#    cyber/state.cc
#    cyber/init.cc
)

file(GLOB protobuf_files ${CMAKE_SOURCE_DIR}/cyber/proto/*.proto)

set(PROTO_META_BASE_DIR ${CMAKE_CURRENT_BINARY_DIR})
list(APPEND PROTO_FLAGS -I${CMAKE_CURRENT_SOURCE_DIR})

set(PROTOC "protoc")

FOREACH(FIL ${protobuf_files})

    GET_FILENAME_COMPONENT(FIL_WE ${FIL} NAME_WE)

    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" FILE_NAME ${FIL})
    string(REGEX REPLACE "(.+)\\${FILE_NAME}.*" "\\1" FILE_PATH ${FIL})

    string(REGEX MATCH "(/cyber/proto/)" OUT_PATH ${FILE_PATH})

    set(PROTO_SRCS "${CMAKE_CURRENT_BINARY_DIR}${OUT_PATH}${FIL_WE}.pb.cc")
    set(PROTO_HDRS "${CMAKE_CURRENT_BINARY_DIR}${OUT_PATH}${FIL_WE}.pb.h")
    message("command = ${PROTOC} ${PROTO_FLAGS} --cpp_out=${PROTO_META_BASE_DIR} ${FIL}")
    EXECUTE_PROCESS(
            COMMAND ${PROTOC} ${PROTO_FLAGS} --cpp_out=${PROTO_META_BASE_DIR} ${FIL}
    )

    file(COPY ${PROTO_SRCS} DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}${OUT_PATH})
    file(COPY ${PROTO_HDRS} DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}${OUT_PATH})
ENDFOREACH()

file(GLOB ALL_PROTO_SRCS ${CMAKE_CURRENT_BINARY_DIR}/cyber/proto/*.pb.cc)
file(GLOB ALL_PROTO_HDRS ${CMAKE_CURRENT_BINARY_DIR}/cyber/proto/*.pb.h)

include_directories(${INSTALL_PATH}/include)

set(TARGET_NAME cyber)  # Default name for the library built from src/*.cpp (change if you wish)

# --------------------------------------------------------------------------------
#                            Build! (Change as needed)
# --------------------------------------------------------------------------------

set_property(SOURCE cyber/croutine/detail/swap_aarch64.S APPEND PROPERTY COMPILE_OPTIONS "-x" "assembler-with-cpp")

set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_CXX_FLAGS "-Wl,-rpath-link=${INSTALL_PATH}/lib")

add_library(${TARGET_NAME} SHARED ${SOURCES} ${ALL_PROTO_SRCS} ${ALL_PROTO_HDRS})

#add_dependencies(${TARGET_NAME} GLOG GFLAGS PROTOBUF fastdds)

target_include_directories(${TARGET_NAME} PUBLIC ${PROJECT_SOURCE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

target_link_directories(${TARGET_NAME} PUBLIC ${INSTALL_PATH}/lib)

#target_link_libraries(${TARGET_NAME} PUBLIC fastrtps fastcdr gflags glog c atomic protobuf uuid)
target_link_libraries(${TARGET_NAME} PUBLIC c atomic protobuf glog)

set(cyber_CMake_BINDIR ${CMAKE_INSTALL_BINDIR})
set(cyber_CMake_LIBDIR ${CMAKE_INSTALL_LIBDIR})

install (TARGETS cyber
        EXPORT cyber-targets
        RUNTIME DESTINATION ${cyber_CMake_BINDIR}
        LIBRARY DESTINATION ${cyber_CMake_LIBDIR}
)

#target_set_warnings(${TARGET_NAME} ENABLE ALL AS_ERROR ALL DISABLE Annoying)
# target_compile_options(${TARGET_NAME} ... )  # For setting manually.

# Add an executable for the file app/main.cpp.
# If you add more executables, copy these lines accordingly.
# add_executable(main app/main.cpp)   # Name of exec. and location of file.
# target_link_libraries(main PRIVATE ${TARGET_NAME})  # Link the executable to library (if it uses it).
# target_set_warnings(main ENABLE ALL AS_ERROR ALL DISABLE Annoying) # Set warnings (if needed).
# target_enable_lto(main optimized)  # enable link-time-optimization if available for non-debug configurations

# Set the properties you require, e.g. what C++ standard to use. Here applied to library and main (change as needed).
#set_target_properties(
#    ${TARGET_NAME} main
#      PROPERTIES
#        CXX_STANDARD 17
#        CXX_STANDARD_REQUIRED YES
#        CXX_EXTENSIONS NO
#)
