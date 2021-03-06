cmake_minimum_required(VERSION 2.6)
project(otclient)

set(VERSION "1.0.0")

option(FRAMEWORK_SOUND "Use SOUND " ON)
option(FRAMEWORK_GRAPHICS "Use GRAPHICS " ON)
option(FRAMEWORK_XML "Use XML " ON)
option(FRAMEWORK_NET "Use NET " ON)
option(FRAMEWORK_SQL "Use SQL" OFF)

include(src/framework/CMakeLists.txt)
include(src/client/CMakeLists.txt)

add_subdirectory(canary-lib)

# functions map for reading backtraces
if(NOT APPLE)
    set(CMAKE_CXX_LINK_FLAGS "${CMAKE_CXX_LINK_FLAGS} -fPIC -no-pie -Wl,-Map=${PROJECT_NAME}.map")
endif()

option(USE_PCH "Use precompiled header (speed up compile)" OFF)

set(executable_SOURCES
    src/main.cpp
)

# add executable icon for win32 platforms
if(WIN32)
    set(executable_SOURCES ${executable_SOURCES})
endif()

add_definitions(-D"VERSION=\\"${VERSION}\\"")

# add client executable
add_executable(${PROJECT_NAME} ${framework_SOURCES} ${client_SOURCES} ${executable_SOURCES})
target_link_libraries(${PROJECT_NAME} ${framework_LIBRARIES} canary_lib::common_lib)

if(USE_PCH)
    include(cotire)
    cotire(${PROJECT_NAME})
    message(STATUS "Use precompiled header: ON")
else()
    message(STATUS "Use precompiled header: OFF")
endif()

# installation
set(DATA_INSTALL_DIR share/${PROJECT_NAME})
install(TARGETS ${PROJECT_NAME}
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib
		ARCHIVE DESTINATION lib)
install(FILES README.md BUGS LICENSE AUTHORS init.lua ${PROJECT_NAME}rc.lua DESTINATION ${DATA_INSTALL_DIR})
install(DIRECTORY data modules DESTINATION ${DATA_INSTALL_DIR} PATTERN ".git" EXCLUDE)

# add "make run"
add_custom_target(run COMMAND ${PROJECT_NAME} DEPENDS ${PROJECT_NAME} WORKING_DIRECTORY ${CMAKE_PROJECT_DIR})
