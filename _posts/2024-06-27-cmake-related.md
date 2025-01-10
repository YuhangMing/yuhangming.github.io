---
layout: post
title:  CMake Bugs
date:   2024-06-27 11:10:16
description: debugging notes.
tags: linux, programming
categories: mix-posts
---

# Table of Content
- [Table of Content](#table-of-content)
- [install without sudo](#install-without-sudo)
- [undefined symbol](#undefined-symbol)
- [could not find OpenCV](#could-not-find-opencv)
- [could not find CUDA](#could-not-find-cuda)

# install without sudo
Error Message: 
```
Install the project...
-- Install configuration: "Release"
CMake Error at cmake_install.cmake:46 (file):
  file cannot create directory: /usr/local/lib/cmake.  Maybe need
  administrative privileges.

make: *** [Makefile:100: install] Error 1
```

Cause: the default install path is `/usr/local/lib/cmake`, which requires sudo permission.

Solution: change the install path to a user-writable directory.
```
mkdir build && cd build
cmake -D CMAKE_INSTALL_PREFIX='/home/username/local' ..
make -j8 && make install
```

!!! Don't forget to change environment variables in the `.bashrc` file.
```
export PATH=$PATH:/home/username/local/bin 
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/username/local/lib 
export CMAKE_PREFIX_PATH=/home/username/local
```


# undefined symbol
Error Message: 
```
Traceback (most recent call last):
  File "test_vis.py", line 3 in <module>
    from cpp.build import pcl
ImportError: /xxx/pcl.cpython-38-x86_64-linux-gnu.so: undefined symbol: _ZN12PointMatcherIfE12ICPChainBase10setDefaultEv
```

Cause: the built module is not linked to the necessary libraries.

Solution: add
```
pybind11_add_module(pcl, pcl.cpp)
target_link_libraries(pcl PRIVATE ${MY_LIBRARIES})
```

[ref](https://github.com/pybind/pybind11/issues/1403)


[Back to Top](#table-of-content)


# could not find OpenCV
Error Message: 
```
CMake Error at CMakeLists.txt:53 (find_package):
  By not providing "FindOpenCV.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "OpenCV", but
  CMake did not find one.

  Could not find a package configuration file provided by "OpenCV" with any
  of the following names:

    OpenCVConfig.cmake
    opencv-config.cmake

  Add the installation prefix of "OpenCV" to CMAKE_PREFIX_PATH or set
  "OpenCV_DIR" to a directory containing one of the above files.  If "OpenCV"
  provides a separate development package or SDK, be sure it has been
  installed.
```

Cause: CMake need to find the OpenCV installed in system, not in the anaconda virtual environment.

Solution: Install OpenCV in the system with sudo user.
`sudo apt-get install libopencv-dev`.

[Back to Top](#table-of-content)


# could not find CUDA
Error Message: 
```
-- Configuring incomplete, errors occurred!
      Traceback (most recent call last):
        File "/tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/skbuild/setuptools_wrap.py", line 666, in setup
          env = cmkr.configure(
        File "/tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/skbuild/cmaker.py", line 357, in configure
          raise SKBuildError(msg)
      
      An error occurred while configuring with CMake.
        Command:
          /tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/cmake/data/bin/cmake /tmp/pip-install-07gczwgt/llama-cpp-python_dac2049bbf404ad88046ca7ba38e3fdb -G Ninja -DCMAKE_MAKE_PROGRAM:FILEPATH=/tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/ninja/data/bin/ninja --no-warn-unused-cli -DCMAKE_INSTALL_PREFIX:PATH=/tmp/pip-install-07gczwgt/llama-cpp-python_dac2049bbf404ad88046ca7ba38e3fdb/_skbuild/linux-x86_64-3.10/cmake-install -DPYTHON_VERSION_STRING:STRING=3.10.12 -DSKBUILD:INTERNAL=TRUE -DCMAKE_MODULE_PATH:PATH=/tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/skbuild/resources/cmake -DPYTHON_EXECUTABLE:PATH=/usr/bin/python3 -DPYTHON_INCLUDE_DIR:PATH=/usr/include/python3.10 -DPYTHON_LIBRARY:PATH=/usr/lib/x86_64-linux-gnu/libpython3.10.so -DPython_EXECUTABLE:PATH=/usr/bin/python3 -DPython_ROOT_DIR:PATH=/usr -DPython_FIND_REGISTRY:STRING=NEVER -DPython_INCLUDE_DIR:PATH=/usr/include/python3.10 -DPython_NumPy_INCLUDE_DIRS:PATH=/usr/lib/python3/dist-packages/numpy/core/include -DPython3_EXECUTABLE:PATH=/usr/bin/python3 -DPython3_ROOT_DIR:PATH=/usr -DPython3_FIND_REGISTRY:STRING=NEVER -DPython3_INCLUDE_DIR:PATH=/usr/include/python3.10 -DPython3_NumPy_INCLUDE_DIRS:PATH=/usr/lib/python3/dist-packages/numpy/core/include -DCMAKE_MAKE_PROGRAM:FILEPATH=/tmp/pip-build-env-vyy1n26b/overlay/local/lib/python3.10/dist-packages/ninja/data/bin/ninja -DLLAMA_CUBLAS=on -DCMAKE_BUILD_TYPE:STRING=Release -DLLAMA_CUBLAS=on
        Source directory:
          /tmp/pip-install-07gczwgt/llama-cpp-python_dac2049bbf404ad88046ca7ba38e3fdb
        Working directory:
          /tmp/pip-install-07gczwgt/llama-cpp-python_dac2049bbf404ad88046ca7ba38e3fdb/_skbuild/linux-x86_64-3.10/cmake-build
      Please see CMake's output for more information.
      
      [end of output]
  
  note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for llama-cpp-python
Failed to build llama-cpp-python
ERROR: Could not build wheels for llama-cpp-python, which is required to install pyproject.toml-based projects
```

Cause: CMake need to find the CUDA installed in system, not in the anaconda virtual environment.

Solution: Force CMake to find the CUDA installed in the system.
`CUDACXX=/usr/local/cuda-12.1/bin/nvcc CMAKE_ARGS="-DLLAMA_CUBLAS=on -DCMAKE_CUDA_ARCHITECTURES=native" FORCE_CMAKE=1 pip install ./DPViewer`

[Back to Top](#table-of-content)
