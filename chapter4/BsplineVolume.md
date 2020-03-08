# BsplineVolume
相关文件：
- BsplineVolume.h
- BsplineVolume.cpp

一个B样条体的方程可表示如下：
$$
H(u,v,w) = \sum_{i=0}^m\sum_{j=0}^n\sum_{l=0}^pP_{ijl}N_{i,k_u}(u)N_{j,k_v}(v)N_{l,k_w}(w)
$$

### BSplineVolume类
```cpp
#pragma once
#ifndef BSPLINEVOLUME_H
#define BSPLINEVOLUME_H
#include "Volume.h"
#include "FitPoint.hpp"

const double PI = 3.141592653;

class BsplineVolume :public Volume{

public:
	vector<vector<vector<Point3d>>> control_grid;   // 控制网格
	vector<Eigen::VectorXd> knot_vector;            // u,v,w方向的节点向量

	// 用于体拟合迭代时缓存相应基函数的值
	vector<vector<vector<vector<double>>>> matri;   
	vector<int> Bi_start_indexs;
	vector<int> Bj_start_indexs;
	vector<int> Bk_start_indexs;
};
#endif // !BSPLINEVOLUME_H

```
其中`control_grid, knot_vector`是B样条体的控制网格和$u,v,w$方向的节点向量。


### API
以下是BsplineVolume主要的函数，其他函数及具体实现见代码。
```cpp
#pragma once
#ifndef BSPLINEVOLUME_H
#define BSPLINEVOLUME_H
#include "Volume.h"
#include "FitPoint.hpp"

const double PI = 3.141592653;

class BsplineVolume :public Volume{
public:
	// 获取参数值(u,v,w)对应的点的坐标
	Point3d eval(double u, double v, double w) override;
	// read volume from file
	int readVolume(string) override;
	// save volume to file
	int saveVolume(string) override;

	// 获取参数值为t的等参面，dir为'u'或'v'或'w'
	void get_isoparam_surface(NURBSSurface& surface, double t, char dir);


	// 使用优化方法，生成一个拟合指定点，且雅克比值很好的B样条体
	void fitBsplineSolid(vector<FitPoint3D>& fit_points, int x_points, int y_points, int z_points, double alpha, double delta);

	//体拟合迭代的误差
	double GetSoildFiterror(vector<FitPoint3D>& fit_points,
		int x_points, int y_points, int z_points, double alpha, double delta);

	
	// lspia 拟合给定的数据点
	void lspia(vector<FitPoint3D>& fit_points, int x_points, int y_points, int z_points, int max_iter_num = 100, double eps = 1e-5);
};
#endif // !BSPLINEVOLUME_H
```