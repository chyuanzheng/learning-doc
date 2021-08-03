cmake_minimum_required(VERSION 3.19)
project(TestWx)

set(CMAKE_CXX_STANDARD 14)


add_executable(TestWx  WIN32  MyApp.cpp MyApp.h)

# find_package(wxWidgets COMPONENTS gl core base OPTIONAL_COMPONENTS net)
find_package(wxWidgets REQUIRED COMPONENTS net core base)
if(wxWidgets_FOUND)
    include(${wxWidgets_USE_FILE})
    # and for each of your dependent executable/library targets:
    target_link_libraries(TestWx ${wxWidgets_LIBRARIES})
endif()



