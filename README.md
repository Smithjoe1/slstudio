## SLStudio -- Real Time Structured Light

This software is designed to enable anyone to implement a custom 3D structured light scanner using a single camera and light projector. It is modular and has a focus on processing speed, enabling real-time structured light capture at 20 Hz and more. When using standard commercial projector and a webcam, the obtainable speed is lower due to the lack of hardware triggering.

When using the software in academic work, please consider citing the following publication.

Wilm et al., *SLStudio: Open-Source Framework for Real-Time Structured Light*, IPTA 2014 [IEEE Xplore](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?reload=true&arnumber=7002001)

Usage is permitted under GPLv3 license since August 2021. Hence, any distributed (internally or externally) derivative work must be published openly. Use before that date constitutes copyright violation. A commercial license can be obtained for commercial users not wanting to disclose their source code (see License section). Feature/functionalty development on a contract basis are also offered. Please contact the author if interested.

## Realtime setup
![Realtime setup with optical filters and enclosure](example_data/realtime_trigger_enclosure.jpg?raw=true "Realtime setup with optical filters and enclosure")

## Demo videos
[![SLStudio: Real-time 2x3 PSP](http://img.youtube.com/vi/tti4-9ADYLs/0.jpg)](https://www.youtube.com/watch?v=tti4-9ADYLs)
[![SLStudio: Calibration](http://img.youtube.com/vi/swszXuPxGZI/0.jpg)](https://www.youtube.com/watch?v=swszXuPxGZI)
[![SLStudio: Example data](http://img.youtube.com/vi/B0P4ZdHPTqA/0.jpg)](https://youtu.be/B0P4ZdHPTqA)

## Compiling and installing
SLStudio is being developed with qmake (QtCreator). The /src/SLStudio.pro file is the project file which contains all information about the project and its dependencies needed for compiling. 
It is has a number of dependencies that you need to install before being able to compile the program on your machine:
* Qt 5.X (4.x or 6.x could be used with relatively few modifications)
* OpenCV 4.x or newer (3.x can be used with relatively few modifications)
* Point Cloud Library 1.7 or newer
* VTK 9
* VTK Visualizer (https://github.com/codebydant/pcl_visualizer)
* Boost 
* Eigen
* FLANN
* GLEW
* Depending on your camera: libdc1394, FlyCapture API, XIMEA xiApi, IDS Imaging uEye API

The project has successfully been compiled on Ubuntu 22.04, OS X 10.9 and Windows 7. The tested and recommended OS is Ubuntu 22.04.

## Using components
The mathematical/algorithmic and hardware interface parts reside in individual classes from the presentation layer and business logic. The algorithmic classes (Codec*,  Triangulator, Camera, Projector) have very few external dependencies and are designed to be used on their own or in different environments. The Matlab interface shows what this might look like. 

While the algorithm classes could easily be separated into a static/dynamic library, we currently see little value in doing so, and recommend to copy the relevant classes to a new project. Please note license terms.

### Arch 6.1.111
#insall vtk9
sudo pacman -S vtk

#Build pcl
sudo yay -S pcl-git    
git clone https://github.com/codebydant/pcl_visualizer.git
cd pcl_visualizer                                         ✔ 
mkdir build
cd build
cmake ..
make
make install

#Add the other depenancies
sudo pacman -S qt5-base opencv boost eigen glew

#clone this library
cd ~
git clone https://github.com/Smithjoe1/slstudio.git
cd ~/slstudio
qmake ./src/SLStudio.pro
make

Run the file
./SLStudio 




### Ubuntu 22.04
Ubuntu also has all of the dependencies available as packages (except camera libraries). Running the following line should have you (almost) set:
```
    sudo apt-get install libpcl-dev libopencv-dev libvtk9-qt-dev libglew-dev freeglut3-dev qtcreator libusb-1.0-0-dev qmake6 libxrandr-dev
```
One advantage of using Ubuntu is that you are able to render structured light patterns on a secondary X screen, which does not interfere with your main screen in which Gnome desktop and the SLStudio GUI run. Usually, this is an unusual use-case, as normally you are able to move the mouse or windows onto the second screen or use ALT-Tab. However, by setting up two X Screens in xorg.conf with a gap in between them, you can make Gnome completely ignore the second screen so it is only SLStudio that draws onto it. This also depends on your graphics driver supporting multiple X screens (work with current proprietary nVidia and AMD drivers).

### OS X 10.X
On OS X, the dependencies are available through MacPorts. You will not get completely independent screens, so your measurements may be corrupted by GUI activity. Otherwise the program runs well.

### Windows 7/8/10
On Windows machines, OpenCV and PCL are available as downloads, but since everything needs to be ABI compatible, it is probably best to compile the dependencies from source with the same compiler (e.g. MSVC) which is rather time-consuming but possible. It is some time since we have last compiled on Windows, but all of the code should be compatible. 

### Camera APIs
Because high performance operation of cameras often requires vendor- or device-specific features or setup, we choose not to use an intermediate layer such as OpenCV to communicate with the camera. Currently, wrappers for a number of camera APIs is provided, and we invite anyone who implements others to contribute them. Be aware that we have only tested on a limited number of cameras, and some of the configurations performed may only apply to our specific models (e.g. images are assumed to be in grayscale). In any case, you may need to modify some of the camera code to get this to work with hardware triggering (required for real time performance).

### Matlab wrappers
The project also contains Matlab mex-wrappers for the OpenGL projector and Camera. This makes it possible to e.g. determine the gamma response of your camera-projector setup, or other debugging tasks. The mex-wrappers are compiled from Matlab by running matlab/make.m. Tested on Ubuntu only.

## Running

### Projector source
SLStudio can run in different projector modes. For projection you can choose OpenGL projection, which renders the structured light patterns on "screen 2". You would then use an HDMI connection to the projector. Specifically for LightCrafter (LightCrafter 3500), LightCrafter4500 and newer, the software can communicate over USB with these projectors, which can preload the patterns. 

### Camera Triggering
In the preference pane, software and hardware triggering can be selected. The calibration procedure allways uses software triggering. For real-time structured light, you need hardware triggering from the projector to the camera by means of a trigger signal cable. Few projectors provide the trigger output signal, but with some effort you may be able to source one from the HDMI vsync pin. LightCrafter4500 has trigger output pins and is the recommended device. You will have to produce a trigger cable with the specific headers of your projector and camera. 
With commercial projectors you can usually project 60 patterns per second at most (LC4500 allows for 120 8-bit patterns per second). Usually the camera requires a pause between two consecutive exposures, and you will need to waste one refresh period. With LC4500 we can project 60 grayscale patterns per second resulting in up to 20 point clouds per second.

Some cameras have only a single I/O port, e.g. Ximea MQ013RG-E2. This is enough for frame triggering, but synchronization with regard to the sequence is difficult to achieve. For this problem, a triggering circuit bases on an Arduino microcontroller was developed. The source code found in ``trigger_circuit`` can be uploaded to an Arduino and takes two triggering pulses from the projector (frame start and sequence start) and generates one output signal (long pulse for sequence start and short pulses for all other frames). Camera frames are marked with the I/O status shortly after triggering which allows us to very robustly determine the sequence start.

### Calibration
Calibration of your hardware setup determines the internal camera and projector parameters, and the relative position of the projector relative to the camera. The calibration procedure only is seen in the video above, and requires a flat board with a grey-white checkerboard printed onto it. Good calibration requires a large number of calibration positions covering all of the sensor field of view and with some foreshortening (angle of the calibration board relative to view-axis). You can use e.g.  https://calib.io/pages/camera-calibration-pattern-generator to generate a PDF for the checkerboard.

### Codec
SLStudio serves as a structured light platform. A number of encoding strategies has been implemented, and these differ greatly in the number of patterns, accuracy and robustness to e.g. shiny or semitransparent surfaces. For many applications, 2x3 phase shifting provides good results. It uses 6 patterns, so you will reach 10Hz point cloud update frequency with the appropriate hardware. 

### Recommended settings
Please note that some parts of this software are still experimental, while others have been matured. For reliable point cloud capture we recommend the following choices:
* Ubuntu 20.04
* Newer dedicated nVidia graphics card
* Ximea or Point Grey camera
* Calibration in 20 positions
* Calibration board with 10 x 10 saddle points
* Software triggering
* 2x3 Phase Shifting

For real-time (10Hz +) performance we recommend the following:
* LightCrafter4500 configured for 120Hz HDMI gray-scale projection (e.g. configure using /tools/lc4500startup)
* Ximea MQ013RG-E2
* Hardware triggering

## Support
While we are interested in providing our software to a rich audience and make real-time structured light available to many user groups, it usually requires a fair amount of customization and knowledge to build these systems. With the current state of this project and the challenges in hardware, you do need C++ programming experience to make it work with your specific setup, unless it consists of the exact components listed above. We can answer some questions pertaining to software and hardware, but cannot provide low-level support. We are though very interested in incorporating relevant and meaningful improvements and bug fixes, so if you have such, please contact us or make a pull request!

## License
SLStudio did not provide an explicit license before August 2021. Any usage of the software before that date is technically a copyright violation. Since August 2021, the software is released under a dual license scheme. Users may use it under GPLv3 (see LICENSE.md), or obtain a commercial license, which allows for distribution of derivative works without disclosing any source code. Contact jw@vision-consulting.dk if you are interested in obtaining a commercial license.


