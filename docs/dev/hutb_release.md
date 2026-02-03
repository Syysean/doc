# 如何用 Python 打造一款超大文件下载器（以 HUTB 模拟器下载为例）


为大家分享一个实用的 Python 工具（HUTB 模拟器发行版下载工具），它能自动从仓库下载发行版，合并分块文件并解压，最终为用户提供可执行的完整软件包。

![](../img/dev/hutb_downloader.jpg)


!!! 注意
    如果仅使用，可以直接从该链接直接下载： https://gitee.com/OpenHUTB/sw/releases/download/up/hutb_downloader.exe ，双击 hutb_download.exe 后会在当前目录下载整个模拟器（下载或运行时可能会报警告，忽视即可），点击`hutb/CarlaUE4.exe`即可启动模拟器。


## 一、为什么需要这样的工具？

在开源项目分发中，开发者常常面临一个难题：如何让用户方便地获取最新的发行版？传统的方式是提供直接的下载链接，但这种方式有几个明显的缺点：

1. 更新麻烦：每次发布新版本都需要手动更新下载链接

2. 大文件传输问题：Git对于大文件支持不够友好

3. 下载体验差：用户需要手动执行多个步骤才能使用软件

我们的解决方案是：将大文件分割成小块上传到仓库（大文件仓库对最大文件有限制），然后通过一个智能下载器自动下载、合并并解压这些文件。这样既解决了大文件存储问题，又为用户提供了便捷的一键下载体验。

## 二、工具的核心设计

### 2.1 技术选型：为什么选择Python？

Python 以其简洁的语法和丰富的第三方库而闻名。对于这个项目，我们主要利用了以下几个库：

* GitPython：提供与Git仓库交互的高级接口

* shutil：用于文件和目录的高级操作

* zipfile：处理ZIP压缩文件

* os：提供操作系统相关功能

这些库的组合使得我们能够以相对简单的代码实现复杂的功能。

### 2.2 架构设计

整个工具的核心是一个名为 GitRepository 的类，它封装了与Git仓库交互的所有操作。让我们看看这个类的设计亮点：

```python
class GitRepository(object):
    """
    git仓库管理
    """

    def __init__(self, local_path, repo_url, branch='master'):
        self.local_path = local_path
        self.repo_url = repo_url
        self.repo = None
        self.initial(repo_url, branch)

    def initial(self, repo_url, branch):
        """
        初始化git仓库
        :param repo_url:
        :param branch:
        :return:
        """
        if not os.path.exists(self.local_path):
            os.makedirs(self.local_path)

        git_local_path = os.path.join(self.local_path, '.git')
        if not is_git_dir(git_local_path):
            self.repo = Repo.clone_from(repo_url, to_path=self.local_path, branch=branch, progress=Progress(), depth=1)
        else:
            self.repo = Repo(self.local_path)
```
这个类采用了面向对象的设计思想，将Git操作封装为对象的方法，提高了代码的可读性和可维护性。




## 三、关键技术实现

### 3.1 智能下载：断点续传与进度显示

为了提升用户体验，我们实现了下载进度显示功能。通过继承`git.remote.RemoteProgress`类，我们可以实时获取下载进度：

```python
class Progress(git.remote.RemoteProgress):
    def update(self, op_code, cur_count, max_count=None, message=""):
        print(
            'Download (operation code: %s, current count: %s, max count: %s, percentage: %s, message: %s)...'%(
                op_code,
                cur_count,
                max_count,
                cur_count / (max_count or 100.0),
                message or "NO MESSAGE",
            )
        )
```
这个进度显示不仅告诉用户下载的百分比，还提供了操作代码、当前计数和最大计数等详细信息，让用户清楚地知道下载过程的状态。

### 3.2 大文件处理：分块与合并

由于Git对单个文件大小有限制，我们需要将大文件分割成多个小块。这里我们实现了两个关键函数：

**文件分割函数**：

```python
def split_file(input_file, output_dir, chunk_size):
    with open(input_file, 'rb') as f:
        index = 0
        while True:
            chunk = f.read(chunk_size)
            if not chunk:
                break
            # 按0对齐为了保证合并时也是按顺序进行合并
            output_file = os.path.join(output_dir, f'hutb_{index:05d}.dat')
            with open(output_file, 'wb') as out:
                out.write(chunk)
            index += 1
```

