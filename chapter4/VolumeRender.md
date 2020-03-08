# VolumeRender窗口显示样条体
相关文件：
- VolumeRender.h
- VolumeRender.cpp

与`MeshRender`类似，
为了方便显示T样条体，而且轻松切换T样条体的不同显示模式，本软件添加了`VolumeRender`模块，可以通过键盘切换不同显示模式并且易于使用。

### VolumeRender设计
```cpp
#ifndef VOLUMERENDER_H
#define VOLUMERENDER_H

#include "Window.h"
#include "Volume.h"
class VolumeRender : public Window {

public:
	VolumeRender(Volume* _volume,
		bool _showmesh = false, bool _showpolygon = true,
		bool _showvolume = true, double _resolution = 0.1)
		:volume(_volume), showmesh(_showmesh), showpolygon(_showpolygon),
		showvolume(_showvolume), resolution(_resolution)
	{
		volume->setViewer(&viewer);
	}

protected:
	void draw() override;

private:
	Volume* volume;
	bool showmesh;
	bool showpolygon;
	bool showvolume;
	double resolution;
};

#endif // !VOLUMERENDER_H


```
由于显示T样条体是基于`Window`模块的，因此`VolumeRender`继承了`Window`并重写了`draw`函数，在其中加入了T样条体的显示及键盘控制，`draw`函数的代码如下：
```cpp
#include "VolumeRender.h"

void VolumeRender::draw()
{
	volume->draw(showmesh, showpolygon, showvolume, resolution);
	viewer.callback_key_down = [&](igl::opengl::glfw::Viewer& viewer, unsigned char key, int modifier) ->bool
	{
		if (key == 32) {
			
		}
		else if (key == 'M') {
			viewer.data().clear();
			showmesh = !showmesh;
			volume->draw(showmesh, showpolygon, showvolume, resolution);
		}
		else if (key == 'P') {
			viewer.data().clear();
			showpolygon = !showpolygon;
			volume->draw(showmesh, showpolygon, showvolume, resolution);
		}
		else if (key == 'S') {
			viewer.data().clear();
			showvolume = !showvolume;
			volume->draw(showmesh, showpolygon, showvolume, resolution);
		}
		return false;
	};
}

```
如代码所示，通过`M,P,S`三个按键，可以很方便的切换样条体的显示模式。

### 使用方法
有了`VolumeRender`模块，代码可非常简洁明了。
以`Test.cpp`中的显示B样条体为例，代码如下：
```cpp
void Test::test_BsplineVolume(string modelname, double ratio, bool reverse)
{
	BsplineVolume volume;
	volume.readVolume("../out/volume/" + modelname + "_bspline.txt");
	volume.setReverse(reverse);
	
	// 使用VolumeRender模块显示B样条体
	VolumeRender render(&volume, false, false, true, ratio);
	render.launch();
}
```

