# Skinning抽象基类
相关文件：
- skinning.h
- skinning.cpp

T样条曲面蒙皮方法主要有以下几个步骤：
1. 对给定的曲线进行参数化，为每一条曲线赋予一个参数值
2. 根据给定的NURBS曲线，初始化T-preimage，即将NURBS曲线的节点组成T-preimage
3. 插入中间线，如Nasri方法中插入一列中间线，MinJae方法中插入两列中间线
4. 根据插入的中间线的点的坐标，通过Nasri论文中的公式更新T样条曲面控制顶点的坐标，从而满足插值性
   
### Skinning类
为此，本软件使用了面向对象中的多态设计思想，设计了一个Skinning抽象基类，用于抽象上述的计算步骤，具体代码如下：
```cpp
#ifndef SKINNING_H
#define SKINNING_H

#include "utility.h"
#include "NURBSCurve.h"
using namespace std;
using namespace t_mesh;
class Skinning {
public:
	Skinning(const vector<NURBSCurve>& _curves) :curves(_curves) {
		curves_num = curves.size();
	}
	virtual void parameterize();    // 曲线参数化得到u方向参数 u_knots
	virtual void init() = 0;        // 根据NUUBSCurve初始化T-preimage
	virtual void insert() = 0;      // 在T-preimage中插入中间节点
	virtual void calculate();       // 计算流程

	// update的辅助函数，用于计算基函数的线性组合系数
	void basis_split(const map<double, Node<Point3d>*>& fewer_map, const map<double, Node<Point3d>*>& more_map, map<double, Point3d>& coeff);
	
	// update coordinates of control points by the formula from (nasri 2012)
	// aX' + bW + cY' = V
	void update();

	// 用于skinning过程中显示中间结果
	void setViewer(Viewer* _viewer) {
		viewer = _viewer;
	}

public:
	Mesh3d tspline;
protected:
	int curves_num;
	VectorXd s_knots;
	vector<NURBSCurve> curves;
	Viewer* viewer;

};
#endif // !SKINNING_H
```
在Skinning中`calculate`函数是计算的入口，用于执行各个步骤，默认的计算步骤如下：
```cpp
void Skinning::calculate()
{
	// 1. compute s-knot for curves
	parameterize();
	// 2. construct basis T-mesh 
	init();
	// 3. insert intermediate vertices
	// the coordinate of vertices is the midpoint of the corresponding points in C_r and C_(r+1)
	insert();

	// 4. update coordinates of control points by the formula from (nasri 2012)
	// aX' + bW + cY' = V
	update();
}
```

其他所有的T样条蒙皮方法的具体实现类均继承自Skinning类。