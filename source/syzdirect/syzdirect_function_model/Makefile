CUR_DIR = $(shell pwd)
# LLVM_BUILD := /path/to/your/llvm/toolchain
ANALYZER_DIR := ${CURDIR}/src
ANALYZER_BUILD := ${CURDIR}/build


UNAME := $(shell uname)
ifeq ($(UNAME), Linux)
	NPROC := ${shell nproc}
else
	NPROC := ${shell sysctl -n hw.ncpu}
endif

CC := ${LLVM_BUILD}/bin/clang
CXX := ${LLVM_BUILD}/bin/clang++

build_analyzer_func = \
		(mkdir -p ${2} \
		&& cd ${2} \
		&& LLVM_TOOLS_BINARY_DIR=${LLVM_BUILD}/bin \
			LLVM_LIBRARY_DIRS=${LLVM_BUILD}/lib \
			LLVM_INCLUDE_DIRS=${LLVM_BUILD}/include \
			LLVM_DIR=${LLVM_BUILD} \
			CC=${CC} CXX=${CXX} \
			cmake ${1}	\
				-DCMAKE_BUILD_TYPE=Release \
				-DLLVM_ENABLE_ASSERTIONS=ON \
				-DCMAKE_CXX_FLAGS_RELEASE="-std=c++17 -fno-rtti -fpic -g" \
		&& make -j${NPROC})

all: interface_generator

interface_generator:
	$(call build_analyzer_func, ${ANALYZER_DIR}, ${ANALYZER_BUILD})

clean:
	rm -rf ${ANALYZER_BUILD}
