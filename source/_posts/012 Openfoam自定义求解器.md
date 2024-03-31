---
title: Openfoam自定义求解器
excerpt: 学习oepnfoam的乐趣就在于能自己修改求解器，本文详细如何使用wmake编译自己的求解器，如何使用自定义求解器。
# sticky: 100
published: true
date: "2024/03/20 14:12:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240320141514.png
category: 
- [模拟,Openfoam]
tags:
- Openfoam
- 模拟
---

# 1、求解器

## 1.1 复制源码

本案例以icoFoam为例，复制【openFOAM/OpenFOAM-9/applications/solvers/incompressible/icoFoam】文件夹至run文件夹下（我的是【openFOAM/mtl-9/run/solvers/incompressible】）

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202306131700207.png)

## 1.2 修改名称

将文件夹重新命名为【myIconFoam】
![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202306131703290.png)

修改该目录下的文件名称，为了便于分辨是自己的求解器，在源代码里输出一些内容

![image-20230614001149142](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614001149142.png)

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202306131703091.png)

## 1.3 修改files

修改【Make/files】

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202306131704567.png)

注意：这里路径要改为【$(FOAM_USER_APPBIN)】，与之前【FOAM_USER_APPBIN】区别

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202306131705640.png)

## 1.4 编译

输入【wmake】

![image-20230614001032908](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614001032908.png)

编译成功之后开始使用

# 2、使用

复制一份cavity案例

blockMesh划分网格

myIcoFoam求解

![image-20230614001314507](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614001314507.png)

# 3、进阶

本案例修改icoFoam求解器，添加温度项。
$$\frac{\partial T}{\partial t} + \bigtriangledown \cdot (UT)-{\bigtriangledown }^2(D_{T}T)=0$$

## 3.1 修改源文件

添加以下代码，及上述方程

![image-20230614005652819](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614005652819.png)

```c++
fvScalarMatrix TEqn
(
    fvm::ddt(T) + fvm::div(phi, T) - fvm::laplacian(DT, T)
);
TEqn.solve();
```

## 3.2 修改场文件

新增以下有关温度项的代码

![image-20230614005903552](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614005903552.png)

```c++
IOdictionary myProperties // 新增一个myProperties项
(
    IOobject
    (
        "myProperties",
        runTime.system(), // 存储在system文件夹下
        mesh,
        IOobject::MUST_READ_IF_MODIFIED,
        IOobject::NO_WRITE
    )
);

dimensionedScalar DT // 定义一个变量DT，从myProperties中获取
(
    "DT",
    myProperties.lookup("DT")
);

Info<< "Reading field T\n" << endl;
volScalarField T // 定义一个变量T
(
    IOobject
    (
        "T",
        runTime.timeName(),
        mesh,
        IOobject::MUST_READ,
        IOobject::AUTO_WRITE
    ),
    mesh
);
```

## 3.3 编译

修改完成之后，【wmake】编译

![image-20230614010106515](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010106515.png)

## 3.4 使用

### 3.4.1 初始条件

复制p文件至T，改变名称、单位、初始值和边界条件

![image-20230614010203279](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010203279.png)

### 3.4.2 自定义属性

复制constant里的文件【transportProperties】文件至system，修改名称myProperties

![image-20230614010351434](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010351434.png)

### 3.4.3 新增离散格式

新增div(phi,T)的离散格式

![image-20230614010458808](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010458808.png)

### 3.4.4 新增求解项

![image-20230614010607465](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010607465.png)

### 3.4.5 求解

划分网格：blockMesh

求解：myIcoFoam

![image-20230614010711878](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/image-20230614010711878.png)

完成