# Linux下的Geant4安装

## 1 WSL2和Ubuntu-22.04
- 控制面板 -> 程序和功能 -> 启用或关闭Windows功能，勾选“适用于Linux的Windows子系统”以及“虚拟机平台”，重启

- 管理员身份运行powershell
```bash
wsl --set-default-version 2

wsl --update

wsl --shutdown
```

- Microsoft store搜索下载“Ubuntu 22.04.x LTS”，设置用户名和密码；

-设置root
```bash
sudo passwd root
// 设置密码
// 后续只用su命令，输入密码进入root

wsl --manage Ubuntu-22.04 --set-default-user root 
//powershell管理员打开，默认root登录，方便vscode直接修改文件
//这样修改之后可能代码无法跳转，先把原用户文件夹下的.vscode-server删掉，然后重新连接，再到插件页面安装C++ package

```


- VScode下载WSL插件，`ctrl shift P`选择“连接WSL”

## 2 Geant4
### Preview
- gcc安装
```bash

sudo apt update
sudo apt upgrade
sudo apt-get install build-essential -y
```

- openssl安装
```
sudo apt-get install libssl-dev -y
```

- cmake\ccmake安装
```bash
apt  install -y cmake  # version 3.22.1-1ubuntu1.22.04.2
apt  install -y cmake-curses-gui  # version 3.22.1-1ubuntu1.22.04.2
```
- 在geant4文件中安装clhep
```bash
cd /home
mkdir geant4
cd geant4

wget https://proj-clhep.web.cern.ch/proj-clhep/dist1/clhep-2.4.7.1.tgz
tar -xvf ./clhep-2.4.7.1.tgz
cd 2.4.7.1
mkdir build
cd build
cmake ../CLHEP 
make -j24
sudo make install
```
- 安装Expat与Zlib、gedit
```bash
sudo apt-get install -y libexpat-dev
sudo apt install -y zlib1g-dev
sudo apt-get install -y gedit
apt install unzip
```

- 在geant4文件夹中安装xerce
```bash

cd /home/geant4
wget https://dlcdn.apache.org//xerces/c/3/sources/xerces-c-3.3.0.tar.gz
tar -xvf ./xerces-c-3.3.0.tar.gz
cd xerces-c-3.3.0
mkdir build
cd build
cmake ../
make -j24
make test
sudo make install

```



- 安装5.15

    ```bash
    cd /home/fanghaodu
    wget https://download.qt.io/archive/qt/5.15/5.15.2/single/qt-everywhere-src-5.15.2.tar.xz 
    
    tar -xvf qt-everywhere-src-5.15.2.tar.xz
    
    cd qt-everywhere-src-5.15.2 
    
    //修改.qt-everywhere-src-5.15.2/qtbase/src/corelib/global/qglobal.h, 增加 "#include <limits>"在"#include <alogrithm>"后面
    
    sudo apt-get install -y qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
    //Ubuntu21.04开始qt5-default包定位不到，需要安装上面几个包替代。 
    
    
    //安装一些包
    sudo apt-get install -y libglu1-mesa-dev freeglut3-dev mesa-common-dev
    sudo apt-get install -y libx11-dev libxmu-dev 
    sudo apt-get install -y libmotif-dev
    sudo apt-get install -y libcanberra-gtk-module
    sudo apt-get install -y \
      libxcb1 \
      libxcb-util1 \
      libx11-xcb1 \
      libxrender1 \
      libxi6 \
      libxext6 \
      libxcb-keysyms1 \
      libxcb-image0 \
      libxcb-shm0 \
      libxcb-icccm4 \
      libxcb-sync1 \
      libxcb-xfixes0 \
      libxcb-shape0 \
      libxcb-randr0 \
      libxcb-render-util0 \
      libxcb-glx0 \
      libxcb-xinerama0
      libxfixes-dev libxcb-xfixes0-dev
      
    sudo apt install libx11-dev libxcb1-dev libx11-xcb-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-xinerama0-dev libxkbcommon-dev libxkbcommon-x11-dev
    
    
    sudo apt-get install apt-file autoconf automake automake1.11 tcl8.6-dev tk8.6-dev libglew-dev libglw1-mesa-dev gfortran inventor-dev libxaw7-dev libxerces-c-dev libxmltok1-dev libxi-dev libclutter-gtk-1.0-0 libxmlrpc-core-c3-dev tclxml tclxml-dev libgtk2.0-dev libxpm-dev x11proto-gl-dev x11proto-input-dev -y
    
    sudo apt-get install libxcb1-dev libx11-xcb-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev -y
    sudo apt-get install build-essential libgl1-mesa-dev -y
    sudo apt-get install '^libxcb.*-dev' libx11-xcb-dev libglu1-mesa-dev libxrender-dev libxi-dev -y
    sudo apt-get install flex bison gperf libicu-dev libxslt-dev ruby -y
    sudo apt-get install libx11-* -y
sudo apt-get install libxcb-* -y
    sudo apt-get install libxcb* -y

    ./configure -skip qtvirtualkeyboard -skip qtlocation -xcb// 跳过这两个包的编译，一个是地图的，一个是虚拟键盘的，这个安装包可能缺这两包的源代码，干脆跳过, -xcb很关键，涉及到libqxcb.so这个库
    
    //默认安装在/usr/local/Qt-5.15.2, -prefix 显式指定路径
    
    //等待一会，第一个选项选o，第二个选项选y,如果中间出错了，需要清除缓存之后再重新安装
    sudo rm -rf config.cache
    ./configure 
```
    
    - 开始编译
    
    ```bash
    // 有点久，半小时起步
    make -j24 
    sudo make install
    
    
    ```

    - qtchooser路径配置
    ```bash
    cd /usr/lib/x86_64-linux-gnu/qt-default/qtchooser
    sudo gedit default.conf
    
    //删去默认的，找到安装路径下的，编辑如下：
    /usr/local/Qt-5.15.2/bin //qmake所在路径
    /usr/local/Qt-5.15.2/lib  //qt的cmake所在路径
```
    - qmake设置
    
    ```bash
    
    sudo gedit /etc/profile
    // 可视化界面打开后写入以下指令
    export QTDIR=/usr/local/Qt-5.15.2  //改为安装路径的bin的前一级
    export PATH=$QTDIR/bin:$PATH
    export LD_LIBRARY_PATH=$QTDIR/lib:$LD_LIBRARY_PATH
    ```
    
    这时候检验一下qt版本对了不：
    ```bash
    qmake -v //5.15.x就对了
    ```

