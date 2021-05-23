# Rate Control  Calibration
### 目的
建立控制输出率Control Rate 与称重流量 Flow 之间的关系。
<br/>
提供两种标定方法：
- 分布标定
- 快速标定
---
### 分布标定
- 20%控制率输出并维持10秒，然后记录当前流量，记为F20
- 依次类推，获得F40，F60，F80，F100
- 分布标定相对比较简单
---
### 快速标定
- T2项目新引入的快速标定方法
- 控制输出从20%以一定速率连续增加到100%(可以从10%开始，确保数据稳定)，测量整个过程从20%时刻点起到100%时刻的称重重量W(x)
- 快速标定相对比较复杂，下面具体说明快速标定的相关算法与处理过程

#### W(x)及其拟合
W(t)为离散的测量数据集，不妨将测量数据集中一个测量数据点记为：
$$(x_i,y_i), i = 1,2,...,n$$
其中:
- $x_i$ 为该数据点的控制输出率
- $y_i$ 为该数据点测量到的的称重重量值
- n 是快速标定测量过程收集的数据点数

一般的，用函数$p(x)=a_0+a_1x+a_2x^2+...+a_mx^m$作为近似曲线来拟合测试数据集，则其均方误差表示为：
$$
Q(a_0,a_1,a_2,\cdots,a_m) = \sum_{i=1}^n(a_0 + a_1x_i + a_2x_i^2 + \cdots + a_mx_i^m - y_i)^2
$$
那么，求最小均方误差得到
$$

\frac{\partial Q }{\partial{a_0}} =  \frac{\partial Q}{\partial a_1} = \cdots = \frac{\partial Q}{\partial a_m} = 0
$$

进而得到方程组：
$$
\left[ 
\begin{array}{c l l l}
n & \sum{x_i} & \cdots & \sum{x_i^m} \\
\sum{x_i} & \sum{x_i^2} & \cdots & \sum{x_i^{m+1}}\\
\vdots & \vdots & \ddots & \vdots \\
\sum{x_i^m} & \sum{x_i^{m+1}} & \cdots & \sum{x_i^{2m}}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
a_0\\
a_1\\
\vdots \\
a_m
\end{array}
\right ]
}
=
\left [
\begin{array}{l}
\sum{y_i}\\
\sum{x_iy_i}\\
\vdots\\
\sum{x_i^my_i}
\end{array}
\right ]
$$

上述法方程组是形式为$A*X=B$的矩阵方程，
其中 矩阵$A$和$B$ 是已知的，可以由测试点数据计算得到。

再用高斯消元法解出$X$，
即可得到系数 $\hat{a_0},\hat{a_1},\cdots,\hat{a_m}$。


#### 快速标定的数据计算
- T2项目中以三次函数: $p(x) = a_0 + a_1x + a_2x^2 + a_3x^3$ 来拟合快速标定测量数据集W(x),  并通过如下三次拟合法方程组求出三次拟合系数 $\hat{a_0},\hat{a_1},\hat{a_2}, \hat{a_3}$

$$
\left[ 
\begin{array}{c c c c}
n & \sum_{i=1}^nx_i & \sum_{i=1}^nx_i^2  & \sum_{i=1}^nx_i^3 \\
\sum_{i=1}^nx_i & \sum_{i=1}^nx_i^2 & \sum_{i=1}^nx_i^3  & \sum_{i=1}^nx_i^4 \\
\sum_{i=1}^nx_i^2 & \sum_{i=1}^nx_i^3 & \sum_{i=1}^nx_i^4  & \sum_{i=1}^nx_i^5 \\
\sum_{i=1}^nx_i^3 & \sum_{i=1}^nx_i^4 & \sum_{i=1}^nx_i^5  & \sum_{i=1}^nx_i^6
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
a_0 \\
a_1 \\
a_2 \\
a_3
\end{array}
\right ]
}
=
\left [
\begin{array}{l}
\sum_{i=1}^ny_i\\
\sum_{i=1}^nx_iy_i\\
\sum_{i=1}^nx_i^2y_i\\
\sum_{i=1}^nx_i^3y_i
\end{array}
\right ]
$$

- 由该三次拟合函数的导数，得到流量曲线: 
$$F(x) = a_1 + 2a_2x+3a_3x^2$$
- 取控制率输出$x$=20%、40%、60%、80%、100%时的流量$F(x)$即为所求

 

#### 快速标定的处理流程

@startuml
start
: 设置RC模式为快速标定模式;
: 启动RC;
: 不断调用快速标定处理过程;
if ( //快速标定已完成 \n rapidCalComplete) then (Y)
: return 0.0f;;
else (N)
: //计算当前测量的控制率 \n controlRate = (calRatio * calCurrentPos + PRE_START_RATE) / 100.0;;
: //得到当前测量的重量值 \n DC_Get(DI_wt0117, &rapidCalData[calCurrentPos]);;
if (当前测量数已达到合法数量) then (Y)
: //设置快速标定已完成 \n repidCalComplete = true;;
: 遍历测试数据集，计算三次拟合法方程组的矩阵A 和 矩阵B;
if ( 高斯消元法计算三次拟合系数成功 ) then (Y)
: 依据流量曲线F(x) 计算 F20, F40, F60, F80, F100;
if ( CheckCalibrationData() ) then (Y)
: VM_Cal_SD_Save();;
else (N)
: CF_Status |= ERR_CALIBRATION_FAILED; \n DA_Set(SMState, &CF_Status);;
endif
else (N)
: //出现错误 \n 停止处理;
endif
: return 0.0f;
else (N)
: //当前测量数加1  \n ++claCurrentPos;
: //返回本次测量的控制率 \n return controlRate;;
endif
endif

stop
@enduml


---
By GuJun， 2021-5-24
