![nvidia_logo](img/ue/nvidia_hor.png)

# NVIDIA HUTB 生态系统

NVIDIA 是一家跨国科技公司，提供高性能 GPU，同时也是人工智能领域的领先开发者。NVIDIA 赞助了 CARLA 模拟器的开发，并通过为自动驾驶汽车模拟社区提供技术贡献来支持其发展。

以下章节概述了已集成到 HUTB 中的各种 NVIDIA 技术。


# [SimReady](nvidia_simready.md)

SimReady 是针对用于模拟的 3D 资产所附加信息和元数据的 3D 标准规范。SimReady 标准确保 3D 资产不仅是数字孪生中的高保真视觉资源，而且还具有组织良好的元数据结构，包含语义标签、密集字幕、非视觉传感器信息和物理属性。有关在 HUTB 中使用 SimReady 资产的详细信息，请参阅 [SimReady 文档](nvidia_simready.md) 。


# [神经渲染](nvidia_nurec.md)

NVIDIA 的神经渲染技术使神经网络能够从现实世界中捕获的一系列传感器数据（例如一系列 2D 摄像头图像或激光雷达数据）中学习出丰富的 3D 环境表征。之后，可以在 3D 表征中应用变化和随机化，例如在重新模拟之前扰动轨迹或调整传感器配置。这有助于仅使用一个记录的传感器数据序列来生成场景的各种扰动。神经渲染是丰富训练数据或测试场景的强大工具。有关如何安装和使用该工具的详细信息，请参阅 [神经渲染文档](nvidia_nurec.md) 。




# [Cosmos 世界基础模型](https://baike.baidu.com/item/Cosmos%E4%B8%96%E7%95%8C%E5%9F%BA%E7%A1%80%E6%A8%A1%E5%9E%8B/65287190?fr=aladdin)

由生成式世界基础模型、高级标记器、护栏模块及加速视频处理管道四大核心组件构成。该平台通过整合文本、图像、视频及传感器数据生成遵循物理规律的高质量合成视频数据，致力于降低自动驾驶汽车与机器人领域的数据采集成本，支持开发者通过微调构建定制化模型，并可与 Omniverse 仿真平台协同实现未来情景预测。

# [Cosmos Transfer](nvidia_cosmos_transfer.md)

NVIDIA 的 Cosmos Transfer 是 Cosmos 世界基础模型( [Cosmos World Foundation Models (WFMs)](https://www.nvidia.com/en-us/ai/cosmos/) , WFM) ) 的一个分支，专门用于多模态可控条件世界生成或世界到世界的迁移。Cosmos Transfer 旨在弥合模拟环境和真实世界环境之间的感知鸿沟。

用户可以通过简单的文本提示，从 CARLA 序列生成无穷无尽的超逼真视频变体。此功能非常适合以下场景：

* 扩展感知数据集中的视觉多样性
* 弥合模拟训练与真实训练之间的领域差距
* 探索具有逼真纹理、光照和天气变化的极端情况

有关如何将此工具与 CARLA 一起使用的详细信息，请参阅 [Cosmos Transfer 文档](nvidia_cosmos_transfer.md) 。

