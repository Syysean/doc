
# 大文件支持

```shell
git init .
# 让仓库支持LFS
git lfs install
git lfs track "*"
git add .gitattributes
git commit -m "Add \"*.bigfile\" LFS config "

# 添加大文件
git add dyrone.bigfile
git commit -m "Add a really big file"
# 查看跟踪的大文件
git lfs track
git push
```

常用命令
```shell
git lfs status  // 查看当前git lfs对象的状态

git lfs ls-files  // 查看当前哪些文件是使用lfs管理的

# 放弃工作区的更改
git checkout -- .
# 放弃暂存区的更改
git reset HEAD
```


## 资产仓库
目前使用的是腾讯工蜂社区版 git lfs 进行资产管理

###### 查看仓库的容量

进入 [项目首页](https://git.code.tencent.com/OpenHUTB/Content) （如果没有权限访问，则注册后发送用户名到 [whd@hutb.edu.cn](whd@hutb.edu.cn) ），打开左侧的`设置->高级设置`，在页面中选择`版本库设置`。该产品的版本库为5GB，LFS储存为500GB。

###### 注册网络回调钩子

按照上一步中的操作，进入`设置->高级设置->网络回调钩子`，设置 Url 和秘密令牌，当有推送事件时候触发 hutb 仓库的编译。

## 常见问题

###### 拉取时候出现错误：smudge filter lfs failed
设置以下命令后再拉取
```shell
# 跳过污点：稍后会以更快的速度批量下载二进制文件。
set GIT_LFS_SKIP_SMUDGE=1
git pull
# 获取二进制文件
git lfs pull
```



## 参考

* [如何存储 Git 大文件？](https://www.cnblogs.com/88223100/p/How-do-I-store-large-Git-files.html) 

* [如何使用 Git LFS](https://help.aliyun.com/zh/yunxiao/user-guide/how-to-use-git-lfs)




