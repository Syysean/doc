# AI 渲染

CARLA 的集成 AI 渲染技术扩展了使用在大型真实世界数据集上训练的神经网络创建逼真、高度多样化的模拟数据集的可能性。

CARLA目前支持2种 AI 渲染技术：

* [使用 NVIDIA Omniverse 的 NuRec 工具进行神经重建](nvidia_nurec.md)
* [使用 NVIDIA Cosmos Transfer1 世界基础模型进行风格迁移](nvidia_cosmos_transfer.md) 

---

## 神经重建

NVIDIA 的神经重建技术能够让神经网络从真实世界中采集的一系列传感器数据（例如一系列二维相机图像或激光雷达数据）中学习到丰富的 3D 环境表示。之后，可以在 3D 表示中应用各种变化和随机化，例如扰动轨迹或在重新仿真之前调整传感器配置。这使得仅使用一组记录的传感器数据即可生成场景的各种扰动。神经重建是丰富训练数据或测试场景的强大工具。有关如何安装和使用该工具的详细信息，请参阅 [神经重建文档](nvidia_nurec.md) 。


## Cosmos Transfer

NVIDIA 的 Cosmos Transfer 是 Cosmos 世界基础模型(Cosmos World Foundation Models, WFMs) 的一个分支，专门用于多模态可控条件世界生成或世界到世界的迁移。Cosmos Transfer 旨在弥合模拟环境和真实世界环境之间的感知鸿沟。

用户可以通过简单的文本提示，从 CARLA 序列生成无穷无尽的超逼真视频变体。此功能非常适合以下场景：

* 扩展感知数据集中的视觉多样性
* 弥合模拟训练与真实训练之间的领域差距
* 探索具有逼真纹理、光照和天气变化的极端情况

有关如何将此工具与 CARLA 一起使用的详细信息，请参阅 [Cosmos Transfer 文档](nvidia_cosmos_transfer.md) 。
