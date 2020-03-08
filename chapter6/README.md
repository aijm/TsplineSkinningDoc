# T样条体蒙皮
相关文件：
- VolumeSkinning.h
- VolumeSkinning.cpp
- VolumePiaMethod.h
- VolumePiaMethod.cpp

T样条体蒙皮是从T样条曲面蒙皮延伸的概念，用于从一组给定的T样条曲面生成一个插值这组曲面的T样条体。

**VolumeSkinning**中采用了插入两排中间面的方式，并通过与[MinJae 2018](https://www.sciencedirect.com/science/article/abs/pii/S0010448518301969)
类似的插值公式使得满足插值性。

**VolumePiaMethod**是基于引导线、LSPIA、优化的T样条体蒙皮方法，用于生成一个既满足插值性，且雅克比值全为正的适用于等几何分析的T样条体。

