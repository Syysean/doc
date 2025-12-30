# `DReyeVR` 开发

对于想要深入了解 DReyeVR 内部运作原理以及如何开始开发和编写代码的用户来说，无需再四处寻找了！

（本指南假设您已阅读 [`Usage.md`](../Usage.md) 文档并已安装 `DReyeVR` ）。

# 入门

我们建议您采用一种开发环境，以便能够快速识别您对 DReyeVR 的更改与上游更改之间的差异。为此，我们提供了一个预装（并已提交）DReyeVR 的 CARLA 分支，这样您就可以使用一个干净的初始代码库：
```bash
# 克隆我们的分支并替换你原有的 CARLA 仓库。
git clone https://github.com/harplab/carla -b DReyeVR-0.9.13 --depth 1
cd carla
# ./Update.sh # 在 Linux/Mac 系统中
Update.bat # 在 Windows 系统中

cd ../DReyeVR/ # 假设 DReyeVR 代码库与 carla 代码库相邻。

# (在 DReyeVR 仓库)
make install CARLA=../carla # 安装未被 Git 跟踪的内容，例如蓝图/二进制文件

cd ../carla # 切换回 carla 目录
git status
# 现在，git 应该显示相对于我们上游 DReyeVR 分支的更改，而不是相对于 CARLA 0.9.13 分支的更改。
```

## 反向安装

一旦您对 Carla 代码库中与 DReyeVR 相关的部分进行了更改，手动将所有这些更改复制回 DReyeVR 代码库（如果您想将其提交到上游）将非常繁琐。作为我们构建系统的一部分，我们提供了一个“反向安装”（`r-install`）程序，用于镜像安装 `install` 功能，并将 DReyeVR（通过 `make install`）安装的所有相应文件复制回 DReyeVR：

<details>

<summary> 点击打开示例以生成输出</summary>

```bash
make r-install CARLA=../carla # 相当于 "make rev"
make rev CARLA=../carla       # r-install 的别名

Proceeding on /PATH/TO/CARLA (git branch)
/PATH/TO/CARLA/Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/ -- found
/PATH/TO/CARLA/Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/EgoVehicle.h -- found
/PATH/TO/CARLA/Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/EgoVehicle.h -> /Users/gustavo/carla/DReyeVR-Dev/DReyeVR/EgoVehicle.h
/PATH/TO/CARLA/Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/FlatHUD.cpp -- found
/PATH/TO/CARLA/Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/FlatHUD.cpp -> /Users/gustavo/carla/DReyeVR-Dev/DReyeVR/FlatHUD.cpp
...etc.
...
Done Reverse Install!
```
</details>
<br>

请注意，复制回 DReyeVR 的文件遵循 [`Paths/*.csv`](../Scripts/Paths/) 中定义的 DReyeVR <--> Carla 文件对应关系，因此，如果您修改了一个全新的文件（DReyeVR 未跟踪的文件），则需要手动将该文件添加到 DReyeVR 存储库并更新对应关系文件 (.csv)。

## 典型工作流程

The workflow we have designed for our development process on DReyeVR includes using our fork of carla (`DReyeVR-0.9.13` branch) alongside a cloned `DReyeVR` repo that we can use to both push and pull from upstream.

<details>

<summary>Click to open terminal example</summary>

```bash
> ls
carla.harp/    # our HarpLab fork for primary development
DReyeVR        # our DReyeVR installation

cd carla.harp
... # make some changes in carla.harp
make launch && make package # ensure carla still works with these changes

cd ../DReyeVR
make rev CARLA=../carla.harp # "reverse-install" changes from carla.harp to DReyeVR
git stuff # do all sorts of upstreaming and whatnot.

----------------- # if changes have been made upstream for you to install
cd DReyeVR/
git pull # upstream changes
make clean CARLA=../carla.harp   # optional to reset carla.harp to a clean git state
make install CARLA=../carla.harp # install new DReyeVR changes over it
cd ../carla.harp && make launch && make package && etc.

# optionally, you can keep a carla.vanilla around to test that a fresh install of your updated DReyeVR repo works on carla
make install CARLA=../carla.vanilla
```

</details>
<br>

![Directories](Figures/Dev/Directories.jpg)

# 了解 Carla + DReyeVR 代码库的位置

在 Carla 上进行开发时，您需要重点关注以下几个方面：

