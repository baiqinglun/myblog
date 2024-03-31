---
title: 圆柱绕流
excerpt: Openfoam案例之圆柱绕流实现
published: false
date: "2024/03/17 20:46:25"
index_img: "https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102126758.png"
category: 
- [模拟,Openfoam,案例]
tags:
- Openfoam
- Openfoam案例
---

# 1、原视频地址

https://www.bilibili.com/video/BV1ME411A73k/?spm_id_from=333.1007.top_right_bar_window_custom_collection.content.click&vd_source=33b50a4dd201d7564e6e63d321809ce9

# 2、网格划分及导入

## 2.1 网格划分

本案例使用ICEM划分网格，并导入openfoam中
![image](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102041957.png)

## 2.2 网格转换

> 目前通过在 3 维中定义网格来处理 2 维几何，其中前平面和后平面定义为空边界块类型。读取二维 Fluent 网格时，转换器会自动在第三方向拉伸网格并添加空面片，将其命名为 frontAndBackPlanes。

`fluentMeshToFoam`：读取fluent.msh网格文件。（指南的5.5章）

命令：

```bash
fluentMeshToFoam <meshFile>
```

![image-20230410205215587](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102052633.png)

## 2.3 检查网格

```bash
checkMesh
```

![image-20230410205306166](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102053209.png)

> system/blockMeshDict文件可以删除

# 3、边界条件

## 3.1 网格boundary

这是转换网格后自动生成的。

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
    class       polyBoundaryMesh;
    location    "constant/polyMesh";
    object      boundary;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

5
(
    INLET
    {
        type            patch;
        nFaces          30;
        startFace       13060;
    }
    OUTLET
    {
        type            patch;
        nFaces          30;
        startFace       13090;
    }
    WALL
    {
        type            wall;
        inGroups        List<word> 1(wall);
        nFaces          100;
        startFace       13120;
    }
    CYLINDER
    {
        type            wall;
        inGroups        List<word> 1(wall);
        nFaces          120;
        startFace       13220;
    }
    frontAndBackPlanes // 空，表示2维
    {
        type            empty;
        inGroups        List<word> 1(empty);
        nFaces          13200;
        startFace       13340;
    }
)

// ************************************************************************* //
```

## 3.2 0/P文件

1. 在OpenFOAM中，zeroGradient是一种边界条件，用于指定在边界上的量的梯度为零，这意味着该**边界的值不会改变**。例如，对于速度边界条件，zeroGradient表示速度沿该边界的梯度为零，即速度沿该边界的值不会改变。通常，这种边界条件在流体边界和对称边界上使用。在代码中，它表示为：![image-20230410210614033](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102106071.png)。
2. 在 OpenFOAM 中，`fixedValue` 是一种边界条件类型，表示在特定的边界上施加固定的值。这个值通常是在 `0`、`uniform`、`nonuniform` 或 `list` 中指定的，具体取决于场的类型和边界的特征。例如，在一个流体模拟中，可以将固定的速度值施加在流场的边界上，这可以通过设置边界条件类型为 `fixedValue` 并指定速度值来实现。同样地，可以将固定的温度、压力或其他物理量值应用于相应的场变量上。在 OpenFOAM 中，边界条件的设置是在 case 目录下的 `0` 文件夹中的 `boundary` 文件中完成的。使用 `fixedValue` 边界条件时，用户需要指定固定的值并将其应用于适当的边界。
3. 在OpenFOAM中，empty边界类型用于定义**没有物理边界**或者**该边界不需要与周围区域相互作用**的情况。例如，计算空气在室内流动的情况，室内墙壁可以定义为empty边界。在empty边界中，OpenFOAM不会对物理量进行任何修改，因此需要在边界条件的设置中进行额外的注意。在empty边界中，通常会使用fixedValue或者fixedGradient来设置边界条件。

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
    class       volScalarField;
    object      p;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 2 -2 0 0 0 0];

internalField   uniform 0;

boundaryField
{
    INLET // 压力梯度为0
    {
        type            zeroGradient; // 压力梯度为0
    }
    
    WALL
    {
        type            zeroGradient;
    }

    CYLINDER
    {
        type            zeroGradient;
    }
    
    OUTLET // 出口压力为固定值0
    {
        type            fixedValue; 
        value           uniform 0;
    }

    frontAndBack
    {
        type            empty;
    }
}

// ************************************************************************* //
```

## 3.3 0/U文件

1. **internalField**是指某个场量在整个计算域内的初始值。在进行流体流动计算时，internalField可以表示压力、速度、温度等场量的初始值。internalField可以被定义为标量、向量、对称张量和非对称张量场量。常见的初始化方法包括常数、分布式初始化和从其他场量插值得到。例如，对于速度场量，可以通过设置其为一个初始速度场量或者通过从初始条件的压力梯度中计算得到。一旦internalField被定义，将无法在后续的计算过程中修改。因此，需要根据计算需求仔细地定义initialField。

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
    object      U;
}
// * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * //

dimensions      [0 1 -1 0 0 0 0];

// 场内初始值
internalField   uniform (50 0 0);

boundaryField
{
    INLET // 速度为固定值ux=50m/s
    {
        type            fixedValue; 
        value           uniform (50 0 0);
    }
    
    WALL
    {
        type            fixedValue;
        value           uniform (50 0 0);
    }
    
    OUTLET
    {
        type            zeroGradient;
    }

    CYLINDER
    {
        type            fixedValue;
        value           uniform (0 0 0);
    }

    frontAndBack
    {
        type            empty;
    }
}

// ************************************************************************* //
```

> oepnfoam中zeroGradient和uniform (0 0 0)的区别：
>
> zeroGradient只表示在边界上梯度不变，并不是不随时间变化。
>
> uniform (0 0 0)更强硬一些，就是为某个值。

# 4、计算参数

## 4.1 controlDict

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

application     icoFoam;

startFrom       startTime;

startTime       0;

stopAt          endTime;

endTime         0.05;

deltaT          0.00001; // Co = U * t / x

writeControl    timeStep;

writeInterval   100;

purgeWrite      0;

writeFormat     ascii;

writePrecision  6;

writeCompression off;

timeFormat      general;

timePrecision   6;

runTimeModifiable true;


// ************************************************************************* //
```

## 4.2 fvSchemes

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
    grad(p)         Gauss linear;
}

divSchemes
{
    default         none;
    div(phi,U)      Gauss linear;
}

laplacianSchemes
{
    default         Gauss linear orthogonal;
}

interpolationSchemes
{
    default         linear;
}

snGradSchemes
{
    default         orthogonal;
}


// ************************************************************************* //
```

## 4.3 fvSolution

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
    p
    {
        solver          PCG;
        preconditioner  DIC;
        tolerance       1e-06;
        relTol          0.05;
    }

    pFinal
    {
        $p;
        relTol          0;
    }

    U
    {
        solver          smoothSolver;
        smoother        symGaussSeidel;
        tolerance       1e-05;
        relTol          0;
    }
}

PISO
{
    nCorrectors     2;
    nNonOrthogonalCorrectors 0;
    pRefCell        0;
    pRefValue       0;
}


// ************************************************************************* //
```

# 5、计算

```bash
icoFoam > log
```



![image-20230410212245907](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102122157.png)



# 6、后处理

生成临时后处理文件

```bash
paraFoam -builtin
```

![image-20230410212654657](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202304102126758.png)
