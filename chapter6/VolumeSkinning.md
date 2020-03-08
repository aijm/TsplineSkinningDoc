# VolumeSkinning
相关文件：
- VolumeSkinning.h
- VolumeSkinning.cpp
  
**VolumeSkinning**中采用了插入两排中间面的方式，并通过与[MinJae 2018](https://www.sciencedirect.com/science/article/abs/pii/S0010448518301969)
类似的插值公式使得满足插值性。

且**VolumeSkinning**中也抽象了体蒙皮的基本步骤。

### VolumeSkinning类

```cpp
#ifndef VOLUMESKINNING_H
#define VOLUMESKINNING_H

#include "TsplineVolume.h"
#include "NURBSCurve.h"
class VolumeSkinning {
public:
	VolumeSkinning(const vector<Mesh3d>& _surfaces) :surfaces(_surfaces) {
		surfaces_num = surfaces.size();
	}
	
	virtual void parameterize();    // 曲线参数化得到w方向参数 w_params
	virtual void init();        // 根据T样条曲面初始化体网格
	virtual void insert();      // 在体网格中插入中间面
	virtual void calculate();       // 计算流程

	// update coordinates of control points by the formula from (nasri 2012)
	// aX + bW + cY = V
	void update();

	// 用于skinning过程中显示中间结果
	void setViewer(Viewer* _viewer) {
		viewer = _viewer;
	}
private:
	Point3d centerOfmesh(const Mesh3d& mesh);
public:
	TsplineVolume volume;
protected:
	vector<Mesh3d> surfaces;
	int surfaces_num;
	Eigen::VectorXd w_params;
	Viewer* viewer;
};


#endif // !VOLUMESKINNING_H
```

`calculate`中包含了具体的计算步骤：
```cpp
void VolumeSkinning::calculate()
{
	// 1. compute w_params for surfaces
	parameterize();
	// 2. construct basis 3-dimension T-mesh 
	init();
	// 3. insert intermediate surfaces
	// the coordinate of vertices is the midpoint of the corresponding points in C_r and C_(r+1)
	insert();

	// 4. update coordinates of control points by the formula from (nasri 2012)
	// aX' + bW + cY' = V
	update();
}
```
各个步骤具体的实现见代码。