![image-20250520135252576](C:\code\knowhow-transfer\md_pics\image-20250520135252576.png)

### G4 Install
- 为cmake指定寻找qt库的路径（暂时的把这个全局变量改成qt的cmake路径）:
```bash
# export CMAKE_PREFIX_PATH=home/geant4/Qt5/5.14.2/gcc_64/lib/cmake 这是5.14的安装,下面是5.15的安装
export CMAKE_PREFIX_PATH=/usr/local/Qt-5.15.2/lib/cmake

```

- G4源码下载解压build

```bash
cd /home/geant4
wget https://gitlab.cern.ch/geant4/geant4/-/archive/v11.3.1/geant4-v11.3.1.tar.gz
tar -xvf geant4-v11.3.1.tar.gz



mkdir geant4-v11.3.1-install
mkdir geant4-v11.3.1-build
cd geant4-v11.3.1-build //这一步一定要执行

ccmake ../geant4-v11.3.1 //cmake不允许在源代码目录下搞。出现下述界面，按c进行配置，然后按e退出，出现配置项目，按图配置，再按c配置，然后退出


//下面的命令,包含了参数配置，qt5.15.2手动安装后，要改环境变量
sudo cmake -DQT_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5 -DQt5Core_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5Core -DQt5Gui_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5Gui -DQt5OpenGL_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5OpenGL -DQt5Widgets_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5Widgets -DQt5_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt5 -DQt53DCore_DIR=/usr/local/Qt-5.15.2/lib/cmake/Qt53DCore   /home/geant4/geant4-v11.3.1


// DGEANT4_INSTALL_DATA=ON会自动下载data文件夹至home/geant4/geant4-v11.3.1-build文件夹中,速度太慢容易出错，建议OFF，make install 完成之后手动下载


sudo hwclock -s //可能出现警告make[1]: warning:  Clock skew detected.  Your build may be incomplete. 是wsl时间戳的问题，同步一下和windows的事件
sudo make -j24 
sudo make install
```




![image-20250519195250328](C:\code\knowhow-transfer\md_pics\image-20250519195250328.png)

![image-20250521215608018](C:\code\knowhow-transfer\md_pics\image-20250521215608018.png)



- 添加数据包的路径
```bash
gedit ~/.bashrc
// 末尾加以下
source  /home/geant4/geant4-v11.3.1-install/bin/geant4.sh
#source  /home/geant4/geant4-v11.3.1/geant4-v11.3.1-install/share/Geant4/geant4make/geant4make.sh
```


根据版本信息在官网下载至`/home/geant4/geant4-v11.3.1-install/share/Geant4/data`目录，并解压
```bash
cd /home/geant4/geant4-v11.3.1-install/share/Geant4
mkdir data
cd data
wget https://cern.ch/geant4-data/datasets/G4NDL.4.7.1.tar.gz
wget https://cern.ch/geant4-data/datasets/G4EMLOW.8.6.1.tar.gz
wget https://cern.ch/geant4-data/datasets/G4PhotonEvaporation.6.1.tar.gz
wget https://cern.ch/geant4-data/datasets/G4RadioactiveDecay.6.1.2.tar.gz
wget https://cern.ch/geant4-data/datasets/G4PARTICLEXS.4.1.tar.gz
wget https://cern.ch/geant4-data/datasets/G4PII.1.3.tar.gz
wget https://cern.ch/geant4-data/datasets/G4RealSurface.2.2.tar.gz
wget https://cern.ch/geant4-data/datasets/G4SAIDDATA.2.0.tar.gz
wget https://cern.ch/geant4-data/datasets/G4ABLA.3.3.tar.gz
wget https://cern.ch/geant4-data/datasets/G4INCL.1.2.tar.gz
wget https://cern.ch/geant4-data/datasets/G4ENSDFSTATE.3.0.tar.gz
wget https://cern.ch/geant4-data/datasets/G4CHANNELING.1.0.tar.gz
tar -xvf G4NDL.4.7.1.tar.gz
tar -xvf G4EMLOW.8.6.1.tar.gz
tar -xvf G4PhotonEvaporation.6.1.tar.gz
tar -xvf G4RadioactiveDecay.6.1.2.tar.gz
tar -xvf G4PARTICLEXS.4.1.tar.gz
tar -xvf G4PII.1.3.tar.gz
tar -xvf G4RealSurface.2.2.tar.gz
tar -xvf G4SAIDDATA.2.0.tar.gz
tar -xvf G4ABLA.3.3.tar.gz
tar -xvf G4INCL.1.2.tar.gz
tar -xvf G4ENSDFSTATE.3.0.tar.gz
tar -xvf G4CHANNELING.1.0.tar.gz
```