1. `Unreal/CarlaUE4/Source/CarlaUE4/DReyeVR/`
    - 这包含了我们所有的自定义 DReyeVR C++ 代码，这些代码通常是在现有的 Carla 代码基础上构建的。
2. `Unreal/CarlaUE4/Plugins/Carla/Source/Carla/`
    - 这里定义了 UE4 C++ Carla 的主要逻辑，涵盖了从传感器到车辆，再到记录器/回放器和天气等所有内容。
    - 这里有一些代码，例如用于自定义传感器和小功能补丁的代码。
3. `LibCarla/source/carla/`
    - 这里存放了几乎所有与 `Python` 交互的 Carla C++ 代码。其中大部分是对 `CarlaUE4/Plugins` 代码的重新实现，但没有使用 Unreal C++ API，并且非常注重向 Python API 传输数据流逻辑。
    - 这里有一小段代码，用于确保传感器数据能够正确地传输到 Python。
4. `PythonAPI/examples/`
    - 在这里您可以找到与 Carla 交互的大部分重要 Python 脚本。
    - 这里有一些文件，用于改善 DReyeVR 和 Carla 的 PythonAPI 的使用体验。


# 内部运作

本节将讨论 DReyeVR 的内部运作，包括设计范式以及与 Carla 的相应握手。

## EgoVehicle
- 相关文件： [EgoVehicle.h](../DReyeVR/EgoVehicle.h), [EgoVehicle.cpp](../DReyeVR/EgoVehicle.cpp), [EgoInputs.cpp](../DReyeVR/EgoInputs.cpp), [Content/DReyeVR/EgoVehicle/BP_*.uasset](../Content/DReyeVR/EgoVehicle/)

EgoVehicle 是我们的“英雄车”，也是我们的主要载体。为了提升 Carla 中人类驾驶员的沉浸感，EgoVehicle 包含许多普通 Carla 车辆所不具备的以人为中心的功能。例如，EgoVehicle 定义了车内后视镜、动态方向盘、仪表盘、人为输入、音频等的逻辑。这些都是人工智能车辆无需关注的功能，因此 Carla 在其他所有车辆中都省略了这些功能。


