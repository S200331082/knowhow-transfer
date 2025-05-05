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
```

- VScode下载WSL插件，`ctrl shift P`选择“连接WSL”

## 2 Geant4
### Preview
- gcc安装
```bash
sudo apt-get install build-essential
```

- openssl安装
```
sudo apt-get install libssl-dev
```

- cmake安装
```bash
wget http://www.cmake.org/files/v3.28/cmake-3.28.3.tar.gz //打开该网址找到该压缩文件并下载
tar -xvf cmake-3.28.3.tar.gz //对该压缩文件解压
cd cmake-3.28.3
./configure
make -j24
sudo apt-get install checkinstall 
sudo checkinstall //无脑回车即可
sudo make install
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
- 安装Expat与Zlib
```bash
sudo apt-get install -y libexpat-dev
sudo apt install -y zlib1g-dev
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

- 在geant4文件夹中安装qt

    - 先到qt官网申请个账号

    - 先下载[`qt-opensource-linux-x64-5.14.2.run`]( https://pan.baidu.com/s/1VyiY70Nj_MdrrdLvniXqcw)，密码g2n5

    - win和linux文件复制,假设你要将 `C:\Users\YourUsername\Documents\file.txt` 移动到 WSL 中的 `/home/yourusername/` 目录下：
        ```bash
        cp /mnt/c/Users/YourUsername/Documents/file.txt /home/yourusername/
        ```
    - 运行安装.run文件
        ```bash
        chmod +x qt-opensource-linux-x64-5.14.2.run
        ./qt-opensource-linux-x64-5.14.2.run
        ```
    - 上述运行可能会出现没有相关插件的错误，确保已经安装相关库以及设置环境变量
        ```bash
            sudo apt-get install libxcb-xinerama0
            export DISPLAY=:0
        ```
    - 可视化界面下安装，登录账号，勾选协议，一路next，安装路径选择`/home/geant4/Qt5`(新建)
    - 安装包勾选Qt目录下的`5.14.2`的所有，开始install

- 继续安装qt相关的包并进行配置
```bash
sudo apt-get install qt5-qmake
sudo apt-get install qtbase5-dev

sudo apt-get install -y vim
sudo apt-get install -y gedit
cd /usr/lib/x86_64-linux-gnu/qt-default/qtchooser
sudo gedit default.conf
```

- gedit可视化界面显示后，末尾写入以下指令并保存
```bash
/home/geant4/Qt5/5.14.2/gcc_64/bin
/home/geant4/Qt5/5.14.2/gcc_64
```

- qmake设置
```bash
sudo gedit /etc/profile
// 可视化界面打开后写入以下指令
export QTDIR=/home/geant4/Qt5/5.14.2/gcc_64
export PATH=$QTDIR/bin:$PATH
export LD_LIBRARY_PATH=$QTDIR/lib:$LD_LIBRARY_PATH
```

- 上述qt5.14.2安装后对于B1的编译没有影响，但是其他examples链接库用的qt版本是5.15，需要安装5.15
    
    - 安装包此处下载

    ```bash
    tar -xvf qt-everywhere-src-5.15.2.tar.xz

    cd qt-everywhere-src-5.15.2 
    
    //修改.qt-everywhere-src-5.15.2/qtbase/src/corelib/global/qglobal.h, 增加 "#include <limits>"在"#include <alogrithm>"后面

    ./configure

    //等待一会，第一个选项选o，第二个选项选y,如果中间出错了，需要清除缓存之后再重新安装
    sudo rm -rf config.cache
    ./configure 
    ```

    - 开始编译
    
    ```bash
    make -j24 // 半小时
    sudo make install
    ```

    - qtchooser路径配置
    这里的路径配置可能和上面的5.14的产生交叉了，如果没有安装上述5.14的话，qtchooser的.conf文件可能不在/usr/lib/x86_64-linux-gnu/qt-default/qtchooser路径下，我是执行了下面的命令后，发现在原来的default.conf下出现了一个新的conf
    ```bash
    qtchooser -install qt-5.15.2 /usr/local/Qt-5.15.2/bin/qmake
    export QT_SELECT=qt-5.15.2
    ```

    ![alt text](../md_pics/conf.png)

    这时候要在这个新的conf文件下，去做原来在default.conf下的操作，只是路径要变成下面这两个，因为手动编译安装的qt路径不一样：
    ```bash
    /home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/bin
    /home/fanghaodu/qt-everywhere-src-5.15.2/qtbase
    ```

    然后/etc/profile里的QT_DIR要变：

    ```bash
    export QTDIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase  //改为安装路径的bin的前一级
    ```

    这时候检验一下qt版本对了不：
    ```bash
    qmake -v //5.15.2就对了
    ```








- 继续运行以下
```bash
sudo apt-get install -y dpkg
sudo apt-get install -y libgl1-mesa-dev 
sudo apt-get install -y libglu1-mesa-dev 
sudo apt-get install -y libx11-dev libxmu-dev 
sudo apt-get install -y libmotif-dev
sudo apt-get install -y freeglut3 freeglut3-dev binutils-gold
sudo apt-get install -y libcanberra-gtk-module
```
### G4 Install
- 为cmake指定寻找qt库的路径:
```bash
export CMAKE_PREFIX_PATH=home/geant4/Qt5/5.14.2/gcc_64/lib/cmake
sudo apt-get install build-essential apt-file gcc g++ autoconf automake automake1.11 tcl8.6-dev tk8.6-dev libglu1-mesa-dev libgl1-mesa-dev libxt-dev libxmu-dev libglew-dev libglw1-mesa-dev gfortran inventor-dev libxaw7-dev freeglut3-dev libxerces-c-dev libxmltok1-dev libxi-dev libclutter-gtk-1.0-0 cmake libxmlrpc-core-c3-dev tclxml tclxml-dev libexpat1-dev libgtk2.0-dev libxpm-dev x11proto-gl-dev x11proto-input-dev -y
 
