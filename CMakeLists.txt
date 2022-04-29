cmake_minimum_required(VERSION 3.8)
project(pigpio_vendor)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(PIGPIO_DIR ${CMAKE_CURRENT_SOURCE_DIR}/pigpio/pigpio)
list (APPEND CMAKE_MODULE_PATH ${PIGPIO_DIR}/cmake)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(Threads REQUIRED)
find_package(RT REQUIRED)

file(GLOB PIGPIO_HEADERS ${PIGPIO_DIR}/*.h)

include_directories(include ${PIGPIO_DIR})

# libpigpio.(so|a)
add_library(pigpio ${PIGPIO_DIR}/pigpio.c ${PIGPIO_DIR}/command.c ${PIGPIO_DIR}/custom.cext)

# libpigpiod_if.(so|a)
add_library(pigpiod_if ${PIGPIO_DIR}/pigpiod_if.c ${PIGPIO_DIR}/command.c)

# libpigpiod_if2.(so|a)
add_library(pigpiod_if2 ${PIGPIO_DIR}/pigpiod_if2.c ${PIGPIO_DIR}/command.c)

# x_pigpio
add_executable(x_pigpio ${PIGPIO_DIR}/x_pigpio.c)
target_link_libraries(x_pigpio pigpio RT::RT Threads::Threads)

# x_pigpiod_if
add_executable(x_pigpiod_if ${PIGPIO_DIR}/x_pigpiod_if.c)
target_link_libraries(x_pigpiod_if pigpiod_if RT::RT Threads::Threads)

# x_pigpiod_if2
add_executable(x_pigpiod_if2 ${PIGPIO_DIR}/x_pigpiod_if2.c)
target_link_libraries(x_pigpiod_if2 pigpiod_if2 RT::RT Threads::Threads)

# pigpiod
add_executable(pigpiod ${PIGPIO_DIR}/pigpiod.c)
target_link_libraries(pigpiod pigpio RT::RT Threads::Threads)

# pigs
add_executable(pigs ${PIGPIO_DIR}/pigs.c ${PIGPIO_DIR}/command.c)
target_link_libraries(pigs Threads::Threads)

# pig2vcd
add_executable(pig2vcd ${PIGPIO_DIR}/pig2vcd.c ${PIGPIO_DIR}/command.c)
target_link_libraries(pig2vcd Threads::Threads)

ament_export_include_directories(${PIGPIO_DIR})
ament_export_dependencies(ament_cmake)
ament_export_targets(${PROJECT_NAME}Targets)
ament_export_libraries(pigpio pigpiod_if pigpiod_if2)

set(TARGETS_LIST
        pigpio
        pigpiod_if
        pigpiod_if2
        x_pigpio
        x_pigpiod_if
        x_pigpiod_if2
        pigpiod
        pigs
        pig2vcd)

install(FILES ${PIGPIO_HEADERS}
        DESTINATION include)

install(TARGETS ${TARGETS_LIST}
        EXPORT ${PROJECT_NAME}Targets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
