---
title: Python调用REFPROP
excerpt: 将REFPROP封装为python库方便调用物性参数
published: true
time: "2024/06/29 20:30:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/1719663075628.png
category: 
- [编程,Python]
tags:
  - 编程
  - Python
  - REFPROP
---

# python调用REFPROP

## 1、安装调试

1. 安装refprop10
2. 安装python库`pip install ctREFPROP`

测试使用

```python
import os, numpy as np
from ctREFPROP.ctREFPROP import REFPROPFunctionLibrary

def NBP():
    RP = REFPROPFunctionLibrary(os.environ['RPPREFIX'])
    RP.SETPATHdll(os.environ['RPPREFIX'])
    print(RP.RPVersion())
    MOLAR_BASE_SI = RP.GETENUMdll(0,"MOLAR BASE SI").iEnum

    r = RP.REFPROPdll("PROPANE","PQ","T",MOLAR_BASE_SI, 0,0,101325, 0, [1.0])
    print(r.ierr, r.herr, r.Output[0])

if __name__=='__main__':
    # 路径
    os.environ['RPPREFIX'] = r'D:/REFPROP10'
    
    NBP()
    
>> 
10.0
0  231.0362479100902
```

## 2、基本语法

```python
import os, numpy as np
from ctREFPROP.ctREFPROP import REFPROPFunctionLibrary

def calculate_properties():
    # 初始化REFPROP
    RP = REFPROPFunctionLibrary(os.environ['RPPREFIX'])
    RP.SETPATHdll(os.environ['RPPREFIX'])

    # 将单位设置为国际单位制（摩尔基准）
    MOLAR_BASE_SI = RP.GETENUMdll(0, "MASS BASE SI").iEnum

    # 指定物质（氢）、计算类型（H为焓值，D为密度，S为熵值）
    substance = "HYDROGEN"
    
    # 指定输入条件（温度、压力）
    temperature = 298.15  # 单位：K
    pressure = 101325  # 单位：Pa

    # 调用REFPROPdll计算性质
    H_result = RP.REFPROPdll(substance, "PT","H",  MOLAR_BASE_SI, 0, 0, pressure, temperature, [1.0]) # J/kg
    D_result = RP.REFPROPdll(substance, "PT","D",  MOLAR_BASE_SI, 0, 0, pressure, temperature, [1.0]) # kg/m3
    S_result = RP.REFPROPdll(substance, "PT","S",  MOLAR_BASE_SI, 0, 0, pressure, temperature, [1.0]) # J/(kg·K)
    # 检查是否有错误
    if H_result.ierr == 0 and D_result.ierr == 0 and S_result.ierr == 0:
        print("计算得到的H=:",H_result.Output[0])
        print("计算得到的D=:",D_result.Output[0])
        print("计算得到的S=:",S_result.Output[0])
    else:
        print(f"错误: {H_result.herr}")

if __name__=='__main__':
    # 路径
    os.environ['RPPREFIX'] = r'D:/REFPROP10'
    
    calculate_properties()
```