**文件合并函数**：

```python
def merge_files(input_dir, output_file):
    files = os.listdir(input_dir)
    files.sort()
    
    with open(output_file, 'wb') as out:
        for file_name in files:
            print("Merging file: ", file_name)
            file_path = os.path.join(input_dir, file_name)
            with open(file_path, 'rb') as f:
                chunk = f.read()
                out.write(chunk)
```
这种分块策略有以下几个优点：

1. 避免了Git对大文件的限制

2. 可以并行下载多个小块，提高下载速度

3. 如果某个小块下载失败，可以只重新下载该部分，实现类似断点续传的功能

### 3.3 权限管理：安全删除只读文件

在Windows系统中，Git创建的某些文件可能被设置为只读属性，这会导致在删除.git文件夹时出现权限错误。我们通过定义一个回调函数解决了这个问题：

```python
def remove_readonly(func, path, _):
    "Clear the readonly bit and reattempt the removal"
    os.chmod(path, stat.S_IWRITE)
    func(path)
```
这个函数会在删除文件前修改其权限，确保删除操作能够成功执行。

## 四、完整工作流程
### 4.1 初始化阶段
1. 检查本地目录是否存在，如果存在则先删除旧版本

2. 克隆Git仓库到本地（使用浅克隆以加快下载速度）

3. 显示下载进度

### 4.2 清理阶段
1. 删除.git文件夹和.gitattributes文件，这些是版本控制相关文件，普通用户不需要

2. 确保下载目录的干净整洁

### 4.3 文件处理阶段
1. 检查是否存在已合并的ZIP文件，如果没有则合并所有.dat文件

2. 解压ZIP文件到当前目录

3. 删除中间文件（ZIP文件和所有的.dat文件）

### 4.4 完成阶段
1. 计算并显示整个下载过程耗时

2. 准备就绪的软件可以直接使用

#### 五、打包为可执行文件
为了让没有Python环境的用户也能使用这个工具，我们将其打包为独立的EXE文件。使用PyInstaller可以轻松实现这一目标：

```shell
pip install pyinstaller
pyinstaller download_from_git.py --onefile --name hutb_downloader.exe
```
这里有几个需要注意的地方：

1. 单文件模式：--onefile参数将所有依赖打包到一个EXE文件中

2. 避免杀毒软件误报：某些杀毒软件可能会将PyInstaller打包的程序误报为病毒。可以通过代码签名或调整打包参数来减少误报

3. 图标定制：可以使用-i参数指定程序图标

## 六、实际应用场景

### 6.1 开源项目分发
对于开源项目维护者，这个工具可以集成到项目的发行流程中。每次发布新版本时，只需要将分块文件上传到Git仓库，用户就可以通过下载器获取最新版本。

### 6.2 企业内部软件更新
在企业环境中，可以使用这个工具来分发内部软件。通过控制Git仓库的访问权限，可以确保只有授权用户能够下载软件。

### 6.3 教育用途
在计算机教育中，教师可以使用这个工具分发教学材料和软件环境，学生只需要运行一个简单的命令就能获得完整的学习环境。

## 七、性能测试

根据实际测试，在网络带宽足够的情况下，整个下载过程大约 2 分钟，合并和解压等操作大约需要 2 分钟。这个时间主要包括：

1. Git仓库克隆：2-3分钟

2. 文件合并：1-2分钟

3. 文件解压：30秒-1分钟


## 八、总结


通过这个Python实现的Git发行版下载器，我们解决了大文件通过Git分发的难题，为用户提供了便捷的一键下载体验。这个工具展示了Python在自动化任务处理方面的强大能力，也体现了模块化的软件设计原则。



无论是对于开源项目维护者，还是企业内部的软件分发，这个工具都提供了一个高效、可靠的解决方案。随着Git在软件开发中的普及，类似的工具将会在软件分发领域发挥越来越重要的作用。

随着技术的不断发展，我们还可以在这个基础上添加更多高级功能，如自动更新、多线程下载、图形界面等，使其成为一个更加完善的软件分发解决方案。希望这个工具的开发思路和实现方法能够给大家带来启发，也期待看到更多基于这个思想的创新应用。

版权声明：本文介绍的代码仅供学习参考，工具代码遵循 MIT 开源协议。


* 源代码：https://github.com/OpenHUTB/hutb
* 文档：https://openhutb.github.io

