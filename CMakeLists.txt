cmake_minimum_required(VERSION 3.10)

project(MyProject C CXX)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11)

# ----------------------
# C firmware target
# ----------------------
if(EXISTS "${CMAKE_SOURCE_DIR}/src/main.c")
    find_program(CMAKE_OBJCOPY NAMES arm-none-eabi-objcopy objcopy)

    add_executable(myfirmware.elf
        src/main.c
    )

    # Generate .bin from .elf
    add_custom_command(TARGET myfirmware.elf POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O binary myfirmware.elf myfirmware.bin
        COMMENT "Generating .bin from .elf"
    )
else()
    message(STATUS "Skipping C firmware build: src/main.c not found")
endif()

# ----------------------
# C++ project target
# ----------------------
if(EXISTS "${CMAKE_SOURCE_DIR}/main.cpp")
    add_executable(MyCMakeProject main.cpp)
else()
    message(WARNING "C++ source main.cpp not found, skipping C++ build")
endif()
