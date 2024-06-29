---
title: ICEM常遇问题
excerpt: 在使用ICEM划分网格时常遇到的问题
published: true
time: "2024/06/29 20:40:00"
index_img: https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/1719663517428.png
category: 
- [模拟,ICEM]
tags:
  - 模拟
  - ICEM
---

## 2D网格不闭合
> Mesh has uncovered edges. ANSYS Fluent needs a complete boundary (lines in 2D) or it will give a variety of errors and not read in the mesh! If this was 2D Hexa, perhaps your edges are not associated with perimeter curves

原因是线和线之间可能没有关联

![image.png](https://test-123456-md-images.oss-cn-beijing.aliyuncs.com/img/202302161539612.png)