不过，EgoVehicle 只是标准 [`ACarlaWheeledVehicle`](https://github.com/OpenHUTB/hutb/blob/hutb/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Vehicle/CarlaWheeledVehicle.h) 的一个封装（子类），因此它会自动继承所有 Carla 车辆操作，并兼容所有 CarlaVehicle 功能。值得注意的是，EgoVehicle 并非由玩家拥有，而是由默认的 [`AWheeledVehicleAIController`](https://github.com/OpenHUTB/hutb/blob/hutb/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Vehicle/WheeledVehicleAIController.h) 拥有。这样做是为了允许玩家和内置的 Carla 自动驾驶系统同时进行输入。我们将在下文的 `DReyeVRPawn` 部分对此进行更详细的讨论。

重要的是，我们所有的节拍同步逻辑都基于 EgoVehicle，其 `Tick` 方法会调用 DReyeVR 中许多其他关键组件的 `Tick` 方法。这确保了模拟器中更新的顺序是确定且一致的，并且将来可以依赖这种顺序。

在这里，我们也手动管理 EgoVehicle 的回放行为，它会遵循从 EgoSensor 捕获的值，而不是 Carla 的默认行为，这样我们就可以更精确地重现从 EgoSensor 收集的确切数据，例如眼睛凝视、相机方向、车辆输入和姿态等。 


我们还定义了车辆中三个后视镜的生成和管理逻辑，因为它们默认情况下并未包含在蓝图网格中。将它们分开处理是明智之举，因为在模拟引擎中，使用平面反射实现的后视镜会严重影响性能，因此应谨慎使用。我们还可以为每个后视镜定义画质设置，以动态调整其分辨率及其相应的性能影响。

EgoVehicle 包含指向几乎所有其他主要 DReyeVR 对象的指针，以便它们能够无缝通信。这些指针在构造时设置，并在这些对象的整个生命周期内保持不变。重要的是，EgoVehicle 会生成并附加 EgoSensor，因此它们本质上是相互关联的，彼此不可或缺。

此外，EgoVehicle 的所有输入逻辑都保存在 `EgoInputs.cpp` 源文件中，这样做纯粹是为了将该逻辑与 EgoVehicle 源代码的其余部分分离。

最后，我们在 Carla 世界中生成 EgoVehicle 的方法是：复制一个现有的 Carla 载具蓝图，并将蓝图中的基类 [重新父级化 reparenting](https://forums.unrealengine.com/t/how-to-change-parent-class-of-blueprint-asset/281843) 到我们的 EgoVehicle。该蓝图位于 `EgoVehicle` 的 `内容(Content)` 文件夹中，所有相关的蓝图都整理在这里。


## EgoSensor

- 相关文件： [EgoSensor.h](../DReyeVR/EgoSensor.h), [EgoSensor.cpp](../DReyeVR/EgoSensor.cpp), [DReyeVRSensor.h|cpp](../Carla/Sensor/DReyeVRSensor.h), [DReyeVRData.h|cpp](../Carla/Sensor/DReyeVRData.h)

EgoSensor 是我们用于追踪各种我们可能感兴趣的以人为中心的数据的虚拟 Carla 传感器。它可以被视为一个**在 Carla 世界中运行的隐形数据采集器**。与其他大多数具有物理描述并安装在 Actor 上的 Carla 传感器不同，EgoSensor 会随 EgoVehicle 自动生成和销毁，并在其整个生命周期内绑定到 EgoVehicle 实例。

EgoSensor 是 `DReyeVRSensor` 的子类，而 DReyeVRSensor 又是通用 `CarlaSensor` 的子类，后者源自 Carla 的“["添加传感器教程(add a sensor tutorial)"](https://openhutb.github.io/doc/tuto_D_create_sensor/)”。`DReyeVRSensor` 父类位于代码库的 `CarlaUE4/Plugin/Source` 区域，因为它遵循了 Carla 的规范实现。重要的是，该类是虚类`virtual`（抽象类），这意味着它应该被另一个提供实现的类（即 `EgoSensor`）继承。


由于 DReyeVR 引入了一些 Carla 本身并不依赖的组件（例如用于眼动追踪的 SRanipal 和用于方向盘硬件的 LogitechWheelPlugin），因此我们在 `EgoSensor` 中实现了它们的接口，而不是在 `CarlaUE4/Plugin/Source` 区域（该区域仅供 Carla 使用）。EgoSensor 随后实现了从 SRanipal 和 Logitech 获取数据并将其格式化为适用于当前模拟器时间步的 `DReyeVRData` 数据包所需的方法。


`DReyeVRData` 类是 `CarlaUE4/Plugin/Source/Carla/DReyeVRData.h` 中定义的一系列结构体，它定义了 DReyeVR 跟踪的各种数据类型的结构。例如，它包含了眼睛凝视数据、车辆输入、其他自车变量等结构体。这样的设计旨在鼓励未来的数据类型遵循类似的结构体设计，并与 DReyeVR 进行接口集成，以便将所有数据收集到 `AggregateData` 中，然后作为一个完整的数据包发送到 Python API 进行流式传输，或发送到记录器进行序列化。


EgoSensor 还实现了其他一些不错的功能，例如相机屏幕截图和启用后带有注视点渲染的可变速率着色。

### 继承关系图：
To clarify the structure of the inheritance at play here (Older generation to youngest):
1. [`AActor`](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/GameFramework/AActor/) (UE4): Low-level unreal class for any object that can be spawned into the world
2. [`ASensor`](https://github.com/carla-simulator/carla/blob/0.9.13/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Sensor/Sensor.h) (Carla): Carla actor that templates the structure for Sensors acting in the Carla world
3. [`ADReyeVRSensor`](../Carla/Sensor/DReyeVRSensor.h) (DReyeVR): Our sensor instance that contains logic for all Carla related tasks
    - Streaming to the PythonAPI
    - Receiving data from replayer to reenact
    - Contains instance of `DReyeVR::AggregateData` containing ALL the data
4. [`AEgoSensor`](../DReyeVR/EgoSensor.h) (DReyeVR): Our primary actor containing all DReyeVR related logic for custom data variables/functions
    - Eye tracking logic (SRanipal), ego vehicle tracking, etc. 

## DReyeVRPawn

- Relevant files: [DreyeVRPawn.h](../DReyeVR/DReyeVRPawn.h), [DreyeVRPawn.cpp](../DReyeVR/DReyeVRPawn.cpp)

Getting back to our discussion on the EgoVehicle simultaneous input with player & AI, the DReyeVRPawn is the actual entity that the *human player* possesses during the duration of the level. Unlike EgoVehicle/EgoSensor, the DReyeVRPawn is not tied to any particular object and can be thought of as **an invisible floating camera that defines the viewport of the player**. 

The DReyeVRPawn therefore manages the in-game [`UCameraComponent`](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Camera/UCameraComponent/) as well as visual and input logic that the player needs. The [SteamVR](https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/XRDevelopment/VR/VRPlatforms/SteamVR/QuickStart/) integration is managed here, as well as the LogiWheel controls scheme mapping, since this is the object that the player possesses and thus gets first input priority to. We also add some visual eye-candy logic such as a visualization indicator for the eye gaze as a reticle that is drawn on the [SpectatorScreen](https://docs.unrealengine.com/4.26/en-US/SharingAndReleasing/XRDevelopment/VR/DevelopVR/VRSpectatorScreen/) (not visible to the VR player) or flat-screen HUD. 

The EgoVehicle *AI and player* dual-input logic comes from the implementation that the DReyeVRPawn simply forwards commands to the EgoVehicle without directly possessing it, so the Carla AI can still possess and control the EgoVehicle. This enables simultaneous "possession" by the player and the Carla AI controller, since all human player inputs still reach the EgoVehicle and take precedence over the AI. 

## DReyeVRGameMode

- Relevant files: [DReyeVRGameMode.h](../DReyeVR/DReyeVRGameMode.h), [DReyeVRGameMode.cpp](../DReyeVR/DReyeVRGameMode.cpp)

A [`GameMode`](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/HowTo/SettingUpAGameMode/) in UE4 is used to define game logic across levels, and this can be easily done in code (unlike [`LevelScripts`](https://docs.unrealengine.com/4.27/en-US/API/Runtime/Engine/Engine/ALevelScriptActor/), which are tied to individual level blueprints). 

The gamemode class is useful because **we can rely on it to always persist throughout any level instance**, so we can define logic beyond the lifetime of a single EgoVehicle/EgoSensor and operate on a more global level. 

For instance, we have code to change the volume of the in-world sound effects, spawn the EgoVehicle in a particular location, transfer control to the default floating spectator (detatch from the EgoVehicle), and manage media controls for playback of a recording (play/pause/step/rewind/etc.). 

The most important thing that the DReyeVR gamemode does is spawn the DReyeVRPawn, so the human player can possess some actor and interact with the world at all.

## DReyeVRFactory

 - Relevant files: [DReyeVRFactory.h](../DReyeVR/DReyeVRFactory.h), [DReyeVRFactory.cpp](../DReyeVR/DReyeVRFactory.cpp)

Carla uses Factories to spawn all their relevent actors (See [`CarlaActorFactory`](https://github.com/carla-simulator/carla/blob/master/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Actor/CarlaActorFactory.h), [`CarlaActorFactoryBlueprint`](https://github.com/carla-simulator/carla/blob/master/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Actor/CarlaActorFactoryBlueprint.h), [`SensorFactory`](https://github.com/carla-simulator/carla/blob/master/Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Sensor/SensorFactory.h), etc.) which spawn everything from vehicles to pedestrians to sensors and props. This design allows Carla to handle all the dirty work of registering the actors with LibCarla so that LibCarla knows about each actor and they can be interacted with in Python. 

We follow suit with a similar design in our `DReyeVRFactory` which defines the important characteristics for our DReyeVR actors and provides the logic necessary to spawn them. For instance, here we define our actors are labeled uniquely such as `"harplab.dreyevr_vehicle.model3"` to avoid conflict with existing Carla `"vehicle.*"` queries. 

This is class you'll want to modify if you're looking to create new DReyeVR actors (such as new vehicle models), walkers, sensors, etc. 

---

# 向自我传感器(ego-sensor)添加自定义数据
While we provide a fairly comprehensive suite of data in our DReyeVRSensor, you may be interested in also tracking other data that we don't currently enable. 

The first file you'll want to look at is `Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Sensor/`[`DReyeVRData.h`](../Carla/Sensor/DReyeVRData.h) which contains the data structures that compose the contents of the ego sensor. Here you'll define the variable and its serialization methods (read/write/print). 

```c++
/// DReyeVRData.h
class AggregateData // all DReyeVR sensor data is held here
{
public:
    ... // existing code
    float GetNewVariable() const;
    ////////////////////:SETTERS://////////////////////

    ...
    void SetNewVariable(const float NewVariableIn);

    ////////////////////:SERIALIZATION://////////////////////
    void Read(std::ifstream &InFile);

    void Write(std::ofstream &OutFile) const;

    FString ToString() const; // this printing is used when showing recorder info

private:
... // existing code
float NewVariable; // <-- Your new variable
};
```

Then, you'll want to write the implementation in `Unreal/CarlaUE4/Plugins/Carla/Source/Carla/Sensor/`[`DReyeVRData.cpp`](../Carla/Sensor/DReyeVRData.cpp) as inline funcitons. 
```c++
/// DReyeVRData.cpp
...
float AggregateData::GetNewVariable() const
{
    return NewVariable;
}

...
void AggregateData::SetNewVariable(const float NewVariableIn)
{
NewVariable = NewVariableIn;
}

void AggregateData::Read(std::ifstream &InFile)
{
    /// CAUTION: make sure the order of writes/reads is the same
    ... // existing code
    ReadValue<float>(InFile, NewVariable);
}

void AggregateData::Write(std::ofstream &OutFile) const
{
    /// CAUTION: make sure the order of writes/reads is the same
    ... // existing code
    WriteValue<int64_t>(OutFile, GetNewVariable());
}

FString AggregateData::ToString() const // this printing is used when showing recorder info
{
    FString print;
    ... // existing code
    print += FString::Printf(TEXT("[DReyeVR]NewVariable:%.3f,\n"), GetNewVariable());
    return print;
}
...
```
Notes:
- It is nice to contain collections of relevant variables together in structures so they can be better organized. To facilitate this we designed our DReyeVRData to contain various `DReyeVR::DataSerializer` objects, which each implement their own serialization methods. Our  `AggregateData` instance contains all our structs and a lightweight API to access member variables. 
- The above is an example of modifying/adding a new variable directly to a `DReyeVR::AggregateData`. But it would be better to either modify an existing `DReyeVR::DReyeVRSerializer` object or create a new one (inheriting from the virtual class) and define all the abstract methods yourself. This enables a more granular sub-class/struct abstraction like most of our variables.

With this step complete, you are free to read/write to this variable by getting the single global (`static`) instance of the `DReyeVR::AggregateData` class using the `GetData()` function of the EgoSensor as follows:
```c++
// In some other file, for example EgoVehicle.cpp:
float NewVariable = EgoSensor->GetData()->GetNewVariable();
... // your code
EgoSensor->GetData()->SetNewVariable(NewVariable + 5.f); // update the new variable
```
  
## [可选] 向 PythonAPI 客户端传输数据：
In order to see the new data from a PythonAPI client, you'll need to duplicate the code to the LibCarla serializer. This requires looking at `LibCarla/Sensor/s11n/`[`DReyeVRSerializer.h`](../LibCarla/Sensor/s11n/DReyeVRSerializer.h) and following the same template as all the other variables:
```c++
class DReyeVRSerializer
{
    public:
    struct Data
    {
        ... // existing code
        float NewVariable;

        MSGPACK_DEFINE_ARRAY(
        ... // existing code
        NewVariable, // <-- New variable
        )
    };
};
///NOTE: you'll also need to interface with this updated struct:
```
Then, to actually interface with the DReyeVR sensor, you'll need to modify the call to the LibCarla stream to include your `NewVariable`.
```c++
// in Carla/Sensor/DReyeVRSensor.cpp
void ADReyeVRSensor::PostPhysTick(UWorld *W, ELevelTick TickType, float DeltaSeconds)
{
    ... // existing code
    Stream.Send(*this,
                carla::sensor::s11n::DReyeVRSerializer::Data{
                    ... // existing code
                    Data->GetNewVariable(), // <-- New variable
                });
} 
```
And finally, to actually get the data from a PythonAPI call, you'll need to modify the list of available attributes to the DReyeVR sensor object as follows:
```c++
// in LibCarla/source/carla/sensor/data/DReyeVREvent.h
class DReyeVREvent : public SensorData
{
    ...

    public:
    ... // existing code
    float GetNewVariable() const // <-- new code
    {
        return InternalData.NewVariable;
    }

    private:
    carla::sensor::s11n::DReyeVRSerializer::Data InternalData;
};
```
Then finally here you'll define what function to call (the variable getter) to get that data from a PythonAPI client. 
```c++
// in PythonAPI/carla/source/libcarla/SensorData.cpp
class_<csd::DReyeVREvent, bases<cs::SensorData>, boost::noncopyable, boost::shared_ptr<csd::DReyeVREvent>>("DReyeVREvent", no_init)
    ... // existing code
    .add_property("new_variable", CALL_RETURNING_COPY(csd::DReyeVREvent, GetNewVariable))
    .def(self_ns::str(self_ns::self))
;
```
After you modify files in `PythonAPI` or `LibCarla` the PythonAPI will need to be rebuilt in order for your changes to take effect:
```bash
conda activate carla13 # if using conda
(carla13) make PythonAPI
# make sure to fix any build errors that may occur!
```

# TODO: 添加更多开发笔记

# 技巧与诀窍
## 1. 启用交付模式下的日志记录

It is super useful to see the CarlaUE4.log file in shipping mode and this is not the default in Carla (or Unreal) perhaps for performance reasons?

If you want to enable these features then you'll need to add the flag for `bUseLoggingInShipping` in the `Carla/Unreal/CarlaUE4/Source/CarlaUE4.Target.cs` file.

```cs
public class CarlaUE4Target : TargetRules
{
	public CarlaUE4Target(TargetInfo Target) : base(Target)
	{
		Type = TargetType.Game;
		ExtraModuleNames.Add("CarlaUE4");
		bUseLoggingInShipping = true;//  <--- added here
	}
}
```

Then you should be able to find the CarlaUE4.log files (timestamped to avoid overwrite) at `C:\Users\%YOUR_USER_NAME%\AppData\Local\CarlaUE4\Saved\Logs\CarlaUE4.log` (on Windows). Also works for Mac/Linux. See [this](https://forums.unrealengine.com/t/how-to-debug-shipping-build-exclusive-crash/411138) for more information.

---

## 2. 如何执行 `LOG`
Logging is useful to track code logic and debug (especially since debugging UE4 code can be a bit rough). By default in Unreal C++ you can always use `UE_LOG(LogTemp, Log, TEXT("some text and %d here"), 55);` but we streamlined this for DReyeVR specific logging. You can use our `LOG` macros (defined in [`CarlaUE4.h`](../CarlaUE4/CarlaUE4.h)) when you are editing `CarlaUE4/DReyeVR/*.[cpp|h]` files.

The main benefits include:
- Less boilerplate code for the programmer (thats you!)
- All logs have the prefix `DReyeVRLog` so they are easy to filter in the overall `CarlaUE4.log` file
- We also attached neat compile-time prefixes to include `"[{INVOKED_FILE}::{INVOKED_FUNCTION}:{LINE_NUMBER}] {message}"` so you can quickly find where this log was invoked from and differentiate from others
    - A typical DReyeVR log using our macros looks like this:
        ```
        LogDReyeVR: [DReyeVRUtils.h::ReadConfigValue:141] "your message here"
        ```

```c++
... // in CarlaUE4/DReyeVR files
void example() {
    LOG("some text and %d (this text is grey)", 55);
    FString warning_str("warning");
    LOG_WARN("some %s here (this text is yellow!)", *warning_str);
    LOG_ERROR("this text is red!");
}
...
```

If you are working instead in the `CarlaUE4/Plugins/Source/Carla` codebase then you'll be able to use similar macros but prefixed with `DReyeVR_`: `DReyeVR_LOG("blah blah")`, `DReyeVR_LOG_WARN("blah blah warning")`, etc. 

---

## 3. 管理多个 Carla/DReyeVR 版本
- Having separate `python` environments (such as `conda`) is extremely useful to have different `carla` Python packages on the same machine with different versions (such as `LibCarla`). To do this, you can simply create an individual conda environment for each Carla version as described in [`Install.md`](Install.md). Remember to activate your new `conda` environment on a per-shell basis!
- If you plan on having multiple CARLA's installed, you'll need to keep them updated with the appropriate `Content`. Rather than calling the `Update` script every time to update this, you can save the `Content.tar.gz` file and copy it into new `Unreal/CarlaUE4/Content/Carla` directories whenever you have a new repo.

---

## 4. TODO 添加更多技巧和诀窍