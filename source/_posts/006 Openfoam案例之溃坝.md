---
title: 溃坝
excerpt: Openfoam案例之溃坝模拟实现
published: false
date: "2024/03/17 21:45:25"
index_img: "https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011707089.png"
category: 
- [模拟,Openfoam,案例]
tags:
- Openfoam
- Openfoam案例
---

[官网](https://doc.cfd.direct/openfoam/user-guide-v9/dambreak)

目录：$FOAM_TUTORIALS/multiphase/interFoam/laminar/damBreak

# 1、介绍

本案例使用interFoam两相算法，基于流体体积分数（VOF）法，每个网格中的**相体积分数**（alpha）通过求解一个**组分运输方程**确定。**物理属性**基于这个相分数通过**加权平均**计算。

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011615369.png)

# 2、网格生成

```bash
blockMesh
```

# 3、边界条件

最顶端atmosphere边界设置为patch
```c++
// 0/U
boundaryField
{
	...
    atmosphere
    {
        type            pressureInletOutletVelocity; // 对所有分量应用zeroGradient条件，当流动为入流时，对边界切向的分量应用fixedValue；
        value           uniform (0 0 0);
    }
    defaultFaces
    {
        type            empty;
    }
}

// 0/p_rgh
boundaryField
{
	...
    atmosphere
    {
        type            totalPressure; // 一种fixedValue条件，利用指定的总压p0和当地速度U计算获得；
        p0              uniform 0;
    }

    defaultFaces
    {
        type            empty;
    }
}

// 0/alpha.water.orig
boundaryField
{
	...
    atmosphere
    {
        type            inletOutlet; // 出流时为zeroGradient，入流时则为fixedValue条件；
        inletValue      uniform 0;
        value           uniform 0;
    }

    defaultFaces
    {
        type            empty;
    }
}
// ************************************************************************* //
```

所有壁面边界处，压力场采用**fixedFluxPressure**边界条件，它**自动调整压力梯度使边界通量符合速度边界条件**。

```c++
boundaryField
{
    leftWall
    {
        type            fixedFluxPressure; // 
        value           uniform 0;
    }

    rightWall
    {
        type            fixedFluxPressure;
        value           uniform 0;
    }

    lowerWall
    {
        type            fixedFluxPressure;
        value           uniform 0;
    }
}
```

defaultFaces代表此2维问题的前后面，像往常一样，是empty类型

```c++
    defaultFaces
    {
        type            empty;
    }
```

# 4、设置初场

相当于fluent中的patch，patch出一个水域。
boxToCell通过定义一个最小和最大的向量来创建一个盒子区域，在此区域内为水相，water被设置为1.
在执行```setFields```之前，先备份0目录下的alpha.water.org（备份文件）为alpha.water。因为setFields工具从文件读取场并重新定义这些场，然后把它们重新写入文件，原始文件会被覆盖，所以需要备份。

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
    object      setFieldsDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

defaultFieldValues
(
    volScalarFieldValue alpha.water 0 // 场内默认为空气
);

regions
(
    boxToCell
    {
        box (0 0 -1) (0.1461 0.292 1); // 对应坐标如下
        fieldValues
        (
            volScalarFieldValue alpha.water 1
        );
    }
);
// ************************************************************************* //
```

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011637015.png)

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011642227.png)

> 使用`paraFoam -builtin`后处理查看

# 5、流体特性

constant/transportProperties

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
    object      transportProperties;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

phases (water air); // 两相

water
{
    transportModel  Newtonian; // 牛顿流体
    nu              1e-06; // 运动粘度
    rho             1000; // 密度
}

air
{
    transportModel  Newtonian;
    nu              1.48e-05;
    rho             1;
}

sigma            0.07; // 表面张力

// ************************************************************************* //
```

> Newtonian为牛顿流体，CrossPowerLawCoeffs为非牛顿流体。

**重力场**

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
    class       uniformDimensionedVectorField;
    location    "constant";
    object      g;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 1 -2 0 0 0 0];
value           (0 -9.81 0);


