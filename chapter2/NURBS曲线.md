# NURBS曲线
相关文件：
- NURBSCurve.h
- NURBSCurve.cpp
  
设NURBS曲线 $P(t)$ 的阶为$k$，控制顶点为$P_0,P_1,\ldots,P_n$，对应的权重为$w_0, w_1, \ldots, w_n$，节点向量为$t_0,t_1,\ldots,t_{n+k}$，则曲线方程为：

$$
P(t) = \frac{\sum_{i=0}^{n}w_iP_iN_{i,k}(t)}{\sum_{i=0}^{n}w_iN_{i,k}(t)}
$$

将$w_i, P_i$表示为齐次坐标形式，即$\widetilde{P}_i = \{w_iP_i, w_i\}$，则

$$
\widetilde{P}(t) = \sum_{i=0}^{n}\widetilde{P}_iN_{i,k}(t)
$$

在NURBSCurve类中`controlPw`即表示了齐次形式的控制顶点。
### NURBSCurve类
下面是NURBSCurve的类表示
```cpp
#ifndef NURBSCURVE_H
#define NURBSCURVE_H

#include <igl/opengl/glfw/Viewer.h>
using namespace Eigen;
using namespace std;
struct NURBSCurve
{
	NURBSCurve(){}
	/*input format:
	_n       : P_0,P_1,...,P_n; _n is the final index
	_k       : order of BSpline
	_controlP: P_0,P_1,...,P_n; (n+1) by 2 or 3
	_knots   : t_0,t_1,...,t_(n+k); */
	NURBSCurve(int _n, int _k, MatrixXd _controlP, VectorXd _knots, bool _isRational = false);

	bool isRational = false;
	int n; // P_0,P_1,...,P_n; _n is the final index
	int k; // order of BSpline
	VectorXd knots;
	MatrixXd controlP;
	MatrixXd controlPw;

};
#endif // !NURBSCURVE_H
```

### NURBSCurve文件格式
以下两个函数用于读取和保存NURBSCurve
```cpp
// load
bool loadNURBS(string);
// save
bool saveNURBS(string);
```
其中NRUBSCurve的格式的一个例子如下：
```
# NURBSCurve 非有理格式示例
0              # isRational，0表示非有理
6 4            # n k
3              # 控制点坐标维数
2    3  0      # 控制点坐标
1.5  1  0
- 1  1  0
- 2   0  0
- 1 - 1  0
1 - 1  0
2    0  0
0    0    0 0 0.25    0.5 0.75    1 1    1    1  # 节点向量

----------------------------------------------------------
1                  # isRational, 1表示有理
6 3                # n k
4                  # 齐次坐标形式的维数
   1    0  0    1  # 齐次坐标
 0.5  0.5  0  0.5
-0.5  0.5  0  0.5
  -1    0  0    1
-0.5 -0.5  0  0.5
 0.5 -0.5  0  0.5
   1    0  0    1
   0    0    0 0.25  0.5  0.5 0.75    1    1    1  # 节点向量
```

### NURBSCurve显示
```cpp
// display by libigl
void draw(igl::opengl::glfw::Viewer& viewer, bool showpolygon=true,bool showsurface=true,double resolution = 0.01, Eigen::RowVector3d color = Eigen::RowVector3d(1, 0, 0));

// draw controlpolygon
void drawControlPolygon(igl::opengl::glfw::Viewer &viewer);

// draw NURBS surface
void drawSurface(igl::opengl::glfw::Viewer &viewer, double resolution = 0.01, Eigen::RowVector3d color = Eigen::RowVector3d(1, 0, 0));
```
如上代码所示，调用`draw`函数可以通过`showpolygon,showsurface`两个布尔变量控制显示控制网格和NURBS曲线。

