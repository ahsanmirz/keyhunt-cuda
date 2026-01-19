#---------------------------------------------------------------------
# Makefile for KeyHunt
#
# Author : Jean-Luc PONS

SRC = Base58.cpp IntGroup.cpp Main.cpp Bloom.cpp Random.cpp \
      Timer.cpp Int.cpp IntMod.cpp Point.cpp SECP256K1.cpp \
      KeyHunt.cpp GPU/GPUGenerate.cpp hash/ripemd160.cpp \
      hash/sha256.cpp hash/sha512.cpp hash/ripemd160_sse.cpp \
      hash/sha256_sse.cpp hash/keccak160.cpp GmpUtil.cpp CmdParse.cpp

OBJDIR = obj

ifdef gpu

OBJET = $(addprefix $(OBJDIR)/, \
        Base58.o IntGroup.o Main.o Bloom.o Random.o Timer.o Int.o \
        IntMod.o Point.o SECP256K1.o KeyHunt.o GPU/GPUGenerate.o \
        hash/ripemd160.o hash/sha256.o hash/sha512.o \
        hash/ripemd160_sse.o hash/sha256_sse.o hash/keccak160.o \
        GPU/GPUEngine.o GmpUtil.o CmdParse.o)

else

OBJET = $(addprefix $(OBJDIR)/, \
        Base58.o IntGroup.o Main.o Bloom.o Random.o Timer.o Int.o \
        IntMod.o Point.o SECP256K1.o KeyHunt.o GPU/GPUGenerate.o \
        hash/ripemd160.o hash/sha256.o hash/sha512.o \
        hash/ripemd160_sse.o hash/sha256_sse.o hash/keccak160.o \
        GmpUtil.o CmdParse.o)

endif

CXX        = g++
CXXCUDA    = /usr/bin/g++

# Allow overriding CUDA/NVCC from environment/command line:
#   make gpu=1 NVCC=/opt/conda/bin/nvcc CCAP=86 all
#   make gpu=1 CUDA=/usr/local/cuda-12.8 CCAP=86 all
#   make gpu=1 NVCC=/usr/local/cuda-12.8/bin/nvcc CUDA=/usr/local/cuda-12.8 CCAP=86 all
NVCC      ?= nvcc
NVCC_PATH := $(if $(findstring /,$(NVCC)),$(NVCC),$(shell command -v $(NVCC) 2>/dev/null))
CUDA_FROM_NVCC := $(if $(NVCC_PATH),$(realpath $(dir $(NVCC_PATH))/..),)
CUDA      ?= $(if $(CUDA_FROM_NVCC),$(CUDA_FROM_NVCC),/usr/local/cuda)
CUDA_INC  ?= $(CUDA)/include
CUDA_LIB  ?= $(if $(wildcard $(CUDA)/lib64),$(CUDA)/lib64,$(CUDA)/lib)
# nvcc requires joint notation w/o dot, i.e. "5.2" -> "52"
ccap       = $(shell echo $(CCAP) | tr -d '.')

ifdef gpu
ifeq ($(strip $(wildcard $(NVCC_PATH))),)
$(error nvcc not found (looked for '$(NVCC)'). Tip: 'command -v nvcc' and pass NVCC=/path/to/nvcc. Example: make gpu=1 NVCC=/opt/conda/bin/nvcc CCAP=86 all)
endif
endif

ifdef gpu
ifdef debug
CXXFLAGS   = -DWITHGPU -m64  -mssse3 -Wno-write-strings -g -I. -I$(CUDA_INC)
else
CXXFLAGS   =  -DWITHGPU -m64 -mssse3 -Wno-write-strings -O2 -I. -I$(CUDA_INC)
endif
LFLAGS     = -lgmp -lpthread -L$(CUDA_LIB) -lcudart
else
ifdef debug
CXXFLAGS   = -m64 -mssse3 -Wno-write-strings -g -I. -I$(CUDA_INC)
else
CXXFLAGS   =  -m64 -mssse3 -Wno-write-strings -O2 -I. -I$(CUDA_INC)
endif
LFLAGS     = -lgmp -lpthread
endif


#--------------------------------------------------------------------

ifdef gpu
ifdef debug
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -G -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -g -I$(CUDA_INC) -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
else
$(OBJDIR)/GPU/GPUEngine.o: GPU/GPUEngine.cu
	$(NVCC) -maxrregcount=0 --ptxas-options=-v --compile --compiler-options -fPIC -ccbin $(CXXCUDA) -m64 -O2 -I$(CUDA_INC) -gencode=arch=compute_$(ccap),code=sm_$(ccap) -o $(OBJDIR)/GPU/GPUEngine.o -c GPU/GPUEngine.cu
endif
endif

$(OBJDIR)/%.o : %.cpp
	$(CXX) $(CXXFLAGS) -o $@ -c $<

all: KeyHunt

KeyHunt: $(OBJET)
	@echo Making KeyHunt...
	$(CXX) $(OBJET) $(LFLAGS) -o KeyHunt

$(OBJET): | $(OBJDIR) $(OBJDIR)/GPU $(OBJDIR)/hash

$(OBJDIR):
	mkdir -p $(OBJDIR)

$(OBJDIR)/GPU: $(OBJDIR)
	cd $(OBJDIR) &&	mkdir -p GPU

$(OBJDIR)/hash: $(OBJDIR)
	cd $(OBJDIR) &&	mkdir -p hash

clean:
	@echo Cleaning...
	@rm -f obj/*.o
	@rm -f obj/GPU/*.o
	@rm -f obj/hash/*.o


