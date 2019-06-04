# Raspbian Install on Raspberry Pi 3
 References:
  * [Raspbian](https://www.raspberrypi.org/downloads/raspbian/)

## Prepare micro SD card
1. Format a micro SD card (should be atleast a 64 GB class 10 card)
* Insert sd card into an external card reader or the card reader of the laptop
* Erase & Format USB drive

* **Windows 10**
    * Right-click USB drive and select **Format**
    * Then type **disk management** in the **start menu** option on windows.
    * Here right click on the sd card memory partitions and select **delete partition** for each of them.
    * The partitions will show up as **unallocated volume**. 
    * All of the deleted partitions will form one big partition same as the size of the sd card.
    * Now right click on this unallocated volume and selecte **create new volume**.
    * This will also give the options to create multiple partitions, but do do that.
    * Keep the partition type as **exFAT**.
    * Once the partition is created it is ready for writing the raspbian image.
* Navigate to the [Raspbian download site](https://www.raspberrypi.org/downloads/raspbian/)
    * Download the **Raspbian image with recommended software**.
    * Download and install **Win32DiskImager** and **7zip**.
    * Unzip the downloaded image using 7zip into a **.img** file.
    * Open Win32DiskImager
    * Click **Select Image** and browse to the **.img** file
    * Select the sd card Drive
    * Click **Write**
    * This may take about **60 minutes** to complete.

## Initial setup
1. Once the installation is done, insert the sd card into the raspberry pi.
* The following devices will need to be connected to the UP Board:
  * Keyboard, Mouse, Monitor, Wifi dongle (for internet access).
* A wifi signal should be available as well.
* **Power up** the board.
* Set the **location** and **time zone** and **password**
* Select the **wifi** and connect to it.
* The board may do some initial updates, let it happen (it may take 60 minutes).

## Increase the Swap Space
* References: [Link](https://wpitchoune.net/tricks/raspberry_pi3_increase_swap_size.html)
  ```
  sudo dphys-swapfile swapoff
  ```
  As root, edit the file **/etc/dphys-swapfile** and modify the variable **CONF_SWAPSIZE** to **2048**.
  ```
  sudo dphys-swapfile swapon
  sudo reboot
  ```

## Install Tensorflow 1.12
  * Reference: [Youtube](https://www.youtube.com/watch?v=zuOiqpWZF2s)

1. Install Dependencies
  ```
  cd ~
  sudo apt install libatlas-base-dev
  ```
* Downloaded the grpcio tar file from this [link](https://pypi.org/project/grpcio/#files).
  * Then unzipped it and then get into the decompressed folder
  * Run the 'setup.py' file as
  ```
  sudo python3 setup.py install
  ```
  
  This took like 20 minutes on the raspberry pi. There will be a lot of **cc1plus** warning messages, but ignore them.
  When installing tensorflow with **pip3**, this **grpcio** package is supposed to get installed automatically. 
  But pip checks the hash files internally and somehow those were not matching with the calculated hash file 
  giving an error like the following:

    'THESE PACKAGES DO NOT MATCH THE HASHES FROM THE REQUIREMENTS FILE. If you have updated the package versions, please update the hashes. 
    Otherwise, examine the package contents carefully; someone may have tampered with them. opencv-contrib-python-headless from 
    https://www.piwheels.org/simple/opencv-contrib-python-headless/opencv_contrib_python_headless-3.4.3.18-cp35-cp35m-linux_armv6l.whl#sha256=ff894c0cc7c98b05b7b260a1dc462e7ad0a4220b042072fc0134a2b7a92bc4a5: 
    Expected sha256 ff894c0cc7c98b05b7b260a1dc462e7ad0a4220b042072fc0134a2b7a92bc4a5 
    Got 4119d8c56d19ef044c1faca317dd10f2bb3b50cbee77426a22feca9b641c5637'

  So we chose this method to bypass pip to install the **grpcio** package.

* Download Tensorflow:
  * Go to the [tensorflow download github page](https://github.com/lhelontra/tensorflow-on-arm/releases)
  * Find the tensorflow for the raspberry pi from here.
  * For the tensorflow version **1.12** that we have installed here for **python3**, the link will be like **https://github.com/lhelontra/tensorflow-on-arm/releases/download/v1.12.0/tensorflow-1.12.0-cp35-none-linux_armv7l.whl**.
  * Notice the **35** in the link. That signifies that it is meant for **python3**. If there was a **27** in this link, then that would have been for **python2**.

* Install Tensorflow
  cd ~
  mkdir tensorflow
  cd tensorflow
  ```
* Run the .whl file for tensorflow:
  ```
  pip3 install https://github.com/lhelontra/tensorflow-on-arm/releases/download/v1.12.0/tensorflow-1.12.0-cp35-none-linux_armv7l.whl
  ```
  or alternatively if the download is taking too long and is failing often, 
  then you can download this file directly from the github by clicking on it and then put it in a tensorflow directory in the home folder of the raspberry pi and then run the command.
  ```
  sudo pip3 install tensorflow-1.12.0-cp35-none-linux_armv7l.whl
  ```
  This may take like 15 minutes.

* Test installation with Python
  ```
  python3
  import tensorflow as tf
  hello = tf.constant('Hello, TensorFlow!')
  sess = tf.Session()
  print(sess.run(hello))
  ```

## Install OpenCV 3.4 (~4 hr)
  * Reference: [OpenCV Docs](https://docs.opencv.org/3.4.3/d7/d9f/tutorial_linux_install.html)
  * Full list of [CMake Options](https://github.com/opencv/opencv/blob/master/CMakeLists.txt#L198)

1. Install Dependencies
  ```
  sudo apt update
  sudo apt install python3-matplotlib build-essential cmake git pkg-config libavcodec-dev \
      libavformat-dev libswscale-dev python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev \
      libjasper-dev libdc1394-22-dev libxine2-dev libv4l-dev libav-tools unzip libdc1394-22 \
      libpng12-dev libtiff5-dev tar dtrx libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev \
      libvorbis-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libqt4-dev \
      libmp3lame-dev libxvidcore-dev x264 libgtk2.0-dev
  ```
* Download OpenCV and OpenCV contrib source code
  ```
  cd ~
  mkdir OpenCV
  cd OpenCV
  git clone -b 3.4 --single-branch https://github.com/opencv/opencv.git
  git clone -b 3.4 --single-branch https://github.com/opencv/opencv_contrib.git
  ```
  
* Build from source (~40 minutes)
  ```
  cd ~/OpenCV/opencv
  mkdir release
  cd release
  sudo cmake -D CMAKE_BUILD_TYPE=RELEASE -D BUILD_NEW_PYTHON_SUPPORT=ON -D INSTALL_C_EXAMPLES=ON -D INSTALL_PYTHON_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_GTK=ON -D WITH_QT=ON -D WITH_GSTREAMER_0_10=ON -D WITH_TBB=ON -D BUILD_TBB=ON -D WITH_OPENGL-ES=ON -D WITH_V4L=ON -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib/modules ..
  sudo make -j4
  sudo make install
  ```
* Create config file (if it doesn't exist)
  * `sudo nano /etc/ld.so.conf.d/opencv.conf`
  * Add `/usr/local/lib/` to the file
  * `sudo ldconfig`
* Test installation with Python
  ```
  python
  import cv2
  recognizer = cv2.face.LBPHFaceRecognizer_create()
  detector = cv2.CascadeClassifier("haarcascade_frontalface_default.xml")
  ```
* Remove OpenCV source to free up storage
  * `sudo rm -r ~/OpenCV`