### API
NURBSCurve的主要函数如下，具体实现细节见代码
```cpp
// This file is part of NURBS, a simple NURBS library.
// github repo: https://github.com/aijm/NURBS
// Copyright (C) 2018 Jiaming Ai <aichangeworld@gmail.com>


#ifndef NURBSCURVE_H
#define NURBSCURVE_H

#include <igl/opengl/glfw/Viewer.h>
using namespace Eigen;
using namespace std;
struct NURBSCurve
{
	NURBSCurve(){}
	/*input format:
	_n       : P_0,P_1,...,P_n; _n is the final index
	_k       : order of BSpline
	_controlP: P_0,P_1,...,P_n; (n+1) by 2 or 3
	_knots   : t_0,t_1,...,t_(n+k); */
	NURBSCurve(int _n, int _k, MatrixXd _controlP, VectorXd _knots, bool _isRational = false);

	NURBSCurve(const NURBSCurve &curve){
		isRational = curve.isRational;
		n = curve.n;
		k = curve.k;
		knots = curve.knots;
		controlPw = curve.controlPw;
	}

	NURBSCurve& operator=(const NURBSCurve &curve){
		isRational = curve.isRational;
		n = curve.n;
		k = curve.k;
		knots = curve.knots;
		controlPw = curve.controlPw;
		return *this;
	}
	// load
	bool loadNURBS(string);
	// save
	bool saveNURBS(string);
	
	// find the knot interval of t by binary searching
	int find_ind(double t)const;

	// evaluate the coordinate of curvePoint with parameter t  
	MatrixXd eval(double t)const;

	MatrixXd eval_tangent(double t) const;

	// chord length parameterization
	static VectorXd parameterize(const MatrixXd &points);
	// basis function N_(i,p)(t)
	static double Basis(const VectorXd &_knots, double _t, int _i = 0, int _p = 3);

	// Berivative of Blending function N[s0,s1,s2,s3,s4](t)
	static Eigen::RowVectorXd DersBasis(const Eigen::MatrixXd &knots, double t, int i = 0, int p = 3);

	// interpolate by bspline of degree 3
	void interpolate(const MatrixXd &points);

	// interpolate with appointed knot vector
	void interpolate(const MatrixXd &points, const VectorXd &knotvector);

	// interpolate with tangent constraint
	void interpolate_tangent(const MatrixXd &points, const MatrixXd &tangent);

	// interpolate with tangent constraint
	void interpolate_tangent(const MatrixXd &points, const MatrixXd &tangent, const VectorXd &params);

	// interpolate with tangent constraint
	void interpolate_tangent_improve(const MatrixXd &points, const MatrixXd &tangent, const VectorXd &params);


	// interpolate with tangent constraint
	void interpolate_optimize(const MatrixXd &points, const MatrixXd &tangent, double learning_rate = 0.01);

	// interpolate with tangent constraint
	void interpolate_optimize(const MatrixXd &points, const MatrixXd &tangent, const VectorXd &params, double learning_rate = 0.01);

	// interpolate with tangent constraint
	void interpolate_optimize1(const MatrixXd &points, const MatrixXd &tangent, const VectorXd &params, double learning_rate = 0.01);
	
	// pia fit by B-spline of degree 3
	void piafit(const MatrixXd &points,int max_iter_num=100, double eps=1e-5);

	// pia fit with appointed knot vector
	void piafit(const MatrixXd &points, const VectorXd &knotvector, int max_iter_num = 100, double eps = 1e-5);

	// given Q_0,...,Q_m, fit by B-spline with control points P_0,...,P_n
	void lspiafit(const MatrixXd & points, int n_cpts, int max_iter_num = 100, double eps = 1e-5);

	// lspia fit with appointed knot vector
	void lspiafit(const MatrixXd & points, const VectorXd& params, int n_cpts, const VectorXd & knotvector, int max_iter_num = 100, double eps = 1e-5);

	// lspia fit with appointed knot vector
	void lspiafit(const MatrixXd & points, const VectorXd& params, const MatrixXd& cpts, const VectorXd & knotvector, int max_iter_num = 100, double eps = 1e-5);


	// kont insertion
	bool insert(double t);

	// display by libigl
	void draw(igl::opengl::glfw::Viewer& viewer, bool showpolygon=true,bool showsurface=true,double resolution = 0.01, Eigen::RowVector3d color = Eigen::RowVector3d(1, 0, 0));

	// draw controlpolygon
	void drawControlPolygon(igl::opengl::glfw::Viewer &viewer);

	// draw NURBS surface
	void drawSurface(igl::opengl::glfw::Viewer &viewer, double resolution = 0.01, Eigen::RowVector3d color = Eigen::RowVector3d(1, 0, 0));


	bool isRational = false;
	int n; // P_0,P_1,...,P_n; _n is the final index
	int k; // order of BSpline
	VectorXd knots;
	MatrixXd controlP;
	MatrixXd controlPw;

};
#endif // !NURBSCURVE_H





```
