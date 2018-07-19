---
title: Gamma校正
categories: 渲染
---

一、Gamma校正的由来
- 人眼对亮度感受的非线性， 类似于一个pow曲线，对暗部的敏感度强一些。图像中的空间有限，没必要在图像中给亮部太多空间。
- CRT显示器，阴极管发射出来的光子的亮度非线性。
二、如何进行Gamma校正
- encoding gamma、display gamma
- 非线性->线性->非线性->Display
