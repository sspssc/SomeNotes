### OSRA在windows下编译
#### 预处理
osra是有源码的化学公式识别器，在linux下编译比较多，这可能是全网唯一一份windows下编译成功的说明文档。官网给的说明文档比较久远，坑也比较多。<br>
我选用msys2-32位编译，因为pacman包管理器比较方便，而且模拟了linux环境，跨平台编译比较方便。<br>

需要说明的是除了gocr是官网上找的修改过的，因为依赖版本问题我也修改过：
```
GraphicsMagic（png.c缺少了两个头文件的问题，跨平台的define的问题）、

osra/configure （找链接库的问题, -lopenbabel缺失-lz; -lgraphicsmagic缺少了它的依赖-lXXX 有9个）等文件。

解析输入路径时因为tclap版本问题cmdline解析中文会多加双引号，故在osra中后处理，去掉多的双引号
```
#### 开始编译
- 编译环境：Windows10-x64 + MSYS2-32 (含gcc) + cmake <br>
- 编译目标: OSRA-1.4 (Optical Structure Recognition Application) ———— 化学公式识别器，官网: https://cactus.nci.nih.gov/osra/ <br>
- 编译主要依赖: TCLAP 、 gocr-0.5-patched 、 ocrad-0.21 、 openbabel-2.4.1 、GraphicsMagick-1.3.12-patched 、 potrace <br>
- 主要依赖的方法是：pacman安装相关依赖、cmake/configure 生成相关依赖的makefile，编译源码
步骤如下:

```
1. tclap 复制.h文件

2. gocr 编译到msys32/usr/目录下
	 ./configure --prefix=/usr
	 make libs
	 make install
   
3. ocrad 编译到msys32/usr/目录下
	 ./configure --prefix=/usr
	 make 
	 make install

4. potrace 用pacman安装32位到mingw下
	 pacman -S mingw-w64-i686-potrace //安装到potrace到mingw32下
   
5. openbabel 编译的2.4.1版本（2.3会编译失败），依赖较多，用pacman安装 eigen, libxml2, zlib
	pacman -S mingw-w64-i686-eigen3  //安装eigen3到mingw32下	
	
  pacman -S mingw-w64-i686-cairo  //安装cairo到mingw32下
	
  必须外部安装cmake（pacman安装的cmake不管用），然后绑定cmake的程序位置到环境变量 PATH=$PATH:/e/Program\ Files\ \(x86\)/CMake/bin/
	
  pacman -S libxml2-devel    //安装libxml2包含了zlib
	
	mkdir build
	cd build   //创建build并切到该目录下
	cmake.exe -G"MSYS Makefiles" -DZLIB_LIBRARY=/usr/lib/libz.a -DZLIB_INCLUDE_DIR=/usr/inclulde -DLIBXML2_LIBRARIES=/usr/lib/libxml2.a -DLIBXML2_INCLUDE_DIR=/usr/include/ -DCAIRO_INCLUDE_DIR=/mingw32/include/ -DCAIRO_LIBRARY=/mingw32/lib/libcairo.a -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_SHARED=OFF -DCMAKE_CXX_FLAGS=-DLIBXML_STATIC -DCMAKE_C_FLAGS=-DLIBXML_STATIC -DCMAKE_CXX_STANDARD_LIBRARIES=-lws2_32 ../
	make 
	make install
	
	openbabel的测试代码(babeltest.cpp来自于osra的configure文件): 
		g++ -o babeltest.exe -g -O2 -I/usr/include/openbabel-2.0 -I/usr/include/gocr -I/usr/include -I/mingw32/include -I/usr/include/GraphicsMagick -L/usr/lib -L/mingw32/lib babeltest.cpp -lopenbabel -lz  -lws2_32 -locrad -lpotrace -lm

6. graphicmagic-1.3.12包的编译，必须使用自己修改过的版本（修改内容为添加了pnginfo.h和pngstruct.h头文件，修改了png.c文件）
	pacman -S mingw-w64-i686-jbigkit  //jbigkit用pacman装2.1.4
  pacman -U mingw-w64-i686-lcms-1.19-5-any.pkg.tar.xz  //lcms用pacman装1.19.5，(网上下载)
  libz     //安装在msys下的1.2.11-1
  bzip2    //安装在pacman下的1.0.8-2
  libpng   //安装在mingw32下1.6.19-1	
  freetype    //一般msys都有，没有则用pacman安装
  
  jpeg-6b        //从外部复制
	jasper-1.701.0 //必须指定版本复制
	tiff3.8.2   //必须指定版本复制
	
  pacman -S mingw-w64-i686-ghostscript //用pacman安装
	
	安装命令:
	./configure --disable-shared --with-x=no --disable-openmp --without-threads --without-wmf --without-xml --without-bzlib --without-trio --prefix=/usr/ CPPFLAGS="-I/mingw32/include" LDFLAGS="-L/mingw32/lib " LIBS="-lz -lbz2 -lpng -ljasper -ljbig -ljpeg -llcms -ltiff -lfreetype"
  make
  make install
  
	graphicMagic结果的测试代码(graphictest.cpp来自于osra的configure文件)：
		g++ -o graphictest.exe -g -O2 -I/usr/include/GraphicsMagick -I/mingw32/include -D_LIB -I/usr/include/openbabel-2.0 -I/usr/include/gocr graphictest.cpp -L/mingw32/lib -lGraphicsMagick -lGraphicsMagick++ -lpng -ltiff -lbz2 -ljpeg -ljasper -ljbig -lfreetype -llcms -lgdi32 -lopenbabel -lz -lws2_32 -locrad -lpotrace -lm
		
7. 安装osra-1.4: 		
	./configure --prefix=/osra-1.4 CPPFLAGS="-I/mingw32/include -I/usr/include/GraphicsMagick" LDFLAGS="-L/mingw32/lib" 
	make all install   //非静态编译，故需要手动复制相关dll
 ```
	

