---
title: 梁友栋-Barsky裁剪算法
tags:
	- 裁剪
	- 软光栅
categories: 渲染
---
梁友栋-Barsky裁剪算法是参数化线段裁剪算法，基于参数t求线段与裁剪区域的交点。
## 一、算法思想
一条端点为P1(x1, y1)，P2(x2, y2)的线段参数方式可以表示如下：
```
x = x1 + t * (x2 - x1)
y = y1 + t * (y2 - y1)
```
令dx = x2 - x1，dy = y2 - y1,则上述方程变形为：
```
x = x1 + t * dx
y = y1 + t * dy
```
设裁剪区域由点(xmin, ymin)，(xmax, ymax)定义，设线段上的点P(x, y)位于裁剪区域内，则其满足以下不等式:
```
xmin <= x1 + t * dx <= xmax
ymin <= y1 + t * dy <= ymax
```
分离化简如下：
```
-dx * t <= (-xmin + x1)
dx * t <= (xmax - x1)
-dy * t <= (-ymin + y1)
dy * t <= (ymax - y1)
```
则四个不等式可以统称为:
```
pn * t <= qn，其中 n = 0, 1, 2, 3
```
通过画图，很容易得出以下结论：
```
平行于边界的线段满足pn = 0，若qn >= 0，则线段在裁剪区域内部，若qn < 0，则线段在裁剪区域外部。
当 pn < 0 时，线段从裁减边界的外部进入内部，
当 pn > 0 时，线段从裁剪边界的内部离开。
```

那么对于给定线段所在的直线，我们可以计算其与裁减区域的交点，设得到的交点参数为t1，t2，t1表示从外进入到内的参数，t2表示从内部离开到外部的参数，则
```
1、t1由直线从外进入裁减区域决定，此时 pn < 0，t >= qn / pn，所以t1是各个 qn/pn 中的最大值。
2、t2由直接从内部离开裁剪区域决定，此时 pn > 0， t <= qn / pn，所以t2是各个 qn/pn 中的最小值。
3、当 pn = 0 并且，qn < 0 时，线段在裁减区域外。
4、当 0 <= t1 <= t2 <= 1 时，线段与裁减区域有交点，将t1和t2带入方程可以得到被裁剪后的线段新端点为：
    (x1 + t1 * dx, y1 + t1 * dy)和(x1 + t2 * dx, y1 + t2 * dy)。
```
<!-- more --> 
## 二、算法实现
1、求出 pn，qn，其中 n = 0, 1, 2, 3。初始化线段交点参数t1 = 0， t2 = 1。

2、对四个不等式分别根据p，q来判断是舍弃线段，还是求得交点。
```
    当 p < 0 时，t1 = max{qn / pn}；
    当 p > 0 时，t2 = min{qn / pn}；
    当发现 t1 > t2 时线段与裁减区域无交点；
    当 p = 0 且 q < 0 时，线段与裁减区域无交点。
```
3、如果第二步没有拒绝交点测试，那么通过得出来的t1 和 t2求的交点即可。

附代码：
``` C++
bool RenderDevice::ClipLine( int& x1, int& y1, int& x2, int& y2 )
{
	auto ClipTest =[]( float p, float q, float& t1, float& t2 )
	{
		if ( p < 0.0f )
		{
			// 计算从外到内的t.
			float t = q / p;
			if ( t > t2 )
			{
				return false;
			}
			else if ( t > t1 )
			{
				t1 = t;
				return true;
			}
		}
		else if ( p > 0.0f )
		{
			// 计算从内到外的t.
			float t = q / p;
			if ( t < t1 )
			{
				return false;
			}
			else if ( t < t2 )
			{
				t2 = t;
				return true;
			}
			
		}
		else if ( q < 0.0f )
		{
			return false;
		}
		
		return true;
	};

	float dx = float( x2 - x1 );
	float dy = float( y2 - y1 );

	float parray[4];
	float qarray[4];
	parray[0] = -dx;
	parray[1] = dx;
	parray[2] = -dy;
	parray[3] = dy;
	qarray[0] = (float) x1;
	qarray[1] = float( mClipXMax - x1 );
	qarray[2] = (float) y1;
	qarray[3] = float( mClipYMax - y1 );
	float t1 = 0.0f;
	float t2 = 1.0f;
	for ( uint i = 0; i < 4; i ++ )
	{
		if ( ClipTest( parray[i], qarray[i], t1, t2 ) == false )
			return false;
	}
	
	int tx1 = x1, ty1 = y1, tx2 = x2, ty2 = y2;
	x1 = int( tx1 + t1 * dx );
	y1 = int( ty1 + t1 * dy );
	x2 = int( tx1 + t2 * dx );
	y2 = int( ty1 + t2 * dy );

	return true;
}
```