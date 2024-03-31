---
title: 带孔盘体应力分析
excerpt: Openfoam案例之带孔盘体应力分析
published: false
date: "2024/03/17 21:40:25"
index_img: "https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240317223441.png"
category: 
- [模拟,Openfoam,案例]
tags:
- Openfoam
- Openfoam案例
---

官网：https://doc.cfd.direct/openfoam/user-guide-v9/platehole

$FOAM_TUTORIALS/stressAnalysis/solidDisplacementFoam下的案例

# 1、网格划分

![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240317223441.png)


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

convertToMeters 1;

vertices
(
    (0.5 0 0)
    (1 0 0)
    (2 0 0)
    (2 0.707107 0)
    (0.707107 0.707107 0)
    (0.353553 0.353553 0)
    (2 2 0)
    (0.707107 2 0)
    (0 2 0)
    (0 1 0)
    (0 0.5 0)
    (0.5 0 0.5)
    (1 0 0.5)
    (2 0 0.5)
    (2 0.707107 0.5)
    (0.707107 0.707107 0.5)
    (0.353553 0.353553 0.5)
    (2 2 0.5)
    (0.707107 2 0.5)
    (0 2 0.5)
    (0 1 0.5)
    (0 0.5 0.5)
);

blocks
(
    hex (5 4 9 10 16 15 20 21) (10 10 1) simpleGrading (1 1 1)
    hex (0 1 4 5 11 12 15 16) (10 10 1) simpleGrading (1 1 1)
    hex (1 2 3 4 12 13 14 15) (20 10 1) simpleGrading (1 1 1)
    hex (4 3 6 7 15 14 17 18) (20 20 1) simpleGrading (1 1 1)
    hex (9 4 7 8 20 15 18 19) (10 20 1) simpleGrading (1 1 1)
);

edges
(
    arc 0 5 (0.469846 0.17101 0)
    arc 5 10 (0.17101 0.469846 0)
    arc 1 4 (0.939693 0.34202 0)
    arc 4 9 (0.34202 0.939693 0)
    arc 11 16 (0.469846 0.17101 0.5)
    arc 16 21 (0.17101 0.469846 0.5)
    arc 12 15 (0.939693 0.34202 0.5)
    arc 15 20 (0.34202 0.939693 0.5)
);

boundary
(
    left
    {
        type symmetryPlane; // 对称边界条件
        faces
        (
            (8 9 20 19)
            (9 10 21 20)
        );
    }
    right
    {
        type patch; // 对称边界条件
        faces
        (
            (2 3 14 13)
            (3 6 17 14)
        );
    }
    down
    {
        type symmetryPlane; // 对称边界条件
        faces
        (
            (0 1 12 11)
            (1 2 13 12)
        );
    }
    up
    {
        type patch;
        faces
        (
            (7 8 19 18)
            (6 7 18 17)
        );
    }
    hole
    {
        type patch;
        faces
        (
            (10 5 16 21)
            (5 0 11 16)
        );
    }
    frontAndBack
    {
        type empty; // 表示一个2D算例
        faces
        (
            (10 9 4 5)
            (5 4 1 0)
            (1 4 3 2)
            (4 7 6 3)
            (4 9 8 7)
            (21 16 15 20)
            (16 11 12 15)
            (12 13 14 15)
            (15 14 17 18)
            (15 18 19 20)
        );
    }
);

mergePatchPairs
(
);

// ************************************************************************* //
```
# 2、边界条件
## 2.1 位移量D
对于无热应力的单纯应力分析，只有位移量D需要指定。0/D
1. 关键词traction：指定牵引力边界。指定牵引力边界矢量和大小；
2. pressure：如果这个边界面法向牵引力的压力指向表面之外，就被定义为负值。
3. right边界牵引力为（10000 0 0）Pa，pressure为0
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
    class       volVectorField;
    object      D;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 1 0 0 0 0 0];

internalField   uniform (0 0 0);

boundaryField
{
    left
    {
        type            symmetryPlane;
    }
    right
    {
        type            tractionDisplacement;
        traction        uniform (10000 0 0); // 牵引力
        pressure        uniform 0;
        value           uniform (0 0 0);
    }
    down
    {
        type            symmetryPlane;
    }
    up
    {
        type            tractionDisplacement;
        traction        uniform (0 0 0);
        pressure        uniform 0;
        value           uniform (0 0 0);
    }
    hole
    {
        type            tractionDisplacement;
        traction        uniform (0 0 0);
        pressure        uniform 0;
        value           uniform (0 0 0);
    }
    frontAndBack
    {
        type            empty;
    }
}

// ************************************************************************* //
```
## 2.2 物理特性
constant/thermophysicalProperties
1. planeStress     yes; // 物理特性指定为yes
2. thermalStress   no; // 不求解热物理方程
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
    object      thermalProperties;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

