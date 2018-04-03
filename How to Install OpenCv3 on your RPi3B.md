## 如何在树莓派3B上安装OpenCv3.3.0

***本文使用python3***

### 第一步：扩展文件系统
使用以下命令进入系统设置界面

	sudo raspi-config
然后进入 **Advanced Options** 菜单，选择 **Expand filesystem** 然后选择 **finish** 完成设置

在以上操作完成后，重启系统使得设置生效

	sudo reboot
在完成重启后，可使用 **df -h**来查看文件系统是否已经被扩展

PS：如果SD卡容量不足的话可以使用以下命令来删除不必要的组件，大概可以获得1GB左右的容量
	sudo apt-get purge wolfram-engine
	sudo apt-get purge libreoffice*
	sudo apt-get clean
	sudo apt-get autoremove

### 第二步：安装相关依赖
首先，需要保证你的包都是最新的，所以第一步需要更新或升级已经装好的包

	sudo apt-get update && sudo apt-get upgrade
然后我们需要安装一些开发者工具，包括cmake

	sudo apt-get install build-essential cmake pkg-config
接下来，安装图像I/O的包，确保我们可以加载各种格式的图片

	sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
除了图像I/O包，我们还需要视频的I/O包

	sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
	sudo apt-get install libxvidcore-dev libx264-dev
要想使用OpenCv下的**highgui**子模块（该模块用于显示图片和构建相应的基本gui），我们需要安装gtk开发库

	sudo apt-get install libgtk2.0-dev libgtk-3-dev
在OpenCv中的很多操作都可以被进一步优化，我们所需要做的就是安装下面这些依赖

	sudo apt-get install libatlas-base-dev gfortran
最后，不要忘记安装python的头文件，不然等会没法编译OpenCv的python接口

### 第三步：下载opencv的源码
推荐命令行直接下载，如果不行的话请自行在github上下载zip包
首先是opencv的源码

	cd ~
	wget -O opencv.zip https://github.com/Itseez/opencv/archive/3.3.0.zip
	unzip opencv.zip
接着是opencv_contrib的源码（该部分包含特征点检测等算法函数如SIFT和SURF等等）

	wget -O opencv_contrib.zip https://github.com/Itseez/opencv_contrib/archive/3.3.0.zip
	unzip opencv_contrib.zip
	
### 第四步：相关环境配置
在我们编译opencv之前，我们需要安装一个python的包管理工具 **pip**

	wget https://bootstrap.pypa.io/get-pip.py
	sudo python get-pip.py
	sudo python3 get-pip.py

本人由于同时使用多个版本的python，所以安装了virtualenv来保证没有冲突，建议读者也安装virtualenv

	sudo pip install virtualenv virtualenvwrapper
	sudo rm -rf ~/.cache/pip
接下来，我们需要配置profile来让虚拟环境生效

	sudo nano ~/.profile
然后在profile尾部加入以下内容

	# virtualenv and virtualenvwrapper
	export WORKON_HOME=$HOME/.virtualenvs
	source /usr/local/bin/virtualenvwrapper.sh
如果不想手动更改profile内容的话可以使用以下命令

	echo -e "\n# virtualenv and virtualenvwrapper" >> ~/.profile
	echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.profile
	echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.profile
接下来，重新加载profile使更改生效

	source ~/.profile
然后为python3创建一个虚拟环境

	mkvirtualenv cv -p python3
以后只需使用以下命令，即可进入刚才创建的虚拟环境

	source ~/.profile
	workon cv
当你进入虚拟环境之后，每一行命令提示符的前面都会多出一个 **（cv）**
接下来我们还需要安装numpy，一个数值处理的包

	pip install numpy

### 第五步：编译并安装opencv
首先我们按上面提到的，进入我们的虚拟空间

	source ~/.profile
	workon cv
然后我们可以开始设置cmake的编译条件

	cd ~/opencv-3.3.0/
	mkdir build
	cd build
	cmake -D CMAKE_BUILD_TYPE=RELEASE \
	    -D CMAKE_INSTALL_PREFIX=/usr/local \
	    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.3.0/modules \
	    -D BUILD_EXAMPLES=ON ..

如果成功安装，你将会在以 **Python3:** 开头的段落里看到python的相关地址

********************************************************
**接下来这步较为关键，为了确保编译成功，我们需要给足够的虚拟内存**
以sudo权限打开你的 **/etc/dphys-swapfile** 然后修改其中的 **CONF_SWAPSIZE** 变量，将其修改为1024，这样，我们就有足够的内存去编译了

为了使上述设置生效，我们需要重启*swap service*：

	sudo /etc/init.d/dphys-swapfile stop
	sudo /etc/init.d/dphys-swapfile start
这样就完成了这部分的设置

********************************************************
 
然后，使用下列命令并开始写作业（滑稽
 
	make -j4
这玩意相当费时间，等他make的时间里，我们可以做完好多作业（逃

然后安装

	sudo make install
	sudo ldconfig

### 第六步：完成安装
在使用了 **make install** 之后，你的opencv应当被安装在以下地址

	/usr/local/lib/python3.5/site-packages
你可以使用 **ls -l** 指令来确认这一点

接下来，我们需要给这文件改个名并且链接到我们的环境


	cd /usr/local/lib/python3.5/site-packages/
	sudo mv cv2.cpython-35m-arm-linux-gnueabihf.so cv2.so
	cd ~/.virtualenvs/cv/lib/python3.5/site-packages/
	ln -s /usr/local/lib/python3.5/site-packages/cv2.so cv2.so

好！冗长的安装总算结束了！

### 第七步：测试安装
还记得上面进虚拟环境的方法吗

	source ~/.profile 
	workon cv
然后开始测试

	$ python
	>>> import cv2
	>>> cv2.__version__
	'3.3.0'
	>>>
看到这个版本号你就可以安心了，弄了这么长时间总算结束了

**但是，我们还有收尾工作要做**

不要忘记把*swap size*改回来！

按照上面的方法打开并将 **CONF_SWAPSIZE** 变量改回100并保存

然后重启服务

	sudo /etc/init.d/dphys-swapfile stop
	sudo /etc/init.d/dphys-swapfile start

大功告成，完结撒花（假装有花