### 运行测试

```bash
cd /home/geant4/geant4-v11.3.1/examples/basic/B1
mkdir build
cd build/
source ~/.bashrc
cmake ../

//可能会出现Qt5::Core路径不对，ccmake改一下，统一改为/usr/local下的camke路径

make -j24
```
![image-20250519220109765](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250519220109765.png)



编译成功后在build找可执行文件`exampleB1`,执行

```bash
./exampleB1
```
其他example也测试下



出现以下gui说明安装成功：

![alt text](../md_pics/G4_success.png)



# Linux下的Gate安装

- ## ITK安装

```
cd /home
mkdir gate
cd gate
mkdir itk
cd itk
wget https://github.com/InsightSoftwareConsortium/ITK/releases/download/v5.4.2/InsightToolkit-5.4.2.tar.gz

tar -zxvf InsightToolkit-5.4.2.tar.gz
cd InsightToolkit-5.4.2
mkdir bin
cd bin

ccmake .. //搞成下图的样子，注意Module_ITKReview=ON要打开taggle才看得到
cmake ..

//gate/itk/InsightToolkit-5.1.1/Modules/ThirdParty/GDCM/src/gdcm/Source/MediaStorageAndFileFormat/gdcmImageChangePhotometricInterpretation.h,添加头文件<limits>

make -j24
make install //时间稍长

```



![image-20250521125122094](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250521125122094.png)



# 安装root

```
cd /home/gate
mkdir ROOT
cd ROOT

wget https://root.cern/download/root_v6.32.02.source.tar.gz
tar -zxvf root_v6.32.02.source.tar.gz

mkdir root-6.32.02-build
mkdir root-6.32.02-install
cd root-6.32.02-build
cmake -DCMAKE_INSTALL_PREFIX=/home/gate/root/root-6.32.02-install  -DGEANT4_USE_OPENGL_X11=ON -DGEANT4_BUILD_MULTITHREADED=ON /home/gate/root/root-6.32.02

make -j10  //搞大了内存不够统一杀c++进程
sudo make install

gedit ~/.bashrc
source /home/gate/root/root-6.32.02-install/bin/thisroot.sh

cd /home/gate/root/root-6.32.02-install/bin
./root

//出现如下图说明成功
```

![image-20250521214047355](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250521214047355.png)





# Gate 9.4.1 安装(对应G4 11.2.x)

```
cd /home/gate
wget https://github.com/OpenGATE/Gate/archive/refs/tags/v9.4.1.tar.gz
tar -zxvf v9.4.1.tar.gz

mkdir gate-9.4.1-build
mkdir gate-9.4.1-install
cd gate-9.4.1-build

sudo apt-get install libxml2-dev

ccmake ../Gate-9.4.1


```



**![image-20250522103301764](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250522103301764.png)**





```
gedit ~/.bashrc
export PATH=/home/gate/gate-9.4.1-install/bin:$PATH

//test
source ~/.bashrc
cd /home/gate/gate-9.4.1-install/bin
./Gate
```

![image-20250522103715619](C:\code\knowhow-transfer\md_pics\image-20250522103715619.png)



复制benchmark文件夹：去官网下在9.0的，找到里面的benchmark文件夹，复制到Gate-9.4.1下面

到benchPET.mac文件把首行二行的注释换一下

![image-20250522135101642](C:\code\knowhow-transfer\md_pics\image-20250522135101642.png)



digitizer语法变更：

```
apt-get install pip



cd /home/gate/Gate-9.4.1
git clone --recursive https://github.com/OpenGATE/GateTools.git
cd GateTools
pip install .
pip install gatetools

gedit ~/.bashrc
export PATH=/home/gate/Gate-9.4.1/GateTools/gatetools/bin
```





变更命令：

```sh
cd /home/gate/Gate-9.4.1/benchmarks/benchPET
gt_digi_mac_converter.py -i digitizer.mac -o digitizer_new.mac -sd LSO -multi SinglesDigitizer

```

![image-20250522140216934](C:\code\knowhow-transfer\md_pics\image-20250522140216934.png)

