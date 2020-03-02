cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_COMPILER g++)

project(MyProject VERSION 1.0.0)

function(create_default_include_list)
  execute_process(COMMAND ${CMAKE_CXX_COMPILER} -v -x c++ -E /dev/null
    ERROR_VARIABLE OUTPUT
    OUTPUT_VARIABLE _)

  # Get all include directories
  execute_process(COMMAND bash "-c" "echo '${OUTPUT}' | awk '/#include <...>/{flag=1;next}/End of search/{flag=0}flag {$1=$1;print}'"
    OUTPUT_VARIABLE OUTPUT)

  # Remove directory that redundant with glibc
  execute_process(COMMAND bash "-c" "echo '${OUTPUT}' | awk '!/\\/usr\\/include$/'"
    OUTPUT_VARIABLE OUTPUT)

  string(REPLACE "\n" ";" INCLUDE_LIST ${OUTPUT})
  set(INCLUDE_LIST ${INCLUDE_LIST} PARENT_SCOPE)
endfunction()

add_executable(my_target main.cpp)

create_default_include_list()

foreach(dir IN ITEMS ${INCLUDE_LIST})
  target_include_directories(my_target PRIVATE ${dir})
endforeach()
