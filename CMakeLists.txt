cmake_minimum_required(VERSION 3.16)

project(custom_4coder LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#for creating compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

#set the main custom binding file
if(SOURCE_FILE)
  message(STATUS "Source file is: ${SOURCE_FILE}")
else()
  message(STATUS "Defaulting sourcefile to ./custom/4coder_default_bindings.cpp")
  set(SOURCE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/../custom/4coder_default_bindings.cpp")
endif()

#set dir of custom dir
set(CODE_HOME ${CMAKE_CURRENT_SOURCE_DIR}/../custom)

#set to use debug if no build type passed
if(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Using Debug build type as default")
  set(CMAKE_BUILD_TYPE Debug)
endif()

#Setup the targets
if(UNIX)
  if(APPLE)
    message(STATUS "Detected MacOS environment")
    set(COMPILE_FLAGS_SET -Wno-write-strings -Wno-null-dereference -Wno-comment -Wno-switch -Wno-missing-declarations -Wno-logical-op-parentheses -g)
    set(COMPILE_DEFINITIONS_SET -DOS_MAC=1 -DOS_WINDOWS=0 -DOS_LINUX=0)
  else()
    message(STATUS "Detected Unix environment")
    set(COMPILE_FLAGS_SET -Wno-write-strings -Wno-null-dereference -Wno-comment -Wno-switch -Wno-missing-declarations -Wno-logical-op-parentheses -g) 
    set(COMPILE_DEFINITIONS_SET -DOS_LINUX=1 -DOS_WINDOWS=0 -DOS_MAC=0)
  endif()


  add_custom_target(metadata
    COMMAND ${CMAKE_CXX_COMPILER} -I${CODE_HOME} ${COMPILE_FLAGS_SET} -DMETA_PASS  ${COMPILE_DEFINITIONS_SET} -std=c++11 ${SOURCE_FILE} -E -o 4coder_command_metadata.i
    )

  add_executable(metadata_generator ${CODE_HOME}/4coder_metadata_generator.cpp)
  target_include_directories(metadata_generator PRIVATE ${CODE_HOME})
  target_compile_definitions(metadata_generator PRIVATE 
    ${COMPILE_DEFINITIONS_SET})
  target_compile_options(metadata_generator PRIVATE ${COMPILE_FLAGS_SET})
  add_dependencies(metadata_generator metadata)

  add_custom_target(move_metadata_generator ALL
    COMMAND ${CMAKE_COMMAND} -E copy metadata_generator ${CODE_HOME}/metadata_generator)
  add_dependencies(move_metadata_generator metadata_generator)

  add_custom_target(run_metadata_generator ALL
    COMMAND ${CODE_HOME}/metadata_generator -R "${CODE_HOME}" ${CMAKE_CURRENT_BINARY_DIR}/4coder_command_metadata.i
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  add_dependencies(run_metadata_generator move_metadata_generator)

  add_library(custom_4coder SHARED ${SOURCE_FILE})
  target_include_directories(custom_4coder PRIVATE ${CODE_HOME})
  target_compile_definitions(custom_4coder PRIVATE 
    ${COMPILE_DEFINITIONS_SET})
  target_compile_options(custom_4coder PRIVATE ${COMPILE_FLAGS_SET})
  add_dependencies(custom_4coder run_metadata_generator)

  #If this script was called from the main 4coder build
  #use the following as output dir for the custom_4coder library
  if(DEFINED MAIN_BUILD)
    set(COPY_DIR_4_CUSTOM_4CODER ${CMAKE_BINARY_DIR}/custom_4coder.so)
  else()
    set(COPY_DIR_4_CUSTOM_4CODER ${CMAKE_SOURCE_DIR}/../custom_4coder.so)
  endif()

  add_custom_target(copy_custom_4coder ALL
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_BINARY_DIR}/libcustom_4coder.so ${COPY_DIR_4_CUSTOM_4CODER})
  add_dependencies(copy_custom_4coder custom_4coder)

  add_custom_target(cleanup-metadata ALL
    COMMAND ${CMAKE_COMMAND} -E rm ${CODE_HOME}/metadata_generator)
  add_dependencies(cleanup-metadata copy_custom_4coder)
else()

  message(STATUS "Detected Windows environment")
  set(COMPILE_FLAGS_SET "/W4 /wd4310 /wd4100 /wd4201 /wd4505 /wd4996 /wd4127 /wd4510 /wd4512 /wd4610 /wd4457 /WX /GR- /nologo /FC /D OS_WINDOWS=1 /D OS_LINUX=0 /D OS_MAC=0")

  message(FATAL_ERROR "Microsoft Windows part not yet implemented, sorry.")
endif()

if(NOT DEFINED MAIN_BUILD)
  add_custom_target(copy-compile-commands ALL
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/compile_commands.json ${CMAKE_SOURCE_DIR}/../compile_commands.json)
endif()
