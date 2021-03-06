#
# Copyright 2020 by Idiap Research Institute, http://www.idiap.ch
#
# See the file COPYING for the licence associated with this software.
#
# Author(s):
#   Phil Garner, August2020
#

# https://developer.nvidia.com/cuda-toolkit
# Here we use 10.0 as it matches Idiap's current drivers
# Ref: https://docs.nvidia.com/deploy/cuda-compatibility/index.html
set(NVIDIA      "https://developer.nvidia.com/compute")
set(CUDA_VERSION "10.0")
set(CUDA_SUBVERSION ".130")
set(CUDA_DRIVER  "410.48")
set(CUDA_FILE    "cuda_${CUDA_VERSION}${CUDA_SUBVERSION}_${CUDA_DRIVER}_linux.run")

# NVidia conda distributions don't contain headers, so we use the original
# If this fails, download by hand and put into buslr/cache
BuSLR_Add(cuda
  URL "${NVIDIA}/cuda/${CUDA_VERSION}/Prod/local_installers/${CUDA_FILE}"
  URL_MD5           9375304c8dc8d42e7f9a9ae37e80b42b
  DOWNLOAD_NAME     ${CUDA_FILE}
  DOWNLOAD_NO_EXTRACT TRUE
  CONFIGURE_COMMAND sh <DOWNLOADED_FILE> --target <SOURCE_DIR> --noexec
  BUILD_COMMAND     true
  INSTALL_COMMAND   ./cuda-installer.pl --silent --override
                    --toolkit --toolkitpath=${CMAKE_INSTALL_PREFIX}/opt/cuda
		    --no-man-page  --no-drm
  )

# CUDNN binaries.  Lift from NVidia's conda dist is easiest
set(NVIDIA_CONDA "https://anaconda.org/nvidia")
set(CUDNN_VERSION "7.6.0")
set(CUDNN_BUILD   "cuda10.0_0")
set(CUDNN_DIR "${NVIDIA_CONDA}/cudnn/${CUDNN_VERSION}/download/linux-64")
BuSLR_Add(cudnn
  URL ${CUDNN_DIR}/cudnn-${CUDNN_VERSION}-${CUDNN_BUILD}.tar.bz2
  URL_MD5           ffd33dfe01265551517291f584d5e062
  CONFIGURE_COMMAND true
  BUILD_COMMAND     true
  INSTALL_COMMAND   cp -dr lib include ${CMAKE_INSTALL_PREFIX}
  )

# NCCL: Use a github archive
set(NCCL_VERSION "2.7.8-1")
set(NCCL_DIR "https://github.com/NVIDIA/nccl/archive")
set(NCCL_TGZ "v${NCCL_VERSION}.tar.gz")
find_package(CUDA)
BuSLR_Add(nccl
  URL               ${NCCL_DIR}/${NCCL_TGZ}
  DOWNLOAD_NAME     "nccl-${NCCL_VERSION}.tar.gz"
  URL_MD5           2ad66167be386ec506426cede8a90df2
  CONFIGURE_COMMAND true
  BUILD_COMMAND     ${MAKE_EXE} src.build CUDA_HOME=${CUDA_TOOLKIT_ROOT_DIR}
  INSTALL_COMMAND   cp -dr build/lib build/include ${CMAKE_INSTALL_PREFIX}
)
ExternalProject_Add_StepDependencies(nccl build cuda)
