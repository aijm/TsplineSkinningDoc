# T样条曲面蒙皮
相关文件：
- skinning.h
- skinning.cpp
- NasriMethod.h
- NasriMethod.cpp
- MinJaeMethod.h
- MinJaeMethod.cpp
- PiaMethod.h
- PiaMethod.cpp
- PiaNasriMethod.h
- PiaNasriMethod.cpp
- PiaMinJaeMethod.h
- PiaMinJaeMethod.cpp

其中**Skinning**类是对各种T样条蒙皮方法步骤进行抽象得到的一个抽象基类；

**NasriMethod, MinJaeMethod**分别是对[Nasri 2012](https://link.springer.com/article/10.1007/s00371-012-0692-1), [MinJae 2018](https://www.sciencedirect.com/science/article/abs/pii/S0010448518301969)这两篇论文中T样条曲面蒙皮方法的具体实现；

**PiaMethod**是对本文提出的基于引导线和LSPIA的T样条蒙皮方法步骤进行抽象得到的一个抽象基类；

**PiaNasriMethod, PiaMinJaeMethod**分别是本文方法在插入一条中间线和两条中间线时的具体实现。

> T样条蒙皮方法的理论知识见[Nasri 2012](https://link.springer.com/article/10.1007/s00371-012-0692-1), [MinJae 2018](https://www.sciencedirect.com/science/article/abs/pii/S0010448518301969)，及本软件对应毕业论文

