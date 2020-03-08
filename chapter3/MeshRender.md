# MeshRender窗口显示Tspline
相关文件：
- MeshRender.h
- MeshRender.cpp
  
为了方便显示T样条，而且轻松切换T样条的不同显示模式，本软件添加了`MeshRender`模块，可以通过键盘切换不同显示模式并且易于使用。

### MeshRender设计
```cpp
#ifndef MESHRENDER_H
#define MESHRENDER_H


#include "Window.h"
#include "utility.h"
class MeshRender : public Window {
public:
	MeshRender(t_mesh::Mesh3d* _mesh,
		bool _showmesh = false, bool _showpolygon = true,
		bool _showsurface = true, double _resolution = 0.01)
		:mesh(_mesh), showmesh(_showmesh), showpolygon(_showpolygon),
		showsurface(_showsurface), resolution(_resolution)
	{
		mesh->setViewer(&viewer);
	}

protected:
    // 重写draw函数用于T样条的显示及键盘控制
	void draw() override;

private:
    // 需要显示的T样条
	t_mesh::Mesh3d* mesh;
    // 布尔变量用于控制T样条显示模式
	bool showmesh;
	bool showpolygon;
	bool showsurface;
	double resolution;

};

#endif // !MESHRENDER_H
```
由于显示T样条是基于`Window`模块的，因此`MeshRender`继承了`Window`并重写了`draw`函数，在其中加入了T样条的显示及键盘控制，`draw`函数的代码如下：
```cpp
void MeshRender::draw()
{
	mesh->draw(showmesh, showpolygon, showsurface, resolution);
	viewer.callback_key_down = [&](igl::opengl::glfw::Viewer& viewer, unsigned char key, int modifier) ->bool
	{
		if (key == 32) {
			/*viewer.data().clear();
			insert_loop(viewer);*/
		}
		else if (key == 'M') {
			viewer.data().clear();
			showmesh = !showmesh;
			mesh->draw(showmesh, showpolygon, showsurface);
		}
		else if (key == 'P') {
			viewer.data().clear();
			showpolygon = !showpolygon;
			mesh->draw(showmesh, showpolygon, showsurface);
		}
		else if (key == 'S') {
			viewer.data().clear();
			showsurface = !showsurface;
			mesh->draw(showmesh, showpolygon, showsurface);
		}
		return false;
	};
}
```
如代码所示，通过`M,P,S`三个按键，可以很方便的切换T样条的显示模式。

### 使用方法
有了`MeshRender`模块，代码可非常简洁明了。
以`Test.cpp`中的显示T样条曲面为例，代码如下：
```cpp
void Test::test_Mesh() {
	Mesh3d mesh;
	mesh.loadMesh("../out/tspline/bonnet.cfg");
	// 通过MeshRender模块显示T样条曲面
	// 初始时显示模式为仅显示控制网格及曲面
	MeshRender render(&mesh, false, true, true);
	render.launch();
}
```

