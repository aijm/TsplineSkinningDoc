# NURBS曲面
相关文件：
- NURBSSurface.h
- NURBSSurface.cpp

设NURBS曲面 $S(u,v)$ 的$u,v$方向的阶为$k_u,k_v$，控制顶点为$\{P_{ij}\},i=0,1,\ldots,m;j=0,1,,\ldots,n$，$u$方向节点向量为$u_0,u_1,\ldots,u_{m+k_u}$，$v$方向节点向量为$v_0,v_1,\ldots,v_{n+k_v}$，则曲线方程为：
$$
S(u,v) = \sum_{i=0}^m\sum_{j=0}^nP_{ij}N_{i,k_u}(u)N_{j,k_v}(v)
$$

### NURBSSurface类
下面是NURBSSurface的类表示:
```cpp
// This file is part of NURBS, a simple NURBS library.
// github repo: https://github.com/aijm/NURBS
// Copyright (C) 2018 Jiaming Ai <aichangeworld@gmail.com>
using namespace Eigen;
using namespace std;
struct NURBSSurface
{
	NURBSSurface():id(-1),isRational(false){}

	/*input format:
	_order       : // _order(0):u direction; _order(1): v direction
	_controlP: _controlP[i] represents u direction control point ,matrix (m+1) by 3 or 4
				  v
				  |
	_controlP[n]: | P_0n P_1n ... P_mn
	_controlP[i]: | ...
	_controlP[1]: | P_01 P_11 ... P_m1
	_controlP[0]: | P_00 P_10 ... P_m0
				   ------------------------> u

	_uknots   : u_0,u_1,...,u_(m+u_order)
	_vknots   : v_0,v_1,...,v_(n+v_order)
	_isRational:          */

	bool isRational = false;
	int u_order; // order of u direction
	int v_order; // order of v direction
	int u_num; // the final index of u direction control point
	int v_num; // the final index of v direction control point
	VectorXd uknots; // knots vector u direction : u_0, u_1, ..., u_(m + u_order)
	VectorXd vknots; // knots vector v direction : v_0,v_1,...,v_(n+v_order)
	vector<MatrixXd> controlP;
	vector<MatrixXd> controlPw;
	int dimension; // the dimension of control point 2 or 3 or 4

	// use libigl mesh(V,F) structure to show the surface
	MatrixXd mesh_V;
	MatrixXi mesh_F;
	int id;
};
```
其中控制顶点通过`vector<MatrixXd>`表示，`vector`中每一项存储了一行$u$方向的控制顶点。

