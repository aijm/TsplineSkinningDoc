# Volume(样条体)
相关文件：
- Volume.h
- Volume.cpp
  
常用的样条体有B样条体和T样条体，为了复用代码，本软件定义了一个抽象基类`Volume`

### Volume类
```cpp
#ifndef VOLUME_H
#define VOLUME_H

#include "utility.h"
using namespace std;
using namespace igl::opengl::glfw;
using namespace t_mesh;
class Volume {
public:
	Volume():id(-1), reverse(false){}
	void setViewer(Viewer* viewer) {
		this->viewer = viewer; // 在头文件中内联，若在cpp中用inline，会导致其他文件找不到定义
	}
	void draw(bool tmesh, bool polygon, bool surface, double resolution = 0.01);
	// calculate the coordinate at paramter (u,v,w)
	virtual Point3d eval(double u, double v, double w) = 0;

	// read volume from file
	virtual int readVolume(string) = 0;
	// save volume to file
	virtual int saveVolume(string) = 0;
	// 通过将参数域按resolution分割，将体离散为六面体网格
	int saveAsHex(string, double resolution = 0.01);
	void setReverse(bool _reverse) {
		reverse = _reverse;
	}
protected:
	virtual void drawTmesh() = 0;
	virtual void drawControlpolygon() = 0;
	// 在体的边界面画出参数线
	virtual void drawParamCurve() = 0;
	// 画出体的六个边界面
	virtual void drawVolume(double resolution = 0.01);

protected:
	Viewer* viewer;
	int id;
	bool reverse;   // u,v,w是否满足右手法则，不满足则需要设置为true
};

#endif // !VOLUME_H
```
`Volume`类是一个抽象基类，`BsplineVolume,TsplineVolume`会继承`Volume`类并给出相应虚函数的各自实现。

