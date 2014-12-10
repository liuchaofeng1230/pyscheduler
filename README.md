pyscheduler
===========

做项目的时候大家都需要的做的一个事情是排计划安排，一般有两种做法：一是用类`Microsoft Project`的软件，这种软件的优点是功能很强大，缺点是有点重，非文本的，想把它嵌入到比如 markdown 文档里面很难；二是自己用表格的语法直接写在一个 markdown 文件里面。我个人比较喜欢第二种做法，这种做法好处是它是纯文本的，不需要借助任务特殊软件就可以展示。它的缺点是功能比较弱：

* 你如果想要对中间某个任务的时间安排做出调整，那么这个任务后面的所有任务你需要重新排一次。
* 你无法像`Microsoft Project`软件那样给你展示整个项目总的资源情况。
* 你无法对整个任务安排进行筛选，让它只显示某个人的工作

这个小项目的作用就是为了解决前面提到的几个问题。


## Usage

### 基本用法

```bash
scheduler.py [-m <man>] /path/to/work-breakdown-file.markdown
```

这个软件的输入是一个`markdown` 文件(已提供,见test.markdown), 你在这个 markdown 文件里面定义你的项目的任务细分，比如下面这段:

```bash
# 项目基本信息
* 项目开始时间: 2014-08-21

# 任务细分
* 任务一 -- 2[James]
* 任务二 -- 1[Lucy]
* 任务三 -- 1[James]
* 任务四 -- 2[Lucy]
```

上面定义了我们这个项目是从`2014-08-21`开始做，并且定义了我们这个项目的主要工作安排，那么运行以下命令，我们可以得到如下的自动排期表(如果你的 Terminal 使用的是等宽字体的话，你会发现下面这个表格是工整对齐的):

```bash
> ./scheduler.py /tmp/test.markdown
任务   | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
------ | ------ | -------- | ---------- | ---------- | ----
任务一 | James  | 2.0      | 2014-08-21 | 2014-08-22 | 0%  
任务二 | Lucy   | 1.0      | 2014-08-21 | 2014-08-21 | 0%  
任务三 | James  | 1.0      | 2014-08-25 | 2014-08-25 | 0%  
任务四 | Lucy   | 2.0      | 2014-08-22 | 2014-08-25 | 0%  

>>> 总人日: 6.0, 已经完成的人日: 0.0, 完成度: 0.00%
```

大家也许注意到了，你如果把这段输出保存成一个`markdown`文件，它其实就形成了一个表格。也就是说你只需要维护上面提到的任务基本信息，利用这个小工具可以自动给你生成排期表。


### 更新任务进度

当然，随着时间的推移你可以对你的任务的进度进行更新：

```bash
# 项目基本信息
* 项目开始时间: 2014-08-21

# 任务细分
* 任务一 -- 2[James][100%]
* 任务二 -- 1[Lucy][80%]
* 任务三 -- 1[James]
* 任务四 -- 2[Lucy]
```
    
再次运行你会得到这样的表格:

```bash
>  ./scheduler.py /tmp/test.markdown
任务   | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
------ | ------ | -------- | ---------- | ---------- | ----
任务一 | James  | 2.0      | 2014-08-21 | 2014-08-22 | 100%
任务二 | Lucy   | 1.0      | 2014-08-21 | 2014-08-21 | 80% 
任务三 | James  | 1.0      | 2014-08-25 | 2014-08-25 | 0%  
任务四 | Lucy   | 2.0      | 2014-08-22 | 2014-08-25 | 0%  

>>> 总人日: 6.0, 已经完成的人日: 2.8, 完成度: 46.67%
```


### 根据任务负责人对任务进行过滤

如果你想只看看所有分配给`James`的任务，那么加个参数即可:

```bash
> ./scheduler.py -m James /tmp/test.markdown
任务   | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
------ | ------ | -------- | ---------- | ---------- | ----
任务一 | James  | 2.0      | 2014-08-21 | 2014-08-22 | 100%
任务三 | James  | 1.0      | 2014-08-25 | 2014-08-25 | 0%  

>>> 总人日: 3.0, 已经完成的人日: 2.0, 完成度: 66.67%
```

### 有人请假怎么办？

做项目的过程中，人员难免请假，对于已经排好的计划怎么办？ 手工调整？不用，你只需要记录谁在哪天请假就好:

```bash
# 项目基本信息
* 项目开始时间: 2014-08-21

# 任务细分
* 任务一 -- 2[James][100%]
* 任务二 -- 1[Lucy][80%]
* 任务三 -- 1[James]
* 任务四 -- 2[Lucy]

# 请假情况
* James -- 2014-08-22
```

再次运行相同命令， 我们会得到如下排期:

```bash
> ./scheduler.py -m James /tmp/test.markdown
任务   | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
------ | ------ | -------- | ---------- | ---------- | ----
任务一 | James  | 2.0      | 2014-08-21 | 2014-08-25 | 100%
任务三 | James  | 1.0      | 2014-08-26 | 2014-08-26 | 0%  

>>> 总人日: 3.0, 已经完成的人日: 2.0, 完成度: 66.67%
```

### 一个大任务的细分

我们有时候会碰到这种问题: 一个任务由很多小任务组成。比如我们做一个网上购物网站，我们可能有几个任务是关于商品的增，删，改，查，很自然的，我们会把描述成这样：

```bash
# 项目基本信息
* 项目开始时间: 2014-08-21

# 商品相关
* 增 -- 2[James]
* 删 -- 1[Lucy]
* 改 -- 1[James]
* 查 -- 2[Lucy]
```

运行命令我们会得到如下的排期:

```bash
任务 | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
-- | ------ | -------- | ---------- | ---------- | ----
增 | James  | 2.0      | 2014-08-21 | 2014-08-22 | 0%  
删 | Lucy   | 1.0      | 2014-08-21 | 2014-08-21 | 0%  
改 | James  | 1.0      | 2014-08-25 | 2014-08-25 | 0%  
查 | Lucy   | 2.0      | 2014-08-22 | 2014-08-25 | 0%  

>> 总人日: 6.0, 已经完成的人日: 0.00, 完成度: 0.00%
```

任务的名字直接显示成`增`,`删`,`改`，`查`, 不知道背景的同学完全不知道这个排期在说什么。

因此我们支持把每个大任务的标题自动添加到每个子任务的任务名字里面去，只要添加`-t`参数:
```bash
> ./scheduler.py -t /tmp/test.md
任务          | 责任人 | 所需人日 | 开始时间   | 结束时间   | 进度
------------- | ------ | -------- | ---------- | ---------- | ----
 商品相关: 增 | James  | 2.0      | 2014-08-21 | 2014-08-22 | 0%  
 商品相关: 删 | Lucy   | 1.0      | 2014-08-21 | 2014-08-21 | 0%  
 商品相关: 改 | James  | 1.0      | 2014-08-25 | 2014-08-25 | 0%  
 商品相关: 查 | Lucy   | 2.0      | 2014-08-22 | 2014-08-25 | 0%  

>> 总人日: 6.0, 已经完成的人日: 0.00, 完成度: 0.00%
```

更多功能？Try it yourself!
