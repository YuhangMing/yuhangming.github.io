---
layout: post
title:  Environment Setup on Asus TUF A15 Laptop
date:   2023-02-01 16:40:16
description: Detailed instruction on programming environment setup.
tags: env-setup
categories: mix-posts
---

# Table of Content
- [Table of Content](#table-of-content)
- [System Info](#system-info)
- [Ubuntu](#ubuntu)
- [NVIDIA](#nvidia)
- [OpenCV](#opencv)
    - [Problems Encountered CV](#problems-encountered-cv)
- [Anaconda](#anaconda)
- [NOCS Network](#nocs-network)
    - [Problems Encountered NOCS](#problems-encountered-nocs)
- [Tensorflow](#tensorflow)
- [Libfusion](#libfusion)
    - [Problems Encountered libfusion](#problems-encountered-libfusion)
- [Azure Kinect Depth Camera](#azure-kinect-depth-camera)
- [Log](#log)
    - [OpenCV CMake Output](#opencv-cmake-output)
    - [Tensorflow Configuration Example](#tensorflow-configuration-example)
    - [Tensorflow Bazel Build Output](#tensorflow-bazel-build-output)


# System Info
2021 Asus TUF Gaming A15, Windows 10 and Ubuntu 20.04.2 dual systems

CPU: AMD Ryzen7 5800H

GPU: GeForce RTX 3070 Laptop

Memory: 16GB

[Back to Top](#table-of-content)


# Ubuntu
* Ubuntu+Windows双系统安装时怎么分配空间
  1. 下载Ubuntu 20.04.2镜像 [here](https://ubuntu.com/download/desktop);
  2. 在Windows下使用[rufus](https://rufus.ie/)/在Mac下使用[Terminal](https://wangchujiang.com/handbook/CentOS/Mac%E4%B8%8B%E5%88%B6%E4%BD%9CCentOS7%E5%90%AF%E5%8A%A8%E7%9B%98.html)制作ubuntu安装盘; 
  3. 在"磁盘管理"中空出相应磁盘空间c
  4. 重启，按F2进入BIOS，选择Boot from USB;
  5. 按提示菜单安装Ubuntu（国内建议选择Minimal Installation并起【不要】选择Download updates during Installation）;
  6. 在选择安装方式时选择"Something Else";
  7. 分8GB给“swap”，剩下的所有空间都分给“/”即可
* 修改阿里源
  Detailed Instructions can be found [here](https://blog.csdn.net/wangyijieonline/article/details/105360138). In short:
  1. 备份原来的源
    ```
    sudo cp /etc/apt/sources.list /etc/apt/sources_init.list
    ```

  2. 修改源with following text<br>
    ```
    deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
    deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
    deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
    ```

  3. 更新
    ```
    sudo apt-get update
    sudo apt-get upgrade
    ```

[Back to Top](#table-of-content)


# NVIDIA
* Driver (460.39): 
  Three different ways to install the NVIDIA-driver in ubuntu, refer to [Install NVIDIA drive on Ubuntu 20.04](https://linuxconfig.org/how-to-install-the-nvidia-drivers-on-ubuntu-20-04-focal-fossa-linux).
  *GUI or Command-line intallation is preferred.*

  To solve the compatibility problem between AMD Ryzen7 5xxx series CPU and RTX 3xxx series GPU, use the [procedure](https://forums.developer.nvidia.com/t/ubuntu-mate-20-04-with-rtx-3070-on-ryzen-5900-black-screen-after-boot/167681/30).

  To check if the installation succeeded, run:
  ```
  nvidia-smi
  ```

  To uninstall NVIDIA Driver (BE CAUTION), run:
  ```
  sudo /usr/bin/nvidia-uninstall
  ```
  
* [CUDA Toolkit](https://developer.nvidia.com/cuda-toolkit-archive) (11.1.1): 
  Note that Cuda toolkit need to be compatible with TensorFlow. Check the [table](https://www.tensorflow.org/install/source#gpu)

  Since the NVIDIA driver is already installed, recommend to install Cuda Toolkit using runfile, and make sure to un-select install driver during the installation.
  
  Because the latest tensorflow 2.4.0 supports CUDA 11.0, cudnn 8.0, and python 3.6-3.8, CUDA 11.0 is installed using runfile. 
  
  UPDATE, because the only Cuda 11.1 and later versions support compute capability 8.6, changed to CUDA 11.1.1 here.
  
  Installation Instructions:
  ```
  wget http://developer.download.nvidia.com/compute/cuda/11.1.1/local_installers/cuda_11.1.1_455.32.00_linux.run
  sudo sh cuda_11.1.1_455.32.00_linux.run
  ```
  
  The output is:
  ```console
  ===========
  = Summary =
  ===========

  Driver:   Not Selected
  Toolkit:  Installed in /usr/local/cuda-11.1/
  Samples:  Installed in /home/yohann/, but missing recommended libraries

  Please make sure that
   -   PATH includes /usr/local/cuda-11.1/bin
   -   LD_LIBRARY_PATH includes /usr/local/cuda-11.1/lib64, or, add /usr/local/cuda-11.1/lib64 to /etc/ld.so.conf and run ldconfig as root

  To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.1/bin
  ***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 455.00 is required for CUDA 11.1 functionality to work.
  To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
      sudo <CudaInstaller>.run --silent --driver

  Logfile is /var/log/cuda-installer.log
  ```
  
  Update add following command to ~/.bashrc file:
  ```
  export PATH=/usr/local/cuda-11.1/bin:$PATH
  export LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64:$LD_LIBRARY_PATH
  ```
  
  To check if the installation succeeded, run:
  ```
  nvcc -V
  nvcc --version
  ```
  
  To delete current CUDA:
  ```
  sudo /usr/local/cuda-11.1/bin/cuda-unintstaller
  ```

* [cuDNN](https://developer.nvidia.com/rdp/cudnn-archive) (8.0.5): 

  Installed using debian: 
  ```
  sudo dpkg -i libcudnn8_8.0.5.39-1+cuda11.1_amd64.deb
  sudo dpkg -i libcudnn8-dev_8.0.5.39-1+cuda11.1_amd64.deb
  sudo dpkg -i libcudnn8-samples_8.0.5.39-1+cuda11.1_amd64.deb
  ```
  
  To check if the installation succeeded, run:
  ```
  cat /usr/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
  ```
    
  No need to uninstall, will be automatically written over (see the output below).
  ```
  Preparing to unpack libcudnn8_8.0.5.39-1+cuda11.1_amd64.deb ...
  Unpacking libcudnn8 (8.0.5.39-1+cuda11.1) over (8.0.5.39-1+cuda11.0) ...
  Setting up libcudnn8 (8.0.5.39-1+cuda11.1) ...
  Processing triggers for libc-bin (2.31-0ubuntu9.2) ...
  ```

[Back to Top](#table-of-content)


# OpenCV
  Links to download OpenCV source codes (3.4.13 chosen here):
  [OpenCV](https://opencv.org/releases/),
  [OpenCV-contrib](https://github.com/opencv/opencv_contrib/releases).
  
  Prerequisite

    [compiler] sudo apt-get install build-essential
    [required] sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
    [optional] sudo apt-get install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libjasper-dev libdc1394-22-dev

  Install OpenCV
  ```
  cmake -D CMAKE_BUILD_TYPE=RELEASE -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-3.4.13/modules -D WITH_CUDA=ON -D WITH_CUDNN=OFF -D BUILD_opencv_cudacodec=OFF -D OPENCV_ENABLE_NONFREE=ON -DBUILD_JAVA=OFF -DBUILD_opencv_java_bindings_generator=OFF -D BUILD_PYTHON=OFF -D BUILD_opencv_python_bindings_generator=OFF -D WITH_QT=ON -D ENABLE_CXX11=ON -D BUILD_TIFF=ON ..
  make -j12
  sudo make install
  ```
  
  [CMake Output](#opencv-cmake-output)

### Problems Encountered CV
  > E: Unable to locate package libjasper-dev
  ```
  sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
  sudo apt update
  sudo apt install libjasper-dev
  ```
  
  > nvcc fatal: Unsupported gpu architecture 'compute_86'
  
  Why: CUDA outdated, 'compute_86' is supported from CUDA 11.1.1
  
  Sol: 
  
  i) update CUDA from 11.0 to 11.1.1;
  
  ii) set -D CUDA_ARCH_BIN=8.0 when building opencv.
   
  > undefined reference to TIFF
  ```console
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFReadRGBAStrip@LIBTIFF_4.0'\s\s
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFReadDirectory@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFWriteEncodedStrip@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFIsTiled@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFWriteScanline@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFGetField@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFScanlineSize@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFWriteDirectory@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFReadEncodedTile@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFReadRGBATile@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFClose@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFClientOpen@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFRGBAImageOK@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFOpen@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFReadEncodedStrip@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFSetField@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFSetWarningHandler@LIBTIFF_4.0'
  /usr/bin/ld: ../../lib/libopencv_imgcodecs.so.3.4.13: undefined reference to `TIFFSetErrorHandler@LIBTIFF_4.0'
  ```
   
  Sol: 在cmake 编译OpenCV时： cmake -D BUILD_TIFF=ON
  
  > undifined reference to cairo
  ```
  /usr/bin/ld: /lib/x86_64-linux-gnu/librsvg-2.so.2: undefined reference to `cairo_tag_end'
  /usr/bin/ld: /lib/x86_64-linux-gnu/librsvg-2.so.2: undefined reference to `cairo_tag_begin'
  /usr/bin/ld: /lib/x86_64-linux-gnu/librsvg-2.so.2: undefined reference to `cairo_font_options_get_variations'
  ```

  Why: Anaconda environments caused the mismatch between librsvg and libcairo.
  
  Sol: 
  
  i) set LD_PRELOAD to the path of match librsvg and libcairo;
  
  ii) remove Anaconda and reinstall after opencv is compiled. (This solution is chosen here)
  
  > python error: no module named 'cv2'
  ```
  pip install opencv-python
  ```

[Back to Top](#table-of-content)


# Anaconda
  Install Anaconda, follow the official instruction [here](https://docs.anaconda.com/anaconda/install/linux/)
  
  *recommend to install OpenCV before Anaconda.*
  
  To uninstall:
  ```
  rm -rf ~/anaconda3
  ```
  and make sure to delete the conda-related lines in ~/.bashrc file.

[Back to Top](#table-of-content)


# NOCS Network
  Updated code can be found [here](https://github.com/YuhangMing/NOCS_CVPR2019).
  
  1. Create anaconda environment and enter virtual environment (open3d in pip supports upto python 3.7)
  ```
  conda create -n NOCS python=3.7
  conda activate NOCS
  ```
  
  2. Install tensorflow
  Detailed instructions see [here](#tensorflow)
  
  
  3. Install keras (2.4.3 chosen here)
  ```
  pip install keras
  ```
  Note: There is not any keras-gpu package; Keras is a wrapper around some backends, including Tensorflow, and these backends may come in different versions, such as tensorflow and tensorflow-gpu. But this does not hold for Keras itself
  
  UPDATE: there is now a keras-gpu package on Anaconda Cloud.
  ```
  conda install -c anaconda keras-gpu
  ```
   
  This will install Keras along with both tensorflow and tensorflow-gpu libraries as the backend. (There is also no need to install separately the CUDA runtime and cudnn libraries as they are also included in the package - tested on Windows 10 and working).
   
  4. Some additional packages (scikit-image, open3d, pycocotools):
  ```
  pip install opencv-python
  pip install scikit-image
  pip install open3d-python
  conda install -c conda-forge pycocotools
  ```
     
### Problems Encountered NOCS

  > CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://repo.anaconda.com/pkgs/main/linux-64/current_repodata.json>
  
  Sol: Change to Tsinghua Mirror using the instruction [here](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/).
  
  *To change back to original conda source, use*
  
    conda config --remove-key channels

  > ReadTimeoutError: pip._vendor.urllib3.exceptions.ReadTimeoutError: HTTPSConnectionPool(host='files.pythonhosted.org', port=443): Read timed out.

  Sol: Change to Tsinghua Mirror temporarily (in single installation).
  ```
  pip install -i mirror-link package-name
  ```
  
  where mirror-link is:
  ```
  清华源：https://pypi.tuna.tsinghua.edu.cn/simple
  豆瓣源：http://pypi.douban.com/simple/
  阿里源：https://mirrors.aliyun.com/pypi/simple/
  腾讯源：http://mirrors.cloud.tencent.com/pypi/simple
  ```
  
  Eg.
  ```
  pip install -i https://pypi.tuna.tsinghua.edu.cn/simple opencv-python
  ```
  
  > Server terminated abruptly (error code: 14, error message: 'Socket closed', log file: '/home/yohann/.cache/bazel/_bazel_yohann/65412b27117725ef7ef78067e0377b6c/server/jvm.out')
  
  Why: Running out of memory in bazel build.
  
  Sol: Add "*-j x*" flag where x is 4/8/16
  ```
  bazel build --config=cuda //tensorflow/tools/pip_package:build_pip_package -j 8
  ```
  
  > Failed to get convolution algorithm.
  ```
  (0) Unknown: Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.
           [[{{node conv1/Conv2D}}]]
  (1) Unknown: Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.
           [[{{node conv1/Conv2D}}]]
           [[mrcnn_bbox/Reshape/_2851]]
  ```
  
  Why: Possilbly because GPU runs out of memory.
  
  Sol:
  ```py
  gpus = tf.config.list_physical_devices('GPU')
  if gpus:
    try:
    # Currently, memory growth needs to be the same across GPUs
    for gpu in gpus:
      tf.config.experimental.set_memory_growth(gpu, True)
      logical_gpus = tf.config.experimental.list_logical_devices('GPU')
      print(len(gpus), "Physical GPUs,", len(logical_gpus), "Logical GPUs")
    except RuntimeError as e:
      # Memory growth must be set before GPUs have been initialized
      print(e)
  ```
   
  > W tensorflow/stream_executor/gpu/asm_compiler.cc:235] Your CUDA software stack is old. We fallback to the NVIDIA driver for some compilation. Update your CUDA version to get the best performance. The ptxas error was: ptxas fatal   : Value 'sm_86' is not defined for option 'gpu-name'
   
   Why: Same as OpenCV, 'sm_86' is supported after Cuda 11.1.1
   
   Sol:
   
   返工！！！
   Update Cuda to 11.1.1 or later. Re-do all the OpenCV and Anaconda setup.

[Back to Top](#table-of-content)


# Tensorflow
Two options here to install tensorflow:

option(1) Installed through pip for released versions. Note, up to today, 2.4.0 is the latest version available in pip.
  ```
  pip install --upgrade pip
  pip install --upgrade tensorflow-gpu==2.4
  ```
option(2) Build from source to support CUDA 11.1 (Given released versions don't support CUDA 11.1 currently) [link](https://www.tensorflow.org/install/source)
  - Install dependencies;
  ```
  pip install -U --user pip numpy wheel
  pip install -U --user keras_preprocessing --no-deps
  ```
  - Install bazel follow [here](https://docs.bazel.build/versions/master/install-ubuntu.html);
    Make sure to install a supported Bazel version: any version between `_TF_MIN_BAZEL_VERSION` and `_TF_MAX_BAZEL_VERSION` as specified in *tensorflow/configure.py*.
  - Clone tensorflow source code, using master branch here;
  ```
  git clone https://github.com/tensorflow/tensorflow.git
  cd tensorflow
  ```
  - Set compile configurations ([configuration example](#tensorflow-configuration-example));
  ```
  ./configure
  ```
  - Build the pip package ([bazel build output](#tensorflow-bazel-build-output));
  *Took about 10-20 mins to download necessary packages and 1 hour to build*.
  ```
  bazel build --config=cuda //tensorflow/tools/pip_package:build_pip_package -j 8
  ```
  - Build the package
  ```
  ./bazel-bin/tensorflow/tools/pip_package/build_pip_package --nightly_flag /tmp/tensorflow_pkg
  ```
  Note "*--nightly_flag*" here is optional.
  - Install the pip package
  ```
  pip install /tmp/tensorflow_pkg/tf_nightly-2.5.0-cp37-cp37m-linux_x86_64.whl

  ```
  
[Back to Top](#table-of-content)


# Libfusion
  **Object Guided Relocalisation**
  1. Install the dependencies:
  - [CUDA](https://developer.nvidia.com/cuda-zone)
  - [OpenCV 3.4.*](https://opencv.org/)
  - [Eigen](http://eigen.tuxfamily.org/)
  - [Pangolin](https://github.com/stevenlovegrove/Pangolin)
  - [Sophus](https://github.com/strasdat/Sophus)
  - [g2o](https://github.com/RainerKuemmerle/g2o)
  2. Cameras
  - OpenNI2 currently removed, add [OpenNI2](https://structure.io/openni) dependency back later.
  - Setup Azure camera follow instructions [here](#azuer_kinect_depth_camera) 
  3. Build the executables and run as:
  ```
  cmake ..
  make -j12
  ./bin/vil_reconst BOR 0 true
  ```

### Problems Encountered libfusion
  > TF crashes on startup
  ```
  GPU_TEST = re.search(r"(test_gpu|test_xla_gpu)$", sys.argv[0])
  IndexError: list index out of range
  ```

  Why: which is caused by calling from C++, but reason unknown.
  
  Sol:
  Use the workaround mention [here](https://github.com/tensorflow/tensorflow/issues/45994).
  ```python
  import sys
  if not sys.argv:
    sys.argv.append("(C++)")

  import tensorflow as tf
  ```
  
  > SegFault in Release Mode, but no bug in Debug Mod
  
  Why: 一般这种问题是因为（1）某个函数没有返回值；（2）某个变量没有初始化。
  
  > Taking way too long in object pose estimation
  
  Why: Because the Debug mode, the timing is normal in Release mode.
    
[Back to Top](#table-of-content)


# Azure Kinect Depth Camera

1. Prerequisite:

  OpenSSL, use `openssl version` to check if it's already installed

  OpenGL, use `glxinfo | grep "OpenGL version"` to check if it's already installed

  [Depth Engine](./libs)
  ```
  sudo dpkg -i libk4a1.4_1.4.1_amd64.deb
  sudo dpkg -i libk4a1.4-dev_1.4.1_amd64.deb
  ```

  In my installation, the following package is also missing.
  ```
  sudo apt install libudev-dev
  ```

2. Install using CMake

  Download the source code [here](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/61951daac782234f4f28322c0904ba1c4702d0ba)
  ```
  cmake ..
  make -j12
  sudo make install
  ```
   
3. Finishing steps
  * make the camera accessable even not in *root*
  ```
  sudo cp scripts/99-k4a.rules /etc/udev/rules.d/
  ```
  
  * add missing *depthengine* dll (can be found [here](./libs/libdepthengine.so.2.0))
  ```
  sudo cp libdepthengine.so.2.0 /usr/lib/x86_64-linux-gpu/
  ```

4. Check Installation
  Run `./viewer_opengl`.

5. Camera Info
```
== Azure Kinect DK Firmware Tool == 
Device Serial Number: 000057201312
Current Firmware Versions:
  RGB camera firmware:      1.6.102
  Depth camera firmware:    1.6.75
  Depth config file:        6109.7
  Audio firmware:           1.6.14
  Build Config:             Production
  Certificate Type:         Microsoft
```
```
===== Device 0: 000057201312 =====
resolution width: 640
resolution height: 576
principal point x: 328.002930
principal point y: 333.227478
focal length x: 504.286377
focal length y: 504.373688
radial distortion coefficients:
k1: 6.564074
k2: 4.790055
k3: 0.263871
k4: 6.888676
k5: 6.996602
k6: 1.347770
center of distortion in Z=1 plane, x: 0.000000
center of distortion in Z=1 plane, y: 0.000000
tangential distortion coefficient x: -0.000009
tangential distortion coefficient y: 0.000084
metric radius: 0.000000

```

6. Unknown Problems:
  * The original visualisation tool `k4aviewer` failed to work with error message: 
  > incompatible audio device
  * The laptop has some wierd noise (coil wine?) when the camera is working.

[Back to Top](#table-of-content)


# Log
### OpenCV CMake Output
```console
-- The CXX compiler identification is GNU 9.3.0
-- The C compiler identification is GNU 9.3.0
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Detected processor: x86_64
-- Could NOT find PythonInterp (missing: PYTHON_EXECUTABLE) (Required is at least version "2.7")
-- Found PythonInterp: /usr/bin/python3 (found suitable version "3.8.5", minimum required is "3.2") 
-- Could NOT find PythonLibs (missing: PYTHON_LIBRARIES PYTHON_INCLUDE_DIRS) (Required is exact version "3.8.5")
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'numpy'
-- Looking for ccache - not found
-- Performing Test HAVE_CXX_FSIGNED_CHAR
-- Performing Test HAVE_CXX_FSIGNED_CHAR - Success
-- Performing Test HAVE_C_FSIGNED_CHAR
-- Performing Test HAVE_C_FSIGNED_CHAR - Success
-- Performing Test HAVE_CXX_W
-- Performing Test HAVE_CXX_W - Success
-- Performing Test HAVE_C_W
-- Performing Test HAVE_C_W - Success
-- Performing Test HAVE_CXX_WALL
-- Performing Test HAVE_CXX_WALL - Success
-- Performing Test HAVE_C_WALL
-- Performing Test HAVE_C_WALL - Success
-- Performing Test HAVE_CXX_WERROR_RETURN_TYPE
-- Performing Test HAVE_CXX_WERROR_RETURN_TYPE - Success
-- Performing Test HAVE_C_WERROR_RETURN_TYPE
-- Performing Test HAVE_C_WERROR_RETURN_TYPE - Success
-- Performing Test HAVE_CXX_WERROR_NON_VIRTUAL_DTOR
-- Performing Test HAVE_CXX_WERROR_NON_VIRTUAL_DTOR - Success
-- Performing Test HAVE_C_WERROR_NON_VIRTUAL_DTOR
-- Performing Test HAVE_C_WERROR_NON_VIRTUAL_DTOR - Failed
-- Performing Test HAVE_CXX_WERROR_ADDRESS
-- Performing Test HAVE_CXX_WERROR_ADDRESS - Success
-- Performing Test HAVE_C_WERROR_ADDRESS
-- Performing Test HAVE_C_WERROR_ADDRESS - Success
-- Performing Test HAVE_CXX_WERROR_SEQUENCE_POINT
-- Performing Test HAVE_CXX_WERROR_SEQUENCE_POINT - Success
-- Performing Test HAVE_C_WERROR_SEQUENCE_POINT
-- Performing Test HAVE_C_WERROR_SEQUENCE_POINT - Success
-- Performing Test HAVE_CXX_WFORMAT
-- Performing Test HAVE_CXX_WFORMAT - Success
-- Performing Test HAVE_C_WFORMAT
-- Performing Test HAVE_C_WFORMAT - Success
-- Performing Test HAVE_CXX_WERROR_FORMAT_SECURITY
-- Performing Test HAVE_CXX_WERROR_FORMAT_SECURITY - Success
-- Performing Test HAVE_C_WERROR_FORMAT_SECURITY
-- Performing Test HAVE_C_WERROR_FORMAT_SECURITY - Success
-- Performing Test HAVE_CXX_WMISSING_DECLARATIONS
-- Performing Test HAVE_CXX_WMISSING_DECLARATIONS - Success
-- Performing Test HAVE_C_WMISSING_DECLARATIONS
-- Performing Test HAVE_C_WMISSING_DECLARATIONS - Success
-- Performing Test HAVE_CXX_WMISSING_PROTOTYPES
-- Performing Test HAVE_CXX_WMISSING_PROTOTYPES - Failed
-- Performing Test HAVE_C_WMISSING_PROTOTYPES
-- Performing Test HAVE_C_WMISSING_PROTOTYPES - Success
-- Performing Test HAVE_CXX_WSTRICT_PROTOTYPES
-- Performing Test HAVE_CXX_WSTRICT_PROTOTYPES - Failed
-- Performing Test HAVE_C_WSTRICT_PROTOTYPES
-- Performing Test HAVE_C_WSTRICT_PROTOTYPES - Success
-- Performing Test HAVE_CXX_WUNDEF
-- Performing Test HAVE_CXX_WUNDEF - Success
-- Performing Test HAVE_C_WUNDEF
-- Performing Test HAVE_C_WUNDEF - Success
-- Performing Test HAVE_CXX_WINIT_SELF
-- Performing Test HAVE_CXX_WINIT_SELF - Success
-- Performing Test HAVE_C_WINIT_SELF
-- Performing Test HAVE_C_WINIT_SELF - Success
-- Performing Test HAVE_CXX_WPOINTER_ARITH
-- Performing Test HAVE_CXX_WPOINTER_ARITH - Success
-- Performing Test HAVE_C_WPOINTER_ARITH
-- Performing Test HAVE_C_WPOINTER_ARITH - Success
-- Performing Test HAVE_CXX_WSHADOW
-- Performing Test HAVE_CXX_WSHADOW - Success
-- Performing Test HAVE_C_WSHADOW
-- Performing Test HAVE_C_WSHADOW - Success
-- Performing Test HAVE_CXX_WSIGN_PROMO
-- Performing Test HAVE_CXX_WSIGN_PROMO - Success
-- Performing Test HAVE_C_WSIGN_PROMO
-- Performing Test HAVE_C_WSIGN_PROMO - Failed
-- Performing Test HAVE_CXX_WUNINITIALIZED
-- Performing Test HAVE_CXX_WUNINITIALIZED - Success
-- Performing Test HAVE_C_WUNINITIALIZED
-- Performing Test HAVE_C_WUNINITIALIZED - Success
-- Performing Test HAVE_CXX_WSUGGEST_OVERRIDE
-- Performing Test HAVE_CXX_WSUGGEST_OVERRIDE - Success
-- Performing Test HAVE_C_WSUGGEST_OVERRIDE
-- Performing Test HAVE_C_WSUGGEST_OVERRIDE - Failed
-- Performing Test HAVE_CXX_WNO_DELETE_NON_VIRTUAL_DTOR
-- Performing Test HAVE_CXX_WNO_DELETE_NON_VIRTUAL_DTOR - Success
-- Performing Test HAVE_C_WNO_DELETE_NON_VIRTUAL_DTOR
-- Performing Test HAVE_C_WNO_DELETE_NON_VIRTUAL_DTOR - Failed
-- Performing Test HAVE_CXX_WNO_UNNAMED_TYPE_TEMPLATE_ARGS
-- Performing Test HAVE_CXX_WNO_UNNAMED_TYPE_TEMPLATE_ARGS - Failed
-- Performing Test HAVE_C_WNO_UNNAMED_TYPE_TEMPLATE_ARGS
-- Performing Test HAVE_C_WNO_UNNAMED_TYPE_TEMPLATE_ARGS - Failed
-- Performing Test HAVE_CXX_WNO_COMMENT
-- Performing Test HAVE_CXX_WNO_COMMENT - Success
-- Performing Test HAVE_C_WNO_COMMENT
-- Performing Test HAVE_C_WNO_COMMENT - Success
-- Performing Test HAVE_CXX_WIMPLICIT_FALLTHROUGH_3
-- Performing Test HAVE_CXX_WIMPLICIT_FALLTHROUGH_3 - Success
-- Performing Test HAVE_C_WIMPLICIT_FALLTHROUGH_3
-- Performing Test HAVE_C_WIMPLICIT_FALLTHROUGH_3 - Success
-- Performing Test HAVE_CXX_WNO_STRICT_OVERFLOW
-- Performing Test HAVE_CXX_WNO_STRICT_OVERFLOW - Success
-- Performing Test HAVE_C_WNO_STRICT_OVERFLOW
-- Performing Test HAVE_C_WNO_STRICT_OVERFLOW - Success
-- Performing Test HAVE_CXX_FDIAGNOSTICS_SHOW_OPTION
-- Performing Test HAVE_CXX_FDIAGNOSTICS_SHOW_OPTION - Success
-- Performing Test HAVE_C_FDIAGNOSTICS_SHOW_OPTION
-- Performing Test HAVE_C_FDIAGNOSTICS_SHOW_OPTION - Success
-- Performing Test HAVE_CXX_WNO_LONG_LONG
-- Performing Test HAVE_CXX_WNO_LONG_LONG - Success
-- Performing Test HAVE_C_WNO_LONG_LONG
-- Performing Test HAVE_C_WNO_LONG_LONG - Success
-- Performing Test HAVE_CXX_PTHREAD
-- Performing Test HAVE_CXX_PTHREAD - Success
-- Performing Test HAVE_C_PTHREAD
-- Performing Test HAVE_C_PTHREAD - Success
-- Performing Test HAVE_CXX_FOMIT_FRAME_POINTER
-- Performing Test HAVE_CXX_FOMIT_FRAME_POINTER - Success
-- Performing Test HAVE_C_FOMIT_FRAME_POINTER
-- Performing Test HAVE_C_FOMIT_FRAME_POINTER - Success
-- Performing Test HAVE_CXX_FFUNCTION_SECTIONS
-- Performing Test HAVE_CXX_FFUNCTION_SECTIONS - Success
-- Performing Test HAVE_C_FFUNCTION_SECTIONS
-- Performing Test HAVE_C_FFUNCTION_SECTIONS - Success
-- Performing Test HAVE_CXX_FDATA_SECTIONS
-- Performing Test HAVE_CXX_FDATA_SECTIONS - Success
-- Performing Test HAVE_C_FDATA_SECTIONS
-- Performing Test HAVE_C_FDATA_SECTIONS - Success
-- Performing Test HAVE_CXX_MSSE (check file: cmake/checks/cpu_sse.cpp)
-- Performing Test HAVE_CXX_MSSE - Success
-- Performing Test HAVE_CXX_MSSE2 (check file: cmake/checks/cpu_sse2.cpp)
-- Performing Test HAVE_CXX_MSSE2 - Success
-- Performing Test HAVE_CXX_MSSE3 (check file: cmake/checks/cpu_sse3.cpp)
-- Performing Test HAVE_CXX_MSSE3 - Success
-- Performing Test HAVE_CXX_MSSSE3 (check file: cmake/checks/cpu_ssse3.cpp)
-- Performing Test HAVE_CXX_MSSSE3 - Success
-- Performing Test HAVE_CXX_MSSE4_1 (check file: cmake/checks/cpu_sse41.cpp)
-- Performing Test HAVE_CXX_MSSE4_1 - Success
-- Performing Test HAVE_CXX_MPOPCNT (check file: cmake/checks/cpu_popcnt.cpp)
-- Performing Test HAVE_CXX_MPOPCNT - Success
-- Performing Test HAVE_CXX_MSSE4_2 (check file: cmake/checks/cpu_sse42.cpp)
-- Performing Test HAVE_CXX_MSSE4_2 - Success
-- Performing Test HAVE_CXX_MF16C (check file: cmake/checks/cpu_fp16.cpp)
-- Performing Test HAVE_CXX_MF16C - Success
-- Performing Test HAVE_CXX_MFMA
-- Performing Test HAVE_CXX_MFMA - Success
-- Performing Test HAVE_CXX_MAVX (check file: cmake/checks/cpu_avx.cpp)
-- Performing Test HAVE_CXX_MAVX - Success
-- Performing Test HAVE_CXX_MAVX2 (check file: cmake/checks/cpu_avx2.cpp)
-- Performing Test HAVE_CXX_MAVX2 - Success
-- Performing Test HAVE_CXX_MAVX512F (check file: cmake/checks/cpu_avx512.cpp)
-- Performing Test HAVE_CXX_MAVX512F - Success
-- Performing Test HAVE_CXX_MAVX512F_MAVX512CD (check file: cmake/checks/cpu_avx512common.cpp)
-- Performing Test HAVE_CXX_MAVX512F_MAVX512CD - Success
-- Performing Test HAVE_CXX_MAVX512F_MAVX512CD_MAVX512VL_MAVX512BW_MAVX512DQ (check file: cmake/checks/cpu_avx512skx.cpp)
-- Performing Test HAVE_CXX_MAVX512F_MAVX512CD_MAVX512VL_MAVX512BW_MAVX512DQ - Success
-- Performing Test HAVE_CPU_BASELINE_FLAGS
-- Performing Test HAVE_CPU_BASELINE_FLAGS - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_SSE4_1
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_SSE4_1 - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_SSE4_2
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_SSE4_2 - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_FP16
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_FP16 - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX2
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX2 - Success
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX512_SKX
-- Performing Test HAVE_CPU_DISPATCH_FLAGS_AVX512_SKX - Success
-- Performing Test HAVE_CXX_FVISIBILITY_HIDDEN
-- Performing Test HAVE_CXX_FVISIBILITY_HIDDEN - Success
-- Performing Test HAVE_C_FVISIBILITY_HIDDEN
-- Performing Test HAVE_C_FVISIBILITY_HIDDEN - Success
-- Performing Test HAVE_CXX_FVISIBILITY_INLINES_HIDDEN
-- Performing Test HAVE_CXX_FVISIBILITY_INLINES_HIDDEN - Success
-- Performing Test HAVE_C_FVISIBILITY_INLINES_HIDDEN
-- Performing Test HAVE_C_FVISIBILITY_INLINES_HIDDEN - Failed
-- Performing Test HAVE_LINK_AS_NEEDED
-- Performing Test HAVE_LINK_AS_NEEDED - Success
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Looking for posix_memalign
-- Looking for posix_memalign - found
-- Looking for malloc.h
-- Looking for malloc.h - found
-- Looking for memalign
-- Looking for memalign - found
-- Check if the system is big endian
-- Searching 16 bit integer
-- Looking for sys/types.h
-- Looking for sys/types.h - found
-- Looking for stdint.h
-- Looking for stdint.h - found
-- Looking for stddef.h
-- Looking for stddef.h - found
-- Check size of unsigned short
-- Check size of unsigned short - done
-- Using unsigned short
-- Check if the system is big endian - little endian
-- Found ZLIB: /usr/lib/x86_64-linux-gnu/libz.so (found suitable version "1.2.11", minimum required is "1.2.3") 
-- Found JPEG: /usr/lib/x86_64-linux-gnu/libjpeg.so (found version "80") 
-- Looking for assert.h
-- Looking for assert.h - found
-- Looking for dlfcn.h
-- Looking for dlfcn.h - found
-- Looking for fcntl.h
-- Looking for fcntl.h - found
-- Looking for inttypes.h
-- Looking for inttypes.h - found
-- Looking for io.h
-- Looking for io.h - not found
-- Looking for limits.h
-- Looking for limits.h - found
-- Looking for memory.h
-- Looking for memory.h - found
-- Looking for search.h
-- Looking for search.h - found
-- Looking for string.h
-- Looking for string.h - found
-- Looking for strings.h
-- Looking for strings.h - found
-- Looking for sys/time.h
-- Looking for sys/time.h - found
-- Looking for unistd.h
-- Looking for unistd.h - found
-- Performing Test C_HAS_inline
-- Performing Test C_HAS_inline - Success
-- Check size of signed short
-- Check size of signed short - done
-- Check size of unsigned short
-- Check size of unsigned short - done
-- Check size of signed int
-- Check size of signed int - done
-- Check size of unsigned int
-- Check size of unsigned int - done
-- Check size of signed long
-- Check size of signed long - done
-- Check size of unsigned long
-- Check size of unsigned long - done
-- Check size of signed long long
-- Check size of signed long long - done
-- Check size of unsigned long long
-- Check size of unsigned long long - done
-- Check size of unsigned char *
-- Check size of unsigned char * - done
-- Check size of size_t
-- Check size of size_t - done
-- Check size of ptrdiff_t
-- Check size of ptrdiff_t - done
-- Check size of INT8
-- Check size of INT8 - failed
-- Check size of INT16
-- Check size of INT16 - failed
-- Check size of INT32
-- Check size of INT32 - failed
-- Looking for floor
-- Looking for floor - found
-- Looking for pow
-- Looking for pow - found
-- Looking for sqrt
-- Looking for sqrt - found
-- Looking for isascii
-- Looking for isascii - found
-- Looking for memset
-- Looking for memset - found
-- Looking for mmap
-- Looking for mmap - found
-- Looking for getopt
-- Looking for getopt - found
-- Looking for memmove
-- Looking for memmove - found
-- Looking for setmode
-- Looking for setmode - not found
-- Looking for strcasecmp
-- Looking for strcasecmp - found
-- Looking for strchr
-- Looking for strchr - found
-- Looking for strrchr
-- Looking for strrchr - found
-- Looking for strstr
-- Looking for strstr - found
-- Looking for strtol
-- Looking for strtol - found
-- Looking for strtol
-- Looking for strtol - found
-- Looking for strtoull
-- Looking for strtoull - found
-- Looking for lfind
-- Looking for lfind - found
-- Performing Test HAVE_SNPRINTF
-- Performing Test HAVE_SNPRINTF - Success
-- Check if the system is big endian
-- Searching 16 bit integer
-- Using unsigned short
-- Check if the system is big endian - little endian
-- Performing Test HAVE_C_WNO_UNUSED_BUT_SET_VARIABLE
-- Performing Test HAVE_C_WNO_UNUSED_BUT_SET_VARIABLE - Success
-- Performing Test HAVE_C_WNO_MISSING_PROTOTYPES
-- Performing Test HAVE_C_WNO_MISSING_PROTOTYPES - Success
-- Performing Test HAVE_C_WNO_MISSING_DECLARATIONS
-- Performing Test HAVE_C_WNO_MISSING_DECLARATIONS - Success
-- Performing Test HAVE_C_WNO_UNDEF
-- Performing Test HAVE_C_WNO_UNDEF - Success
-- Performing Test HAVE_C_WNO_UNUSED
-- Performing Test HAVE_C_WNO_UNUSED - Success
-- Performing Test HAVE_C_WNO_SIGN_COMPARE
-- Performing Test HAVE_C_WNO_SIGN_COMPARE - Success
-- Performing Test HAVE_C_WNO_CAST_ALIGN
-- Performing Test HAVE_C_WNO_CAST_ALIGN - Success
-- Performing Test HAVE_C_WNO_SHADOW
-- Performing Test HAVE_C_WNO_SHADOW - Success
-- Performing Test HAVE_C_WNO_MAYBE_UNINITIALIZED
-- Performing Test HAVE_C_WNO_MAYBE_UNINITIALIZED - Success
-- Performing Test HAVE_C_WNO_POINTER_TO_INT_CAST
-- Performing Test HAVE_C_WNO_POINTER_TO_INT_CAST - Success
-- Performing Test HAVE_C_WNO_INT_TO_POINTER_CAST
-- Performing Test HAVE_C_WNO_INT_TO_POINTER_CAST - Success
-- Performing Test HAVE_C_WNO_MISLEADING_INDENTATION
-- Performing Test HAVE_C_WNO_MISLEADING_INDENTATION - Success
-- Performing Test HAVE_C_WNO_IMPLICIT_FALLTHROUGH
-- Performing Test HAVE_C_WNO_IMPLICIT_FALLTHROUGH - Success
-- Performing Test HAVE_C_WNO_UNUSED_PARAMETER
-- Performing Test HAVE_C_WNO_UNUSED_PARAMETER - Success
-- Performing Test HAVE_CXX_WNO_MISSING_DECLARATIONS
-- Performing Test HAVE_CXX_WNO_MISSING_DECLARATIONS - Success
-- Performing Test HAVE_CXX_WNO_UNUSED_PARAMETER
-- Performing Test HAVE_CXX_WNO_UNUSED_PARAMETER - Success
-- Performing Test HAVE_CXX_WNO_MISSING_PROTOTYPES
-- Performing Test HAVE_CXX_WNO_MISSING_PROTOTYPES - Failed
-- Performing Test HAVE_CXX_WNO_UNDEF
-- Performing Test HAVE_CXX_WNO_UNDEF - Success
-- Performing Test HAVE_C_STD_C99
-- Performing Test HAVE_C_STD_C99 - Success
-- Performing Test HAVE_C_WNO_UNUSED_VARIABLE
-- Performing Test HAVE_C_WNO_UNUSED_VARIABLE - Success
-- Performing Test HAVE_C_WNO_UNUSED_FUNCTION
-- Performing Test HAVE_C_WNO_UNUSED_FUNCTION - Success
-- Found Jasper: /usr/lib/x86_64-linux-gnu/libjasper.so (found version "1.900.1") 
-- Found ZLIB: /usr/lib/x86_64-linux-gnu/libz.so (found version "1.2.11") 
-- Found PNG: /usr/lib/x86_64-linux-gnu/libpng.so (found version "1.6.37") 
-- Looking for /usr/include/libpng/png.h
-- Looking for /usr/include/libpng/png.h - found
-- Looking for semaphore.h
-- Looking for semaphore.h - found
-- Performing Test HAVE_CXX_WNO_SHADOW
-- Performing Test HAVE_CXX_WNO_SHADOW - Success
-- Performing Test HAVE_CXX_WNO_UNUSED
-- Performing Test HAVE_CXX_WNO_UNUSED - Success
-- Performing Test HAVE_CXX_WNO_SIGN_COMPARE
-- Performing Test HAVE_CXX_WNO_SIGN_COMPARE - Success
-- Performing Test HAVE_CXX_WNO_UNINITIALIZED
-- Performing Test HAVE_CXX_WNO_UNINITIALIZED - Success
-- Performing Test HAVE_CXX_WNO_SWITCH
-- Performing Test HAVE_CXX_WNO_SWITCH - Success
-- Performing Test HAVE_CXX_WNO_PARENTHESES
-- Performing Test HAVE_CXX_WNO_PARENTHESES - Success
-- Performing Test HAVE_CXX_WNO_ARRAY_BOUNDS
-- Performing Test HAVE_CXX_WNO_ARRAY_BOUNDS - Success
-- Performing Test HAVE_CXX_WNO_EXTRA
-- Performing Test HAVE_CXX_WNO_EXTRA - Success
-- Performing Test HAVE_CXX_WNO_DEPRECATED_DECLARATIONS
-- Performing Test HAVE_CXX_WNO_DEPRECATED_DECLARATIONS - Success
-- Performing Test HAVE_CXX_WNO_MISLEADING_INDENTATION
-- Performing Test HAVE_CXX_WNO_MISLEADING_INDENTATION - Success
-- Performing Test HAVE_CXX_WNO_DEPRECATED
-- Performing Test HAVE_CXX_WNO_DEPRECATED - Success
-- Performing Test HAVE_CXX_WNO_SUGGEST_OVERRIDE
-- Performing Test HAVE_CXX_WNO_SUGGEST_OVERRIDE - Success
-- Performing Test HAVE_CXX_WNO_INCONSISTENT_MISSING_OVERRIDE
-- Performing Test HAVE_CXX_WNO_INCONSISTENT_MISSING_OVERRIDE - Failed
-- Performing Test HAVE_CXX_WNO_IMPLICIT_FALLTHROUGH
-- Performing Test HAVE_CXX_WNO_IMPLICIT_FALLTHROUGH - Success
-- Performing Test HAVE_CXX_WNO_TAUTOLOGICAL_COMPARE
-- Performing Test HAVE_CXX_WNO_TAUTOLOGICAL_COMPARE - Success
-- Performing Test HAVE_CXX_WNO_REORDER
-- Performing Test HAVE_CXX_WNO_REORDER - Success
-- Performing Test HAVE_CXX_WNO_UNUSED_RESULT
-- Performing Test HAVE_CXX_WNO_UNUSED_RESULT - Success
-- Performing Test HAVE_CXX_WNO_CLASS_MEMACCESS
-- Performing Test HAVE_CXX_WNO_CLASS_MEMACCESS - Success
-- Checking for modules 'gstreamer-base-1.0;gstreamer-video-1.0;gstreamer-app-1.0;gstreamer-riff-1.0;gstreamer-pbutils-1.0'
--   No package 'gstreamer-base-1.0' found
--   No package 'gstreamer-video-1.0' found
--   No package 'gstreamer-app-1.0' found
--   No package 'gstreamer-riff-1.0' found
--   No package 'gstreamer-pbutils-1.0' found
-- Checking for modules 'gstreamer-base-0.10;gstreamer-video-0.10;gstreamer-app-0.10;gstreamer-riff-0.10;gstreamer-pbutils-0.10'
--   No package 'gstreamer-base-0.10' found
--   No package 'gstreamer-video-0.10' found
--   No package 'gstreamer-app-0.10' found
--   No package 'gstreamer-riff-0.10' found
--   No package 'gstreamer-pbutils-0.10' found
-- Checking for module 'libdc1394-2'
--   Found libdc1394-2, version 2.2.5
-- Looking for linux/videodev.h
-- Looking for linux/videodev.h - not found
-- Looking for linux/videodev2.h
-- Looking for linux/videodev2.h - found
-- Looking for sys/videoio.h
-- Looking for sys/videoio.h - not found
-- Checking for modules 'libavcodec;libavformat;libavutil;libswscale'
--   Found libavcodec, version 58.54.100
--   Found libavformat, version 58.29.100
--   Found libavutil, version 56.31.100
--   Found libswscale, version 5.5.100
-- Checking for module 'libavresample'
--   No package 'libavresample' found
-- IPPICV: Download: ippicv_2020_lnx_intel64_20191018_general.tgz
-- found Intel IPP (ICV version): 2020.0.0 [2020.0.0 Gold]
-- at: /home/yohann/SLAMs/depend/opencv-3.4.13/build/3rdparty/ippicv/ippicv_lnx/icv
-- found Intel IPP Integration Wrappers sources: 2020.0.0
-- at: /home/yohann/SLAMs/depend/opencv-3.4.13/build/3rdparty/ippicv/ippicv_lnx/iw
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD
-- Performing Test CMAKE_HAVE_LIBC_PTHREAD - Failed
-- Looking for pthread_create in pthreads
-- Looking for pthread_create in pthreads - not found
-- Looking for pthread_create in pthread
-- Looking for pthread_create in pthread - found
-- Found Threads: TRUE  
-- CUDA detected: 11.1
-- CUDA NVCC target flags: -gencode;arch=compute_35,code=sm_35;-gencode;arch=compute_37,code=sm_37;-gencode;arch=compute_50,code=sm_50;-gencode;arch=compute_52,code=sm_52;-gencode;arch=compute_60,code=sm_60;-gencode;arch=compute_61,code=sm_61;-gencode;arch=compute_70,code=sm_70;-gencode;arch=compute_75,code=sm_75;-gencode;arch=compute_80,code=sm_80;-gencode;arch=compute_86,code=sm_86;-D_FORCE_INLINES
-- Could not find OpenBLAS include. Turning OpenBLAS_FOUND off
-- Could not find OpenBLAS lib. Turning OpenBLAS_FOUND off
-- Could NOT find Atlas (missing: Atlas_CLAPACK_INCLUDE_DIR Atlas_CBLAS_LIBRARY Atlas_BLAS_LIBRARY) 
-- Looking for sgemm_
-- Looking for sgemm_ - not found
-- Looking for sgemm_
-- Looking for sgemm_ - found
-- Found BLAS: /usr/lib/x86_64-linux-gnu/libopenblas.so  
-- Looking for cheev_
-- Looking for cheev_ - found
-- A library with LAPACK API found.
-- Performing Test HAVE_CXX_WNO_UNUSED_LOCAL_TYPEDEFS
-- Performing Test HAVE_CXX_WNO_UNUSED_LOCAL_TYPEDEFS - Success
-- Performing Test HAVE_CXX_WNO_SIGN_PROMO
-- Performing Test HAVE_CXX_WNO_SIGN_PROMO - Success
-- Performing Test HAVE_CXX_WNO_TAUTOLOGICAL_UNDEFINED_COMPARE
-- Performing Test HAVE_CXX_WNO_TAUTOLOGICAL_UNDEFINED_COMPARE - Failed
-- Performing Test HAVE_CXX_WNO_IGNORED_QUALIFIERS
-- Performing Test HAVE_CXX_WNO_IGNORED_QUALIFIERS - Success
-- Performing Test HAVE_CXX_WNO_UNUSED_FUNCTION
-- Performing Test HAVE_CXX_WNO_UNUSED_FUNCTION - Success
-- Performing Test HAVE_CXX_WNO_UNUSED_CONST_VARIABLE
-- Performing Test HAVE_CXX_WNO_UNUSED_CONST_VARIABLE - Success
-- Performing Test HAVE_CXX_WNO_SHORTEN_64_TO_32
-- Performing Test HAVE_CXX_WNO_SHORTEN_64_TO_32 - Failed
-- Performing Test HAVE_CXX_WNO_INVALID_OFFSETOF
-- Performing Test HAVE_CXX_WNO_INVALID_OFFSETOF - Success
-- Performing Test HAVE_CXX_WNO_ENUM_COMPARE_SWITCH
-- Performing Test HAVE_CXX_WNO_ENUM_COMPARE_SWITCH - Failed
-- VTK is not found. Please set -DVTK_DIR in CMake to VTK build directory, or to VTK install subdirectory with VTKConfig.cmake file
-- Looking for dlerror in dl
-- Looking for dlerror in dl - found
-- Performing Test HAVE_CXX_WNO_STRICT_ALIASING
-- Performing Test HAVE_CXX_WNO_STRICT_ALIASING - Success
-- Performing Test HAVE_CXX_WNO_UNUSED_VARIABLE
-- Performing Test HAVE_CXX_WNO_UNUSED_VARIABLE - Success
-- Performing Test HAVE_CXX_WNO_ENUM_COMPARE
-- Performing Test HAVE_CXX_WNO_ENUM_COMPARE - Success
-- OpenCV Python: during development append to PYTHONPATH: /home/yohann/SLAMs/depend/opencv-3.4.13/build/python_loader
-- Caffe:   NO
-- Protobuf:   NO
-- Glog:   NO
-- Checking for module 'freetype2'
--   Found freetype2, version 23.1.17
-- Checking for module 'harfbuzz'
--   Found harfbuzz, version 2.6.4
-- freetype2:   YES (ver 23.1.17)
-- harfbuzz:    YES (ver 2.6.4)
-- Could NOT find HDF5 (missing: HDF5_LIBRARIES HDF5_INCLUDE_DIRS) (found version "")
-- Module opencv_ovis disabled because OGRE3D was not found
-- No preference for use of exported gflags CMake configuration set, and no hints for include/library directories provided. Defaulting to preferring an installed/exported gflags CMake configuration if available.
-- Failed to find installed gflags CMake configuration, searching for gflags build directories exported with CMake.
-- Failed to find gflags - Failed to find an installed/exported CMake configuration for gflags, will perform search for installed gflags components.
-- Failed to find gflags - Could not find gflags include directory, set GFLAGS_INCLUDE_DIR to directory containing gflags/gflags.h
-- Failed to find glog - Could not find glog include directory, set GLOG_INCLUDE_DIR to directory containing glog/logging.h
-- Module opencv_sfm disabled because the following dependencies are not found: Eigen Glog/Gflags
-- Checking for module 'tesseract'
--   No package 'tesseract' found
-- Tesseract:   NO
-- Allocator metrics storage type: 'long long'
-- Performing Test HAVE_CXX_WNO_UNUSED_BUT_SET_VARIABLE
-- Performing Test HAVE_CXX_WNO_UNUSED_BUT_SET_VARIABLE - Success
-- Performing Test HAVE_CXX_WNO_OVERLOADED_VIRTUAL
-- Performing Test HAVE_CXX_WNO_OVERLOADED_VIRTUAL - Success
-- xfeatures2d/boostdesc: Download: boostdesc_bgm.i
-- xfeatures2d/boostdesc: Download: boostdesc_bgm_bi.i
-- xfeatures2d/boostdesc: Download: boostdesc_bgm_hd.i
-- xfeatures2d/boostdesc: Download: boostdesc_binboost_064.i
-- xfeatures2d/boostdesc: Download: boostdesc_binboost_128.i
-- xfeatures2d/boostdesc: Download: boostdesc_binboost_256.i
-- xfeatures2d/boostdesc: Download: boostdesc_lbgm.i
-- xfeatures2d/vgg: Download: vgg_generated_48.i
-- xfeatures2d/vgg: Download: vgg_generated_64.i
-- xfeatures2d/vgg: Download: vgg_generated_80.i
-- xfeatures2d/vgg: Download: vgg_generated_120.i
-- data: Download: face_landmark_model.dat
-- 
-- General configuration for OpenCV 3.4.13 =====================================
--   Version control:               unknown
-- 
--   Extra modules:
--     Location (extra):            /home/yohann/SLAMs/depend/opencv_contrib-3.4.13/modules
--     Version control (extra):     unknown
-- 
--   Platform:
--     Timestamp:                   2021-02-15T14:03:56Z
--     Host:                        Linux 5.10.14-051014-generic x86_64
--     CMake:                       3.16.3
--     CMake generator:             Unix Makefiles
--     CMake build tool:            /usr/bin/make
--     Configuration:               RELEASE
-- 
--   CPU/HW features:
--     Baseline:                    SSE SSE2 SSE3
--       requested:                 SSE3
--     Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2 AVX512_SKX
--       requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2 AVX512_SKX
--       SSE4_1 (16 files):         + SSSE3 SSE4_1
--       SSE4_2 (2 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
--       FP16 (1 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
--       AVX (6 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
--       AVX2 (30 files):           + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2
--       AVX512_SKX (7 files):      + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2 AVX_512F AVX512_COMMON AVX512_SKX
-- 
--   C/C++:
--     Built as dynamic libs?:      YES
--     C++11:                       YES
--     C++ Compiler:                /usr/bin/c++  (ver 9.3.0)
--     C++ flags (Release):         -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -O3 -DNDEBUG  -DNDEBUG
--     C++ flags (Debug):           -fsigned-char -W -Wall -Werror=return-type -Werror=non-virtual-dtor -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wundef -Winit-self -Wpointer-arith -Wshadow -Wsign-promo -Wuninitialized -Wsuggest-override -Wno-delete-non-virtual-dtor -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -fvisibility-inlines-hidden -g  -O0 -DDEBUG -D_DEBUG
--     C Compiler:                  /usr/bin/cc
--     C flags (Release):           -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -O3 -DNDEBUG  -DNDEBUG
--     C flags (Debug):             -fsigned-char -W -Wall -Werror=return-type -Werror=address -Werror=sequence-point -Wformat -Werror=format-security -Wmissing-declarations -Wmissing-prototypes -Wstrict-prototypes -Wundef -Winit-self -Wpointer-arith -Wshadow -Wuninitialized -Wno-comment -Wimplicit-fallthrough=3 -Wno-strict-overflow -fdiagnostics-show-option -Wno-long-long -pthread -fomit-frame-pointer -ffunction-sections -fdata-sections  -msse -msse2 -msse3 -fvisibility=hidden -g  -O0 -DDEBUG -D_DEBUG
--     Linker flags (Release):      -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
--     Linker flags (Debug):        -Wl,--exclude-libs,libippicv.a -Wl,--exclude-libs,libippiw.a   -Wl,--gc-sections -Wl,--as-needed  
--     ccache:                      NO
--     Precompiled headers:         NO
--     Extra dependencies:          m pthread cudart_static -lpthread dl rt nppc nppial nppicc nppidei nppif nppig nppim nppist nppisu nppitc npps cublas cufft -L/usr/local/cuda-11.1/lib64 -L/usr/lib/x86_64-linux-gnu
--     3rdparty dependencies:
-- 
--   OpenCV modules:
--     To be built:                 aruco bgsegm bioinspired calib3d ccalib core cudaarithm cudabgsegm cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev cvv datasets dnn dnn_objdetect dpm face features2d flann freetype fuzzy hfs highgui img_hash imgcodecs imgproc line_descriptor ml objdetect optflow phase_unwrapping photo plot reg rgbd saliency shape stereo stitching structured_light superres surface_matching text tracking ts video videoio videostab xfeatures2d ximgproc xobjdetect xphoto
--     Disabled:                    cudacodec java_bindings_generator python_bindings_generator world
--     Disabled by dependency:      -
--     Unavailable:                 cnn_3dobj hdf java matlab ovis python2 python3 sfm viz
--     Applications:                tests perf_tests apps
--     Documentation:               NO
--     Non-free algorithms:         YES
-- 
--   GUI: 
--     QT:                          YES (ver 5.12.8)
--       QT OpenGL support:         NO
--     GTK+:                        NO
--     VTK support:                 NO
-- 
--   Media I/O: 
--     ZLib:                        /usr/lib/x86_64-linux-gnu/libz.so (ver 1.2.11)
--     JPEG:                        /usr/lib/x86_64-linux-gnu/libjpeg.so (ver 80)
--     WEBP:                        build (ver encoder: 0x020f)
--     PNG:                         /usr/lib/x86_64-linux-gnu/libpng.so (ver 1.6.37)
--     TIFF:                        build (ver 42 - 4.0.10)
--     JPEG 2000:                   /usr/lib/x86_64-linux-gnu/libjasper.so (ver 1.900.1)
--     OpenEXR:                     build (ver 2.3.0)
--     HDR:                         YES
--     SUNRASTER:                   YES
--     PXM:                         YES
-- 
--   Video I/O:
--     DC1394:                      YES (ver 2.2.5)
--     FFMPEG:                      YES
--       avcodec:                   YES (ver 58.54.100)
--       avformat:                  YES (ver 58.29.100)
--       avutil:                    YES (ver 56.31.100)
--       swscale:                   YES (ver 5.5.100)
--       avresample:                NO
--     GStreamer:                   NO
--     libv4l/libv4l2:              NO
--     v4l/v4l2:                    linux/videodev2.h
-- 
--   Parallel framework:            pthreads
-- 
--   Trace:                         YES (with Intel ITT)
-- 
--   Other third-party libraries:
--     Intel IPP:                   2020.0.0 Gold [2020.0.0]
--            at:                   /home/yohann/SLAMs/depend/opencv-3.4.13/build/3rdparty/ippicv/ippicv_lnx/icv
--     Intel IPP IW:                sources (2020.0.0)
--               at:                /home/yohann/SLAMs/depend/opencv-3.4.13/build/3rdparty/ippicv/ippicv_lnx/iw
--     Lapack:                      NO
--     Eigen:                       NO
--     Custom HAL:                  NO
--     Protobuf:                    build (3.5.1)
-- 
--   NVIDIA CUDA:                   YES (ver 11.1, CUFFT CUBLAS)
--     NVIDIA GPU arch:             35 37 50 52 60 61 70 75 80 86
--     NVIDIA PTX archs:
-- 
--   OpenCL:                        YES (no extra features)
--     Include path:                /home/yohann/SLAMs/depend/opencv-3.4.13/3rdparty/include/opencl/1.2
--     Link libraries:              Dynamic load
-- 
--   Python (for build):            /usr/bin/python3
-- 
--   Install to:                    /usr/local
-- -----------------------------------------------------------------
-- 
-- Configuring done
-- Generating done
-- Build files have been written to: /home/yohann/SLAMs/depend/opencv-3.4.13/build
```

### Tensorflow Configuration Example
```console
You have bazel 3.7.2 installed.
Please specify the location of python. [Default is /home/yohann/anaconda3/envs/NOCS/bin/python3]: 


Found possible Python library paths:
  /home/yohann/anaconda3/envs/NOCS/lib/python3.7/site-packages
Please input the desired Python library path to use.  Default is [/home/yohann/anaconda3/envs/NOCS/lib/python3.7/site-packages]

Do you wish to build TensorFlow with ROCm support? [y/N]: 
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Do you wish to build TensorFlow with TensorRT support? [y/N]: 
No TensorRT support will be enabled for TensorFlow.

Found CUDA 11.1 in:
    /usr/local/cuda-11.1/targets/x86_64-linux/lib
    /usr/local/cuda-11.1/targets/x86_64-linux/include
Found cuDNN 8 in:
    /usr/lib/x86_64-linux-gnu
    /usr/include


Please specify a list of comma-separated CUDA compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus. Each capability can be specified as "x.y" or "compute_xy" to include both virtual and binary GPU code, or as "sm_xy" to only include the binary code.
Please note that each additional compute capability significantly increases your build time and binary size, and that TensorFlow only supports compute capabilities >= 3.5 [Default is: 8.6]: 


Do you want to use clang as CUDA compiler? [y/N]: 
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 


Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -Wno-sign-compare]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
	--config=mkl         	# Build with MKL support.
	--config=mkl_aarch64 	# Build with oneDNN support for Aarch64.
	--config=monolithic  	# Config for mostly static monolithic build.
	--config=numa        	# Build with NUMA support.
	--config=dynamic_kernels	# (Experimental) Build kernels into separate shared objects.
	--config=v2          	# Build TensorFlow 2.x instead of 1.x.
Preconfigured Bazel build configs to DISABLE default on features:
	--config=noaws       	# Disable AWS S3 filesystem support.
	--config=nogcp       	# Disable GCP support.
	--config=nohdfs      	# Disable HDFS support.
	--config=nonccl      	# Disable NVIDIA NCCL support.
Configuration finished
```

### Tensorflow Bazel Build Output
```console
bazel build --config=cuda //tensorflow/tools/pip_package:build_pip_package -j 8
WARNING: The following configs were expanded more than once: [cuda, using_cuda]. For repeatable flags, repeats are counted twice and may lead to unexpected behavior.
INFO: Options provided by the client:
  Inherited 'common' options: --isatty=1 --terminal_columns=120
INFO: Reading rc options for 'build' from /home/yohann/NNs/tensorflow/.bazelrc:
  Inherited 'common' options: --experimental_repo_remote_exec
INFO: Reading rc options for 'build' from /home/yohann/NNs/tensorflow/.bazelrc:
  'build' options: --apple_platform_type=macos --define framework_shared_object=true --java_toolchain=@org_tensorflow//third_party/toolchains/java:tf_java_toolchain --host_java_toolchain=@org_tensorflow//third_party/toolchains/java:tf_java_toolchain --define=tensorflow_enable_mlir_generated_gpu_kernels=0 --define=use_fast_cpp_protos=true --define=allow_oversize_protos=true --spawn_strategy=standalone -c opt --announce_rc --define=grpc_no_ares=true --noincompatible_remove_legacy_whole_archive --noincompatible_prohibit_aapt1 --enable_platform_specific_config --config=short_logs --config=v2
INFO: Reading rc options for 'build' from /home/yohann/NNs/tensorflow/.tf_configure.bazelrc:
  'build' options: --action_env PYTHON_BIN_PATH=/home/yohann/anaconda3/envs/NOCS/bin/python3 --action_env PYTHON_LIB_PATH=/home/yohann/anaconda3/envs/NOCS/lib/python3.7/site-packages --python_path=/home/yohann/anaconda3/envs/NOCS/bin/python3 --config=xla --action_env CUDA_TOOLKIT_PATH=/usr/local/cuda-11.1 --action_env TF_CUDA_COMPUTE_CAPABILITIES=8.6 --action_env LD_LIBRARY_PATH=/usr/local/cuda-11.1/lib64: --action_env GCC_HOST_COMPILER_PATH=/usr/bin/x86_64-linux-gnu-gcc-9 --config=cuda --action_env TF_CONFIGURE_IOS=0
INFO: Found applicable config definition build:short_logs in file /home/yohann/NNs/tensorflow/.bazelrc: --output_filter=DONT_MATCH_ANYTHING
INFO: Found applicable config definition build:v2 in file /home/yohann/NNs/tensorflow/.bazelrc: --define=tf_api_version=2 --action_env=TF2_BEHAVIOR=1
INFO: Found applicable config definition build:xla in file /home/yohann/NNs/tensorflow/.bazelrc: --define=with_xla_support=true
INFO: Found applicable config definition build:cuda in file /home/yohann/NNs/tensorflow/.bazelrc: --config=using_cuda --define=using_cuda_nvcc=true
INFO: Found applicable config definition build:using_cuda in file /home/yohann/NNs/tensorflow/.bazelrc: --define=using_cuda=true --action_env TF_NEED_CUDA=1 --crosstool_top=@local_config_cuda//crosstool:toolchain --define=tensorflow_enable_mlir_generated_gpu_kernels=1
INFO: Found applicable config definition build:cuda in file /home/yohann/NNs/tensorflow/.bazelrc: --config=using_cuda --define=using_cuda_nvcc=true
INFO: Found applicable config definition build:using_cuda in file /home/yohann/NNs/tensorflow/.bazelrc: --define=using_cuda=true --action_env TF_NEED_CUDA=1 --crosstool_top=@local_config_cuda//crosstool:toolchain --define=tensorflow_enable_mlir_generated_gpu_kernels=1
INFO: Found applicable config definition build:linux in file /home/yohann/NNs/tensorflow/.bazelrc: --copt=-w --host_copt=-w --define=PREFIX=/usr --define=LIBDIR=$(PREFIX)/lib --define=INCLUDEDIR=$(PREFIX)/include --define=PROTOBUF_INCLUDE_PATH=$(PREFIX)/include --cxxopt=-std=c++14 --host_cxxopt=-std=c++14 --config=dynamic_kernels
INFO: Found applicable config definition build:dynamic_kernels in file /home/yohann/NNs/tensorflow/.bazelrc: --define=dynamic_loaded_kernels=true --copt=-DAUTOLOAD_DYNAMIC_KERNELS
INFO: Analyzed target //tensorflow/tools/pip_package:build_pip_package (423 packages loaded, 35903 targets configured).
INFO: Found 1 target...
INFO: Deleting stale sandbox base /home/yohann/.cache/bazel/_bazel_yohann/65412b27117725ef7ef78067e0377b6c/sandbox
Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package
INFO: Elapsed time: 9425.628s, Critical Path: 204.99s
INFO: 13511 processes: 198 internal, 13313 local.
INFO: Build completed successfully, 13511 total actions
```

[Back to Top](#table-of-content)