sudo apt-get install qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools 
//Ubuntu21.04开始qt5-default包定位不到，需要安装上面几个包替代。 
```

- G4源码下载解压build

```bash
cd /home/geant4
wget https://gitlab.cern.ch/geant4/geant4/-/archive/v11.3.1/geant4-v11.3.1.tar.gz
tar -xvf geant4-v11.3.1.tar.gz
mkdir geant4-v11.3.1-install
mkdir geant4-v11.3.1-build
cd geant4-v11.3.1-build //这一步一定要执行

//下面的命令有点长，包含了参数配置，qt5.15.2手动安装后，要改环境变量
sudo cmake -DCMAKE_INSTALL_PREFIX=/home/geant4/geant4-v11.3.1-install -DGEANT4_USE_OPENGL_X11=ON -DGEANT4_USE_RAYTRACE_X11=ON -DGEANT4_USE_GDML=ON -DGEANT4_INSTALL_DATA=OFF -DGEANT4_USE_QT=ON  -DQT_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5 -DQt5Core_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5Core -DQt5Gui_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5Gui -DQt5OpenGL_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5OpenGL -DQt5Widgets_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5Widgets -DQt5_DIR=/home/fanghaodu/qt-everywhere-src-5.15.2/qtbase/lib/cmake/Qt5 /home/geant4/geant4-v11.3.1
// DGEANT4_INSTALL_DATA=ON会自动下载data文件夹至home/geant4/geant4-v11.3.1-build文件夹中
// 速度太慢容易出错，建议OFF，make install 完成之后手动下载
sudo make -j24 //线程多死命薅，可能出现警告make[1]: warning:  Clock skew detected.  Your build may be incomplete. 是wsl时间戳的问题，同步一下和windows的事件
sudo hwclock -s //再make clean 重新make检验下警告还在不
sudo make install
```
- 添加路径
```bash
gedit ~/.bashrc
// 末尾加以下两句
source  /home/geant4/geant4-v11.3.1-install/bin/geant4.sh
source  /home/geant4/geant4-v11.3.1-install/share/Geant4/geant4make/geant4make.sh
```
- 手动下载data包
查看geant4make.sh中的Datasets配置：

![alt text](../md_pics/G4Datasets.png)

根据版本信息在官网下载至`/home/geant4/geant4-v11.3.1-install/share/Geant4/data`目录
```bash
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
```
并解压
```bash
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
cd home/geant4/geant4-v11.3.1/examples/basic/B1
mkdir build
cd build/
source ~/.bashrc
cmake ../
make -j24
```
编译成功后在build找可执行文件`exampleB1`,执行
```bash
./exampleB1
```
出现以下gui说明安装成功：

![alt text](../md_pics/G4_success.png)





