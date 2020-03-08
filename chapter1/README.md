# 安装
本项目基于cmake构建，cmake的结构来自于[libigl/libigl-example-project](https://github.com/libigl/libigl-example-project), 这是一个空白的项目示例，展示了如何使用libigl和cmake。 

## libigl tutorial
本项目的显示模块基于libigl，因此需要先了解libigl的基本内容和如何安装运行。[libigl tutorial](http://libigl.github.io/libigl/tutorial/).

### libigl下载
libigl的Github项目地址为[libigl](https://github.com/libigl/libigl)

可以使用`git clone`下载libigl源代码
```shell
git clone https://github.com/libigl/libigl
```
也可以在网页上点击`Clone or download`下载`zip`源代码压缩包。

### build libigl
使用标准cmake流程构建libigl
```
cd libigl/
mkdir build
cd build
cmake ../
make
```
在Windows中，可以使用cmake-gui，指定平台为Visual Studio 2015，从而生成VS2015的解决方案，用VS2015打开即可编译运行。


## Compile
在下载和安装libigl之后, 你需要将libigl的路径添加到本项目的`cmake/FindLIBIGL.cmake`文件中。
```
find_path(LIBIGL_INCLUDE_DIR igl/readOBJ.h
    HINTS
        ENV LIBIGL
        ENV LIBIGLROOT
        ENV LIBIGL_ROOT
        ENV LIBIGL_DIR
    PATHS
        ${CMAKE_SOURCE_DIR}/../..
        ${CMAKE_SOURCE_DIR}/..
        ${CMAKE_SOURCE_DIR}
        ${CMAKE_SOURCE_DIR}/libigl
        ${CMAKE_SOURCE_DIR}/../libigl
        ${CMAKE_SOURCE_DIR}/../../libigl
        D:/Program\ Files/libigl # 你可以在这里添加libigl的路径
        /usr
        /usr/local
        /usr/local/igl/libigl
    PATH_SUFFIXES include
)

```
使用标准cmake流程编译此项目
```
mkdir build
cd build
cmake ..
```
在Windows中，可以使用cmake-gui，指定平台为Visual Studio 2015，从而生成VS2015的解决方案。

这时在build目录下会生成`Tspline.sln`。

用VS2015打开，即可编译运行程序。