### NURBSSurface文件格式
以下几个函数用于读取和保存NURBSSurface，以及将曲面离散保存为三角网格
```cpp
// load
bool loadNURBS(string);

// save 
bool saveNURBS(string);

// 按resolution间隔划分参数域，将曲面离散为三角网格并保存为OBJ格式
bool saveAsObj(string, double resolution = 0.01);
```
NURBSSurface详细的文件格式见`loadNURBS,saveNURBS`的具体实现，
以下是一个简单的例子(#为注释，实际文件中需去除)：
```
# 控制顶点为P[i][j]
0      # isRational, 0表示非有理，1表示有理
5 5    # u_num, v_num
4 4    # u_order, v_order
3      # 点的维数
-25 -25 -10 ,-25 -15 -5 ,-25 -5 0 ,-25 5 0 ,-25 15 -5 ,-25 25 -10, # P[0]
-15 -25 -8 ,-15 -15 -4 ,-15 -5 -4 ,-15 5 -4 ,-15 15 -4 ,-15 25 -8, # P[1]
-5  -25 -5 ,-5 -15 -3 ,-5 -5 -8 ,-5 5 -8 ,-5 15 -3 ,-5 25 -5,      # P[2]
5  -25 -3 ,5 -15 -2 ,5 -5 -8 ,5 5 -8 ,5 15 -2 ,5 25 -3,            # P[3]
15  -25 -8 ,15 -15 -4 ,15 -5 -4 ,15 5 -4 ,15 15 -4 ,15 25 -8,      # P[4]
25 -25 -10 ,25 -15 -5 ,25 -5 2 ,25 5 2 ,25 15 -5 ,25 25 -10,       # P[5]
0 0 0 0 1 2 3 3 3 3 # u_knots
0 0 0 0 1 2 3 3 3 3 # v_knots
```

#### 另一种文件格式
另一种文件格式的例子见`out/nurbs/Bsurface.cpt`，该文件格式的读取、保存见`test.h`文件中两个函数
```cpp
// test.h
static void load_nurbs_surface(NURBSSurface& surface, string filename);
static void save_nurbs_surface(const NURBSSurface& surface, string filename);
```
实际测试读取、保存见：
```cpp
// test.h
static void test_load_nurbs_surface();
static void test_save_nurbs_surface();
```

### NURBS曲面显示
显示相关函数如下：
```cpp
// display by libigl
void draw(igl::opengl::glfw::Viewer& viewer, bool showpolygon=true,bool showsurface=true,double resolution = 0.01);

// draw controlpolygon
void drawControlPolygon(igl::opengl::glfw::Viewer &viewer);

// draw NURBS surface
void drawSurface(igl::opengl::glfw::Viewer &viewer, double resolution = 0.01);
```
调用`draw`函数可以通过`showpolygon,showsurface`两个布尔变量控制显示控制网格和NURBS曲面。
而NURBS曲面是通过`resolution`划分参数域进而离散为三角网格，在libigl中表示为`MatrixXd mesh_V,MatrixXi mesh_F`，则libigl可以渲染显示。

#### 显示平均曲率，高斯曲率
在`drawSurface`函数中，以下代码用于计算平均曲率并为曲面加上相应的颜色映射用于显示
```cpp
Eigen::MatrixXd HN;
Eigen::VectorXd H;
Eigen::SparseMatrix<double> L, M, Minv;
igl::cotmatrix(mesh_V, mesh_F, L);
igl::massmatrix(mesh_V, mesh_F, igl::MASSMATRIX_TYPE_VORONOI, M);
igl::invert_diag(M, Minv);
HN = -Minv*(L*mesh_V);
H = HN.rowwise().norm(); //up to sign

// compute curvatrue directions via quadric fitting
Eigen::MatrixXd PD1, PD2;
Eigen::VectorXd PV1, PV2;
igl::principal_curvature(mesh_V, mesh_F, PD1, PD2, PV1, PV2);
// mean curvature
H = 0.5*(PV1 + PV2);

// Compute pseudocolor
Eigen::MatrixXd C;
igl::jet(H, true, C);
viewer.data().set_colors(C);
```
高斯曲率也可以通过igl的相关模块实现，具体请见libigl相关内容。
> [高斯曲率](https://libigl.github.io/tutorial/#gaussian-curvature)
> [平均曲率](https://libigl.github.io/tutorial/#curvature-directions)

## API
NURBSSurface的函数如下，具体实现细节见代码
```cpp
// This file is part of NURBS, a simple NURBS library.
// github repo: https://github.com/aijm/NURBS
// Copyright (C) 2018 Jiaming Ai <aichangeworld@gmail.com>

#ifndef NURBSSURFACE_H
#define NURBSSURFACE_H

using namespace Eigen;
using namespace std;
struct NURBSSurface
{
	NURBSSurface():id(-1),isRational(false){}

	NURBSSurface(VectorXi _order, vector<MatrixXd> _controlP, VectorXd _uknots, VectorXd _vknots, bool _isRational = false);

	// load
	bool loadNURBS(string);

	// save 
	bool saveNURBS(string);

    // 按resolution间隔划分参数域，将曲面离散为三角网格并保存为OBJ格式
	bool saveAsObj(string, double resolution = 0.01);

	// find the knot interval of t by binary searching
	int find_ind(double t, int k, int n, const VectorXd& knots);

	// calculate coordinate of curve point with parameter u & v
	MatrixXd eval(double u, double v) const;

	MatrixXd eval(double t, const MatrixXd &_controlP, const VectorXd &knots);

	// 获取等参线
	void get_isoparam_curve(NURBSCurve& curve, double t, char dir = 'u');

	// knot insertion
	bool insert(double s, char dir='u');
	// kont insertion
	bool insert(double s, double t);

	// display by libigl
	void draw(igl::opengl::glfw::Viewer& viewer, bool showpolygon=true,bool showsurface=true,double resolution = 0.01);

	// draw controlpolygon
	void drawControlPolygon(igl::opengl::glfw::Viewer &viewer);

	// draw NURBS surface
	void drawSurface(igl::opengl::glfw::Viewer &viewer, double resolution = 0.01);

	// surface skinning
	void skinning(const vector<NURBSCurve> &curves, igl::opengl::glfw::Viewer &viewer);

	// surface skinning
	void skinning(const vector<NURBSCurve> &curves,const VectorXd& curves_param, igl::opengl::glfw::Viewer &viewer);

public:
	double mean_curvature(double u, double v) const;
	double guassian_curvature(double u, double v) const;

	void curvature(double u, double v, double& k1, double& k2) const;
	void derivative(double u, double v, RowVector3d& du, RowVector3d& dv, RowVector3d& d2u, RowVector3d& d2v, RowVector3d& duv) const;

	static int FindSpan(const Eigen::MatrixXd &knots, double t, int p = 3);
};
#endif // !NURBSSURFACE_H
```