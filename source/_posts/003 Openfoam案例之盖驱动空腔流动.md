---
title: 盖驱动空腔流动
excerpt: Openfoam案例之盖驱动空腔流动实现
published: false
date: "2024/03/17 21:30:25"
index_img: "https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230313170944.png"
category: 
- [模拟,Openfoam,案例]
tags:
- Openfoam
- Openfoam案例
---

官网：https://doc.cfd.direct/openfoam/user-guide-v9/cavity

# 一、算例实现
文件结构
1. 0：存放初场
2. constant：存放网格信息
3. system：存放网格划分、计算等工具

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230313165028.png)
## 1、画网格
```bash
blockMesh
```
![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230313165143.png)
## 2、求解
```bash
icoFoam
```
![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230313165414.png)
## 3、保存文件
```bash
touch cavity.OpenFOAM
```
## 4、后处理
```bash
paraview
```
使用openFoamReader打开
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328225952287-715025935.png)
显示所有边界
![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230313170944.png)
apply
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328230043559-470437306.png)
# 二、网格加密
## 1、网格划分
在icoFoam文件夹内创建cavityFine文件夹，复制cavity文件夹内的constant和system文件夹至该文件夹。
在system/blockMeshDict文件改变网格数量，由之前的（20,20,1）改为（40,40,1），之后使用blockMesh生成网格信息
![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202303271927561.png)
## 2、mapFields映射结果
将粗网格中的0.5结果，映射到细网格中。
**修改controlDict文件**
![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202303271934395.png)
**映射**
`mapFields ../cavity/cavity -consistent`

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202303271931951.png)

## 3、计算并输出log文件
```
// 计算并输出log文件
icoFoam > log
// 查看log文件
cat log

// 可以合并运行
icoFoam > log && cat log
```
## 4、calcType
可以对速度的某个量进行可视化处理并咬着计算域内的一条线来绘制一个二维的图
```bash
postProcess -func "components(U)"
```
此时会在时间步长文件夹下出现3个新文件
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328231000014-1030958555.png)

## 5、后处理
使用paraview加载数据，就会显示刚才计算出的值。
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328231102154-703276796.png)

# 三、网格非均匀分布
## 1、修改blockMeshDict
在icoFoam文件夹内创建asMeshCavity文件夹，复制cavity文件夹内的constant和system文件夹至该文件夹。
修改constant/blockMeshDict文件,如下
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328231422215-527026598.png)

```c++
/*--------------------------------*- C++ -*----------------------------------*\
 =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  9
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
FoamFile
{
    format      ascii;
    class       dictionary;
    object      blockMeshDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

convertToMeters 0.1;

vertices
(
    (0 0 0)
    (0.5 0 0)
    (1 0 0)
    (0 0.5 0)
    (0.5 0.5 0)
    (1 0.5 0)
    (0 1 0)
    (0.5 1 0)
    (1 1 0)
    (0 0 0.1)
    (0.5 0 0.1)
    (1 0 0.1)
    (0 0.5 0.1)
    (0.5 0.5 0.1)
    (1 0.5 0.1)
    (0 1 0.1)
    (0.5 1 0.1)
    (1 1 0.1)
);

blocks
(
    hex (0 1 4 3 9 10 13 12) (10 10 1) simpleGrading (2 2 1)
    hex (1 2 5 4 10 11 14 13) (10 10 1) simpleGrading (0.5 2 1)
    hex (3 4 7 6 12 13 16 15) (10 10 1) simpleGrading (2 0.5 1)
    hex (4 5 8 7 13 14 17 16) (10 10 1) simpleGrading (0.5 0.5 1)
);

edges
(
);

boundary
(
    movingWall
    {
        type wall;
        faces
        (
            (6 15 16 7)
            (7 16 17 8)
        );
    }
    fixedWalls
    {
        type wall;
        faces
        (
            (3 12 15 6)
            (0 9 12 3)
            (0 1 10 9)
            (1 2 11 10)
            (2 5 14 11)
            (5 8 17 14)
        );
    }
    frontAndBack
    {
        type empty;
        faces
        (
            (0 3 4 1)
            (1 4 5 2)
            (3 6 7 4)
            (4 7 8 5)
            (9 10 13 12)
            (10 11 14 13)
            (12 13 16 15)
            (13 14 17 16)
        );
    }
);

mergePatchPairs
(
);

// ************************************************************************* //
```
## 2、画网格
```bash
blockMesh
```
## 3、生成asyMeshCavity.OpenFOAM文件
```bash
touch asyMeshCavity.OpenFOAM
```
## 4、后处理
```bash
paraview
```
![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20230328224516.png)
## 5、修改时间步长
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328233542168-75131897.png)
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328233622443-1693050358.png)
## 6、映射场
将cavityFine中的0.7文件映射过来
```bash
mapFields -consistent ../cavityFine
```
## 7、计算输出
```bash
icoFoam > log
```
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328233936794-1729891992.png)
# 四、大雷诺数
## 1、创建文件夹并拷贝
创建文件夹
```bash
mkdir cavityHighRe
```
拷贝cavity文件夹至该文件夹
```bash
cp -r ../cavity/cavity/ .
```
## 2、修改雷诺数
修改constant/transportProperties文件
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328235441863-1741845960.png)
即可提高雷诺数10倍
## 3、修改控制文件
从0.5开始计算到2s
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328235605740-175867599.png)
## 4、计算
```bash
nohup nice -19 icoFoam > log && cat log
```
1. nohup：党用户退出登录时，程序依然执行；
2. nice：调整程序优先级，-20对应优先级最高，19对应优先级最低进程。

可以看出1.4s时Ux结束迭代，No Iterations 0表示速度求解停止。
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230328235924248-1170758731.png)
# 五、RAS模型
进入pisoFoam/RAS/cavity
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329090540394-753152053.png)

可以看到在0文件夹下有许多有关湍流模型的参数
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329090657244-428931759.png)
在momentumTransport文件里定义模型
```c++
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  9
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
FoamFile
{
    format      ascii;
    class       dictionary;
    location    "constant";
    object      momentumTransport;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

simulationType  RAS;

RAS
{
    model           kEpsilon;

    turbulence      on;

    printCoeffs     on;
}

// ************************************************************************* //
```
simulationType：laminar, RAS and LES
printCoeffs：为on时，这些参数会被输出到终端
# 六、CavityClipped
## 1、划分网格
```bash
blockMesh
```
## 2、复制0文件夹内容至0.5
```bash
cp -r 0 0.5
```
## 3、设置
controlDict文件开始时间设置为0.5，结束时间设置为0.6
**system/mapFieldsDict文件**
有两个参数
```c++
patchMap        (lid movingWall);

cuttingPatches  ();
```
1. 当用户向把原始场的patches条件投影到背投影场的patches的时候，我们使用patchMap。
2. 当用户打算把原始场内的数据投影到被投影场的边界时，我们使用cuttingPatches列表

这里lib边界条件根cavity算例的movingWall边界条件是一致的。
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329095907107-1887424104.png)

## 4、计算
```bash
icoFoam > log
```
## 5、后处理
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329095035023-786575892.png)
修改样式
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329101615572-2133524506.png)
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329101628978-47308774.png)
修改Legend
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329101656057-1319030686.png)
绘制矢量图
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329102702063-1162684766.png)
![image](https://img2023.cnblogs.com/blog/3059241/202303/3059241-20230329102802101-702785785.png)
