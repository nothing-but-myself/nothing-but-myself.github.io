---
title: IPython 怎么搞定模块重载？
date: 2024-11-27
slug: module-reloading-in-ipython
tags: [python, ipython, workflow]
---
IPython 作为交互式 shell 的典范，它的便捷性毋庸置疑。然而在开发中遇到「 简单修改一行代码，却无法热加载 」的 case 非常让人挠头，反复退出再重启 IPython 的操作绝对是蠢人般的体验。但其实我们稍加施放魔法，就可以解决这个让人哭笑不得的问题，即便你再键指如飞，没法抵制 **autoreload** 这个扩展插件发来的真香告警。

# 痒点

- 频繁改代码 = 频繁 reload
- 深知 *importlib.reload* 牛逼，也不想多敲一个字
- 工作流被打断，原本一气呵成的逻辑，非得因为要重载模块反复复制粘贴
- 浪费时间，拉低开发幸福感

# 怎么弄？

这个插件的启用类似一般的环境变量的配置，分为 *即刻生效版* 和 *一步到位版*，即你是想临时 export 到当前 *shell* 中还是写到 *rc* 文件中方便后续 *shell* 全部启用。

假如你想临时启用，即刻生效，只需在 IPython 中运行如下两个内置命令，这种方式比较适合临时起意，需要调试的情况。但请切记你最好在启动 IPython 后立马运行这两个命令，因为只有在这两个命令之后的所有代码才具备热加载的魔法。

```python
%load_ext autoreload
%autoreload 2
```

搞定！接下来每次执行代码时，IPython 会自动重载那些被改过的函数和模块。

那么适合懒人党的一步到位版又是怎么设置呢？毕竟敲命令还是有忘记的麻烦，直接写入配置文件更符合程序员的思维。

假如你还没配置过 IPython，就先跑一条命令创建个配置文件：

```shell
ipython profile create
```

使用 *vim* 编辑器修改如下配置文件

```shell
vim ~/.ipython/profile_default/ipython_config.py
```

新增如下内容

```python
c = get_config()
c.InteractiveShellApp.extensions = ['autoreload']
c.InteractiveShellApp.exec_lines = ['%autoreload 2']
```

下次启动 IPython，自动重载功能直接生效，再也不用动手动脚了

# 爽完也有坑！

1. 性能问题，每次运行代码都检查模块更新，对大项目可能会有点性能开销（小项目无压力）
2. 某些 C 扩展模块，它压根不支持重载
3. 执行顺序很重要，你必须确保你在导入模块前运行 *autoreload*，不然就会踩坑
4. 对于 _from ... import *_ 支持有限，这玩意儿本身就不推荐用，再加上 *autoreload* 对它也不是 100% 生效，能避则避

# 小尾巴

**autoreload** 在我手，代码修改不发愁。
IPython 调试乐无忧，时间节省幸福留！

官方文档戳 [这里](https://ipython.org/ipython-doc/3/config/extensions/autoreload.html)