rho // 密度
{
    type        uniform;
    value       7854;
}

nu // 泊松比
{
    type        uniform;
    value       0.3;
}

E // 杨氏模量
{
    type        uniform;
    value       2e+11;
}

Cp // 比热容
{
    type        uniform;
    value       434;
}

kappa // 热传导
{
    type        uniform;
    value       60.5;
}

alphav // 热膨胀系数
{
    type        uniform;
    value       1.1e-05;
}

planeStress     yes; // 物理特性指定为yes
thermalStress   no; // 不求解热物理方程


// ************************************************************************* //
```
## 2.3 控制
1. 如果求解器是SIMPLE算法，deltaT设定为多少是无关紧要的。
2. ？ timeFormat      general;
3. ？ timePrecision   6;
4. ？ graphFormat     raw;
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
    location    "system";
    object      controlDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

application     solidDisplacementFoam; // 求解器

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         100;

deltaT          1;

writeControl    timeStep; // 使用步长控制输出

writeInterval   20;

purgeWrite      0; // 文件数量上限，0表示不激活。设置为2只会输出最后2个

writeFormat     ascii; // 文件格式

writePrecision  6; // 文件的有效数字

writeCompression off; // 文件格式是否时压缩格式

timeFormat      general;

timePrecision   6;

graphFormat     raw;

runTimeModifiable true; // 运行中途修改配置文件是否有意义。如果true，则会读取修改文件


// ************************************************************************* //
```
## 2.4 离散格式和求解器
1. 稳态求解：timeScheme为SteadyState，用于屏蔽掉时间离散项；
2. 有限体积离散相建立与高斯定律上，对于大部分模拟高斯定律足够精准。但是在这个案例中没使用least squares；
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
    location    "system";
    object      fvSchemes;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

d2dt2Schemes
{
    default         steadyState;
}

ddtSchemes
{
    default         Euler;
}

gradSchemes
{
    default         leastSquares;
    grad(D)         leastSquares;
    grad(T)         leastSquares;
}

divSchemes
{
    default         none;
    div(sigmaD)     Gauss linear;
}

laplacianSchemes
{
    default         none;
    laplacian(DD,D) Gauss linear corrected;
    laplacian(kappa,T) Gauss linear corrected;
}

interpolationSchemes
{
    default         linear;
}

snGradSchemes
{
    default         none;
}

// ************************************************************************* //
```
fvSolution用于控制求解线性方程组使用的矩阵求解器
1. stressAnalysis：求解所需要的控制参数；
2. nCorrectors：整个方程组求解的外循环数，包括每个时间步长的拉伸边界条件；这里是*稳态*问题，时间步长代表迭代数以直到收敛，设置为1。
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
    location    "system";
    object      fvSolution;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

solvers
{
    "(D|T)"
    {
        solver          GAMG; // D的矩阵求解器GAMG
        tolerance       1e-06; // 容差
        relTol          0.9; // 相对误差，控制每次迭代的残差最小量
        smoother        GaussSeidel;
        nCellsInCoarsestLevel 20;
    }
}

stressAnalysis
{
    compactNormalStress yes;
    nCorrectors     1;
    D               1e-06;
}


// ************************************************************************* //
```

# 3、运行

```bash
solidDisplacementFoam > log && cat log
```

最终残差始终小于最初残差的0.9倍。

![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/highlight.png)

# 4、后处理

计算张量分量

```bash
postProcess -func "components(sigma)"
```

![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240317223841.png)

**对比解析解与数值解**

需要把计算域中的对称面的左边的求解出来，可以使用sample生成。需要调用system下的sampleDict字典。

```bash
/*--------------------------------*- C++ -*----------------------------------*\
  =========                 |
  \\      /  F ield         | OpenFOAM: The Open Source CFD Toolbox
   \\    /   O peration     | Website:  https://openfoam.org
    \\  /    A nd           | Version:  9
     \\/     M anipulation  |
\*---------------------------------------------------------------------------*/
interpolationScheme cellPoint;

setFormat	raw;

sets
(
 leftPatch
 {
	type	uniform;
	axis	y;
	start	(0 0.5 0.25);
	end	(0 2 0.25);
	nPoints	100;	
 }
);

fields	(sigmaxx);
```
生成解析解
```bash
postProcess -func graphUniform
```
![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/highligh%E8%BE%BE%E8%BE%BEt.png)

调用GnuPlot（需要安装）的plot命令

```bash
 plot [0.5:2] [0:] "postProcessing/graphUniform/100/line_sigmaxx.xy",
        1e4*(1+(0.125/(x**2))+(0.09375/(x**4)))
```
![](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240317223727.png)
