
cmake_minimum_required(VERSION 3.18.0)

project(tinycc LANGUAGES C VERSION 0.9.27)

#CORE_FILES = tcc.c tcctools.c libtcc.c tccpp.c tccgen.c tccdbg.c tccelf.c tccasm.c tccrun.c
#CORE_FILES += tcc.h config.h libtcc.h tcctok.h
#i386_FILES = $(CORE_FILES) i386-gen.c i386-link.c i386-asm.c i386-asm.h i386-tok.h
#i386-win32_FILES = $(i386_FILES) tccpe.c
#x86_64_FILES = $(CORE_FILES) x86_64-gen.c x86_64-link.c i386-asm.c x86_64-asm.h
#x86_64-win32_FILES = $(x86_64_FILES) tccpe.c
#x86_64-osx_FILES = $(x86_64_FILES) tccmacho.c
#arm_FILES = $(CORE_FILES) arm-gen.c arm-link.c arm-asm.c arm-tok.h
#arm-wince_FILES = $(arm_FILES) tccpe.c
#arm-eabihf_FILES = $(arm_FILES)
#arm-fpa_FILES     = $(arm_FILES)
#arm-fpa-ld_FILES  = $(arm_FILES)
#arm-vfp_FILES     = $(arm_FILES)
#arm-eabi_FILES    = $(arm_FILES)
#arm-eabihf_FILES  = $(arm_FILES)
#arm64_FILES = $(CORE_FILES) arm64-gen.c arm64-link.c arm64-asm.c
#arm64-osx_FILES = $(arm64_FILES) tccmacho.c
#c67_FILES = $(CORE_FILES) c67-gen.c c67-link.c tcccoff.c
#riscv64_FILES = $(CORE_FILES) riscv64-gen.c riscv64-link.c riscv64-asm.c

#CFLAGS=-Wall -g -O2 -Wdeclaration-after-statement -fno-strict-aliasing -fheinous-gnu-extensions -Wno-pointer-sign -Wno-sign-compare -Wno-unused-result -Wno-string-plus-int

# libtcc sources
#LIBTCC_SRC = $(filter-out tcc.c tcctools.c,$(filter %.c,$($T_FILES)))
set(CORE_SRCS tcctools.c libtcc.c tccpp.c tccgen.c tccdbg.c tccelf.c tccasm.c tccrun.c)
set(CORE_HDRS tcc.h libtcc.h tcctok.h)

set(I386_SRCS i386-gen.c i386-link.c i386-asm.c)
set(I386_HDRS i386-asm.h i386-tok.h)

set(X86_64_SRCS x86_64-gen.c x86_64-link.c i386-asm.c)
set(X86_64_HDRS x86_64-asm.h)

set(ARM_SRCS arm-gen.c arm-link.c arm-asm.c)
set(ARM64_SRCS arm64-gen.c arm64-link.c arm64-asm.c)

if(WIN32)
  list(APPEND I386_SRCS tccpe.c)
  list(APPEND X86_64_SRCS tccpe.c)
  list(APPEND ARM_SRCS tccpe.c)
  list(APPEND ARM64_SRCS tccpe.c)
elseif(APPLE)
  list(APPEND X86_64_SRCS tccmacho.c)
  list(APPEND ARM64_SRCS tccmacho.c)
endif()

set(LIB_SRCS ${CORE_SRCS})
set(LIB_HDRS ${CORE_HDRS})

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
  if (CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "x86_64")
    list(APPEND LIB_SRCS ${X86_64_SRCS})
    list(APPEND LIB_HDRS ${X86_64_HDRS})
  else()
    list(APPEND LIB_SRCS ${ARM64_SRCS})
    list(APPEND LIB_HDRS ${ARM64_HDRS})
  endif()
else()
  message(STATUS "Check this comparison: CMAKE_HOST_SYSTEM_PROCESSOR: ${CMAKE_HOST_SYSTEM_PROCESSOR}")
  if (CMAKE_HOST_SYSTEM_PROCESSOR STREQUAL "x86")
    list(APPEND LIB_SRCS ${I386_SRCS})
    list(APPEND LIB_HDRS ${I386_HDRS})
  else()
    list(APPEND LIB_SRCS ${ARM_SRCS})
    list(APPEND LIB_HDRS ${ARM_HDRS})
  endif()
endif()

configure_file(tcc_config.in.h tcc_config.h)

add_library(tcc SHARED ${LIB_SRCS} ${LIB_HDRS})
#add_executable(tcc_exe tcc.c)

target_include_directories(tcc PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_compile_definitions(tcc PRIVATE ONE_SOURCE=0)

if(MSVC)
  target_compile_options(tcc PRIVATE /W4 /WX)
else()
  target_compile_options(tcc PRIVATE -Wall -Wextra -Wpedantic -Wdeclaration-after-statement -fno-strict-aliasing -fheinous-gnu-extensions -Wno-pointer-sign -Wno-sign-compare -Wno-unused-result -Wno-string-plus-int)
endif()
