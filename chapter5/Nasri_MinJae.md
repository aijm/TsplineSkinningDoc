# NasriMethod&MinJaeMethod
相关文件：
- NasriMethod.h
- NasriMethod.cpp
- MinJaeMethod.h
- MinJaeMethod.cpp
  
### NasriMethod
NasriMethod实现了[Nasri 2012](https://link.springer.com/article/10.1007/s00371-012-0692-1)论文中的方法。

具体实现中是通过重写`insert方法`，按论文中的方式插入一列中间线。
```cpp
#ifndef NASRIMETHOD_H
#define NASRIMETHOD_H

#include "Skinning.h"
class NasriMethod : public Skinning {
public:

	NasriMethod(const vector<NURBSCurve>& _curves)
		:Skinning(_curves) {

	}

	void init() override;
	// 插入一列中间线
	void insert() override;
};


#endif // !NASRIMETHOD_H
```

### MinJaeMethod

MinJaeMethod实现了[MinJae 2018](https://www.sciencedirect.com/science/article/abs/pii/S0010448518301969)论文中的方法。

具体实现中是通过重写`insert方法`，按论文中的方式插入两列中间线，并按照论文中的方法迭代更新插入的中间线控制点坐标，从而得到更好的形状。

```cpp
#ifndef MINJAEMETHOD_H
#define MINJAEMETHOD_H

#include "Skinning.h"
class MinJaeMethod : public Skinning {
public:
	MinJaeMethod(
		const vector<NURBSCurve>& _curves, 
		int _sampleNum = 10, int _maxIterNum = 20, double _eps = 1e-5)
		:Skinning(_curves),sampleNum(_sampleNum),
		maxIterNum(_maxIterNum),eps(_eps) {

	}

	void init() override;		// 根据NUUBSCurve初始化T-preimage
	void insert() override;		// 在T-preimage中插入中间节点
	void calculate() override;  // 计算流程

private:
	void inter_init();          // 初始化中间点的坐标
	double inter_update();        // 更新中间点的坐标

private:
	const int sampleNum;
	const int maxIterNum;
	const double eps;
	map<double, map<double, Point3d>> initial_cpts;
};
#endif // !MINJAEMETHOD_H
```
其中`calculate`计算流程重写为：
```cpp
void MinJaeMethod::calculate()
{
	// 1. compute s-knot for curves
	parameterize();
	// 2. construct basis T-mesh 
	init();
	// 3. insert intermediate vertices
	// the coordinate of vertices is the midpoint of the corresponding points in C_r and C_(r+1)
	insert();

	// 4. 初始化中间点坐标
	inter_init();
	update();
	
	// 5. 迭代更新中间点坐标
	double error = 1.0;
	for (int i = 0; i < maxIterNum; i++) {
		error = inter_update();
		update();
		cout << "*******************iter " << i+1 << ": ,error " << error << "********************" << endl;
		if (error < eps) {
			break;
		}
	}
}
```
其中每个步骤的具体实现见代码。

### 使用方法
`main.cpp`中包含了一些T样条曲面蒙皮方法的测试用例，可参照使用。
```cpp
// main.cpp
Test::test_circle_skinning();
Test::test_venus_skinning();
Test::test_venus_skinning_helper_points();
Test::test_Bsurface_skinning();
Test::test_chess_skinning();
Test::test_ring_skinning();
Test::test_helicoidal_skinning();
Test::test_bonnet_skinning();
Test::test_door_skinning();
Test::test_face_skinning();
```