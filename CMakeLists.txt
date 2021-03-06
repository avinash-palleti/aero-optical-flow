cmake_minimum_required(VERSION 3.1)
option( RTPS "RTPS Messaging" OFF)
project(aero-optical-flow)

find_package(OpenCV REQUIRED)
find_package(Threads REQUIRED)

set(MAVLINK_DIALECT common)

add_subdirectory(modules/OpticalFlow)

if(RTPS)
file(GLOB RTPS_SRC
    "modules/rtps/*.h"
    "modules/rtps/*.cpp"
)
add_definitions(-DRTPS)
add_executable(aero-optical-flow src/mainloop.cpp src/camera.cpp src/main.cpp ${RTPS_SRC})
target_include_directories(aero-optical-flow PUBLIC ${CMAKE_SOURCE_DIR}/modules/rtps)
target_link_libraries(aero-optical-flow fastcdr)
target_link_libraries(aero-optical-flow fastrtps)
else()
add_executable(aero-optical-flow src/mainloop.cpp src/camera.cpp src/mavlink_tcp.cpp src/main.cpp)
endif()

set_property(TARGET aero-optical-flow PROPERTY CXX_STANDARD 11)
set_property(TARGET aero-optical-flow PROPERTY COMPILE_FLAGS "-Wall -Wextra -Wshadow -Wundef -Wformat=2 -Wlogical-op -Wsign-compare -Wformat-security -Wformat-nonliteral -Wpointer-arith -Winit-self -Wmissing-noreturn -Wstrict-aliasing=2 -Wno-overlength-strings -Wno-missing-field-initializers -Wuninitialized")
target_include_directories(aero-optical-flow PUBLIC ${CMAKE_SOURCE_DIR}/modules/mavlink_c/${MAVLINK_DIALECT})
target_link_libraries(aero-optical-flow ${OpenCV_LIBS})
target_link_libraries(aero-optical-flow OpticalFlow)
target_link_libraries(aero-optical-flow pthread)
install(TARGETS aero-optical-flow DESTINATION bin)

configure_file(aero-optical-flow.service.in aero-optical-flow.service)
