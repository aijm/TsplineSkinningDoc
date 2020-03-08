# TsplineVolume
相关文件：
- TsplineVolume.h
- TsplineVolume.cpp

本软件中的T样条体实际上是一个半结构的B样条体，即在`w`方向上有着统一的节点向量。

### TsplineVolume类
```cpp
#ifndef TSPLINEVOLUME_H
#define TSPLINEVOLUME_H
#include "Volume.h"
class TsplineVolume : public Volume{
private:
	TsplineVolume& operator=(const TsplineVolume&){}
public:
	TsplineVolume(){}
	TsplineVolume(const TsplineVolume& other);
	~TsplineVolume();

public:
	map<double, Mesh3d*>       w_map;       // T样条体由多层T网格组成，每层T网格对应一个w方向节点值
	Eigen::VectorXd            w_knots;     // w向节点向量
	
};

#endif // !TSPLINEVOLUME_H


```

### API
以下是TsplineVolume主要的函数，其他函数及具体实现见代码。
```cpp
#ifndef TSPLINEVOLUME_H
#define TSPLINEVOLUME_H
#include "Volume.h"
class TsplineVolume : public Volume{
private:
	TsplineVolume& operator=(const TsplineVolume&){}
public:
	TsplineVolume(){}
	TsplineVolume(const TsplineVolume& other);
	~TsplineVolume();
	// 获取参数值为(u,v,w)的点的坐标
	Point3d eval(double u, double v, double w) override;
	// read volume from file
	int readVolume(string) override;
	// save volume to file
	int saveVolume(string) override;

	// insert a layer to the T-mesh of this volume
	void insert(double w, Mesh3d* mesh);
	// get the number of nodes
	int get_num()const;
	
};

#endif // !TSPLINEVOLUME_H


```
