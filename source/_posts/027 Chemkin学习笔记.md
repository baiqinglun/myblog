---
title: Chemkin学习笔记
excerpt: Chemkin学习笔记
published: false
# sticky: 100
time: "2024/06/29 20:43:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/1719664231327.png
category: 
- [模拟,Chemkin]
tags:
  - 学习笔记
  - Chemkin
  - 模拟
---

功能：

1. 均质着火，一维火焰传播，对冲火焰
2. 敏感性分析，反应路径分析
3. 机理简化（DRG）等

机理文件下载

1. http://combustion.berkeley.edu/gri-mech/version30/text30.html#thefiles
2. https://www.universityofgalway.ie/combustionchemistrycentre/mechanismdownloads/

Chemkin包括的内容

![image.png|500](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/20240520212844.png)

## 氢燃烧案例



## 机理文件

### 一、第三体效应

```
H2+M=2H+M                                               4.5770e+19   -1.400    104400.00
H2/ 2.50/ H2O/ 12.00/ CO/ 1.90/ CO2/ 3.80/ HE/ 0.83/ CH4/ 2.00/ C2H6/ 3.00/ 
```

反应：

$$H_2+M→2H+M$$

速率常数表达式：

速率常数 k(T)k(T)k(T) 的表达式通常是根据阿伦尼乌斯方程来描述的，即： k(T)=A⋅Tn⋅e−EaRTk(T) = A \cdot T^n \cdot e^{-\frac{E_a}{RT}}k(T)=A⋅Tn⋅e−RTEa​​

参数解释：

- **A**：前指数因子，表示频率因子，单位为 cm3mol−1s−1\text{cm}^3 \text{mol}^{-1} \text{s}^{-1}cm3mol−1s−1。
- **n**：温度指数。
- **E_a**：活化能，单位为 cal/mol\text{cal/mol}cal/mol。

具体数值为：

- **A** = 4.5770×10194.5770 \times 10^{19}4.5770×1019
- **n** = -1.400
- **E_a** = 104400.00 cal/mol

媒体效应（第三体效应）：

反应涉及一个第三体 MMM，表示反应需要第三体的参与以维持能量守恒和动量守恒。第三体 MMM 可以是多种不同的气体分子，其对反应速率的贡献不同。不同气体的第三体效应系数如下：

- **H2**：2.50
- **H2O**：12.00
- **CO**：1.90
- **CO2**：3.80
- **HE**：0.83
- **CH4**：2.00
- **C2H6**：3.00

这些系数表明在该反应中，不同气体作为第三体 MMM 时的效率。数值越大，第三体的效应越强。例如，水（H2O）的效应是12.00，表示其作为第三体的效率很高，而氦气（He）的效应是0.83，表示其效率较低。