// ************************************************************************* //
```

# 6、湍流模型

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

simulationType  laminar;  // 层流


// ************************************************************************* //
```

# 7、时间步长

设置adjustTimeStep为on，表示自动调整时间步长，并设置最大库郎书和最大相场库郎数为1.

设置writeContral为adjustableRunTime，表示强制调整一时间点正好为输出结果的时间。因为自动调整时间步长之后，如果按照固定时间步长间隔保存结果会比较乱，所以需要强制调整。

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

application     interFoam;

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         1;

deltaT          0.001;

writeControl    adjustableRunTime; // 重点

writeInterval   0.05;

purgeWrite      0;

writeFormat     binary;

writePrecision  6;

writeCompression off;

timeFormat      general;

timePrecision   6;

runTimeModifiable yes;

adjustTimeStep  yes; // 重点

maxCo           1; // 重点
maxAlphaCo      1; // 重点

maxDeltaT       1;


// ************************************************************************* //
```

# 8、离散求解

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

ddtSchemes
{
    default         Euler;
}

gradSchemes
{
    default         Gauss linear;
}

divSchemes
{
    div(rhoPhi,U)  Gauss linearUpwind grad(U);
    div(phi,alpha)  Gauss interfaceCompression vanLeer 1;
    div(((rho*nuEff)*dev2(T(grad(U))))) Gauss linear;
}

laplacianSchemes
{
    default         Gauss linear corrected;
}

interpolationSchemes
{
    default         linear;
}

snGradSchemes
{
    default         corrected;
}


// ************************************************************************* //
```

# 9、矩阵求解

nAlphaSubCycles：a方程中子循环的数目，子循环是一个给定时间步内对一个方程附加求解。可以再不降低时间步长的情况下保证解的稳定性。

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
    "alpha.water.*"
    {
        nAlphaCorr      2;
        nAlphaSubCycles 1;

        MULESCorr       yes;
        nLimiterIter    5;

        solver          smoothSolver;
        smoother        symGaussSeidel;
        tolerance       1e-8;
        relTol          0;
    }

    "pcorr.*"
    {
        solver          PCG;
        preconditioner  DIC;
        tolerance       1e-5;
        relTol          0;
    }

    p_rgh
    {
        solver          PCG;
        preconditioner  DIC;
        tolerance       1e-07;
        relTol          0.05;
    }

    p_rghFinal
    {
        $p_rgh;
        relTol          0;
    }

    U
    {
        solver          smoothSolver;
        smoother        symGaussSeidel;
        tolerance       1e-06;
        relTol          0;
    }
}

PIMPLE
{
    momentumPredictor   no;
    nOuterCorrectors    1;
    nCorrectors         3;
    nNonOrthogonalCorrectors 0;
}

relaxationFactors
{
    equations
    {
        ".*" 1;
    }
}


// ************************************************************************* //
```

# 10、计算

```bash
interFoam | tee log
```

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011658668.png)

# 11、并行计算

如果采用并行计算，程序根据system/decomposeParDict求解.
numberOfSubdomains：指定算例分割子区域数量，即处理器数
simpleCoeffs中，应满足x * y = numberOfSubdomains
这个算例使用2个处理器并行计算。
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
    object      decomposeParDict;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

numberOfSubdomains 2; // 重点

method          simple;

simpleCoeffs
{
    n               (1 2 1); // 重点
}

hierarchicalCoeffs
{
    n               (1 1 1);
    order           xyz;
}

manualCoeffs
{
    dataFile        "";
}

distributed     no;

roots           ( );


// ************************************************************************* //
```

> 如果使用4个，我这里使用的虚拟机，会报错，提示核数不够。

分割

```bash
decomposePar
```

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011702033.png)

并行计算

```c++
mpirun -np 2 interFoam -parallel > log
```

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011704221.png)

查看paraFoam -builtin -case processor0，为一半

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011705803.png)

合并结果

```bash
reconstructPar
```

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011706803.png)

后处理

```bash
paraFoam -builtin
```

![800](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304011707089.png)
