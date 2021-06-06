# Rate Control  Calibration
### 目的
建立控制输出率Control Rate 与称重流量 Flow 之间的关系。
<br/>
提供两种标定方法：
- 分布标定
- 快速标定
---

### 分步标定
- 20%控制率输出并维持10秒，然后记录当前流量，记为F20
- 依次类推，获得F40，F60，F80，F100
- 分步标定相对比较简单
---

### 快速标定
- T2项目新引入的快速标定方法
- 控制输出从10%以一定速率连续增加到100%，测量整个过程从10%时刻点起到100%时刻的称重重量W(t)
- 快速标定相对比较复杂，下面具体说明快速标定的相关算法与处理过程


#### W(t)的拟合
W(t)为离散的标定测量数据集，不妨将标定测量数据集中一个测量数据点记为：
$$(x_i,y_i), x_i = 1,2,...,n$$
其中:
- $x_i$ 为该标定测量数据点的时间序列值，时间序列值是一个相对标定开始时刻后的规格化时间值，软件实现时以整数表示，时间序列值增加1表示真实时间过了一个标定数据点检测周期
- $y_i$ 为该标定测量数据的的称重重量值
- $n$为快速标定测量过程总测量点数


一般的，用函数$p(x)=a_0+a_1x+a_2x^2+...+a_mx^m$作为近似曲线来拟合测试数据集W(t)，则其均方误差表示为：
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

再用高斯列主元消元法解出$X$，
即可得到系数 $\hat{a_0},\hat{a_1},\cdots,\hat{a_m}$。

#### 时间序列与控制率输出的对应关系
- 快速标定测量过程中，控制率输出值$r$将会从初始值 10% 逐渐增加到结束值 100%，时间序列值越大，控制率输出值越大，每次快速标定测量相比上一次测量，控制率增量为：$\Delta{r} = { { 1 - 0.1 } \over { n - 1 } }$
- 在任意测量点，控制率输出值$r_i$与时间序列值$x_i$的线性关系为: 
$$
r_i = \Delta{r} \times x_i + 0.1
$$
或者
$$
\tag{1} x_i = { { r_i - 0.1 } \over { \Delta{r} } }
$$
- 对于一次快速标定测量过程，$n$与$\Delta{r}$是在初始化阶段确定下来的常量


#### 快速标定的数据计算
- T2项目中以三次函数: $p(x) = a_0 + a_1x + a_2x^2 + a_3x^3$ 来拟合快速标定测量数据集W(t),  并通过如下三次拟合法方程组
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
求出三次拟合系数 $\hat{a_0},\hat{a_1},\hat{a_2}, \hat{a_3}$

- 由该三次拟合函数的导数，得到流量$F$随时间序列$x$变化的曲线: 
$$
\tag{2} F(x) = \hat{a_1} + 2\hat{a_2}x+3\hat{a_3}x^2
$$
- 由此，当控制率输出$r$=20%时, 可以用式（1）求得对应时间序列值$x_{r20} = { { 0.2 - 0.1 } \over { \Delta{r} } }$, 进而用式（2）求出对应的流量值F20 = $F(x_{r20})$
- 同理，可以求得控制率输出$r$=40%、60%、80%、100%时对应时间序列值$x_{r40}$、$x_{r60}$、$x_{r80}$、$x_{r100}$， 进而求出对应的流量值F40 = $F(x_{r40})$、F60 = $F(x_{r60})$、F80 = $F(x_{r80})$、F100 = $F(x_{r100})$。

---

#### 快速标定相关参数
| Name | VaIndex | Description |
| -- | -- | --  |
| CalibrationTime | ASI15 - 10836 | 一次快速标定的时间跨度，单位：秒，DC中默认值是0，需要应用初始化过程设置 |
| calValidDateNum | 内存全局变量 | 一次快速标定的有效测量点数量，可由CalibrationTime * ANALOG_OUTPUT_RATE 计算得到， ANALOG_OUTPUT_RATE目前取值20，即1秒钟20Hz输出频率 |
 

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
if ( 高斯列主元消元法计算三次拟合系数成功 ) then (Y)
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


#### 高斯列主元消元法说明
高斯列主元消元法的理论证明请参阅相关书籍，这里仅示例说明用高斯消元法 解$A*X=B$矩阵方程的具体步骤，以便代码实现或代码阅读。
<br/>
现阶段，在实际RC快速标定应用中，A是含4*4元素的矩阵，不妨假该矩阵方程为如下形式：
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} & a_{1_1} & a_{2_1} & a_{3_1} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3} \\
a_{0_4} & a_{1_4} & a_{2_4} & a_{3_4}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 \\
b_1 \\
b_2 \\
b_3
\end{array}
\right ]
$$

用高斯列主元消元法对$A*X=B$矩阵方程进行变换，目标是变换成如下形式：
$$
\left[ 
\begin{array}{c c c c}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\hat{b_0} \\
\hat{b_1} \\
\hat{b_2} \\
\hat{b_3}
\end{array}
\right ]
$$

这样，方程的解就是$\hat{B}$, 即$x_0 = \hat{b_0}, x_1 = \hat{b_1},x_2 = \hat{b_2},x_3 = \hat{b_3}$，形成该变换的实施步骤如下。

##### 寻找最大元
遍历A矩阵所有4*4=16个元素，找到其中绝对值最大的元素，便于示例说明，不妨假设该最大元是$a_{2_4}$:
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} & a_{1_1} & a_{2_1} & a_{3_1} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3} \\
a_{0_4} & a_{1_4} & a_{Big} & a_{3_4}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 \\
b_1 \\
b_2 \\
b_3
\end{array}
\right ]
$$

##### 位置变换使该最大元成为对角元
若该最大元不是该列中的对角元，则调换 最大元所在行 与 对角元所在行 的位置， 位置变换示例如下：
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} & a_{1_1} & a_{2_1} & a_{3_1} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
a_{0_4} & a_{1_4} & a_{Big} & a_{3_4} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 \\
b_1 \\
b_3 \\
b_2
\end{array}
\right ]
$$

##### 数值变换该对角元值为1
若该对角元值为0，则报错并退出；
若该对角元值不是1，则使该对角元所在行的各元素值 乘以 该对角元值的倒数，数值变换示例如下：
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} & a_{1_1} & a_{2_1} & a_{3_1} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
a_{0_4} \times { 1 \over{a_{Big}} } & a_{1_4} \times { 1 \over{a_{Big}} }  & a_{Big} \times { 1 \over{a_{Big}} } & a_{3_4} \times { 1 \over{a_{Big}} }  \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 \\
b_1 \\
b_3 \times { 1 \over{a_{Big}} } \\
b_2
\end{array}
\right ]
$$
不妨将上述数值变换结果表示为如下形式：
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} & a_{1_1} & a_{2_1} & a_{3_1} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
\dot{a_{0_4}} & \dot{a_{1_4}}  & 1 & \dot{a_{3_4}} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 \\
b_1 \\
\dot{b_3} \\
b_2
\end{array}
\right ]
$$

##### 该列其他元素数值变换为0
若该对角元所在列其他元素不为0，则将该元素所在行 减去 （该对角元所在行 乘以 该元素值），比如$a_{2_1}$不为零, 则数值变换示例如下：
$$
\left[ 
\begin{array}{c c c c}
a_{0_1} - \dot{a_{0_4}} \times {a_{2_1}} & a_{1_1} - \dot{a_{1_4}} \times {a_{2_1}} & a_{2_1} - a_{2_1} & a_{3_1} - \dot{a_{3_4}} \times {a_{2_1}} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
\dot{a_{0_4}} & \dot{a_{1_4}}  & 1 & \dot{a_{3_4}} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
b_0 - \dot{b_3} \times {a_{2_1}} \\
b_1 \\
\dot{b_3} \\
b_2
\end{array}
\right ]
$$

不妨将上述数值变换结果表示为如下形式：
$$
\left[ 
\begin{array}{c c c c}
\dot{a_{0_1}} & \dot{a_{1_1}} & 0 & \dot{a_{3_1}} \\
a_{0_2} & a_{1_2} & a_{2_2} & a_{3_2} \\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
a_{0_3} & a_{1_3} & a_{2_3} & a_{3_3}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
b_1 \\
\dot{b_3} \\
b_2
\end{array}
\right ]
$$

同理，$a_{2_2}$或$a_{2_3}$不为零, 则可以进行相似数值变换， 不妨将变换结果示例如下：
$$
\left[ 
\begin{array}{c c c c}
\dot{a_{0_1}} & \dot{a_{1_1}} & 0 & \dot{a_{3_1}} \\
\dot{a_{0_2}} & \dot{a_{1_2}} & 0 & \dot{a_{3_2}} \\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
\dot{a_{0_3}} & \dot{a_{1_3}} & 0 & \dot{a_{3_3}}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
\dot{b_1} \\
\dot{b_3} \\
\dot{b_2}
\end{array}
\right ]
$$

##### 对另一列进行同样的变换处理
上述示例中，第3列对角元已经变换为1，同理其他列也可以进行与上面相似的寻找最大元、位置变换、数值变换步骤，使其他列的对角元也变换为1、其他元为0。
<br/>
例如在示例中，接下来将在非第3行和非第3列的元素中（红色字体元素）中遍历寻找最大元：
$$
\left[ 
\begin{array}{c c c c}
{\color{Red} \dot{a_{0_1}} } & {\color{Red} \dot{a_{1_1}} } & 0 & {\color{Red} \dot{a_{3_1}} } \\
{\color{Red} \dot{a_{0_2}} } & {\color{Red} \dot{a_{1_2}} } & 0 & {\color{Red} \dot{a_{3_2}} }\\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
{\color{Red} \dot{a_{0_3}} } & {\color{Red} \dot{a_{1_3}} } & 0 & {\color{Red} \dot{a_{3_3}} }
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
\dot{b_1} \\
\dot{b_3} \\
\dot{b_2}
\end{array}
\right ]
$$

不妨假设$\dot{a_{1_3}}$为其中的最大元：
$$
\left[ 
\begin{array}{c c c c}
{\color{Red} \dot{a_{0_1}} } & {\color{Red} \dot{a_{1_1}} } & 0 & {\color{Red} \dot{a_{3_1}} } \\
{\color{Red} \dot{a_{0_2}} } & {\color{Red} \dot{a_{1_2}} } & 0 & {\color{Red} \dot{a_{3_2}} }\\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
{\color{Red} \dot{a_{0_3}} } & {\color{Red} \dot{a_{Big}} } & 0 & {\color{Red} \dot{a_{3_3}} }
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
\dot{b_1} \\
\dot{b_3} \\
\dot{b_2}
\end{array}
\right ]
$$

位置变换(调换最大元所在行与对角元所在行的位置) 使该最大元成为对角元：
$$
\left[ 
\begin{array}{c c c c}
{\color{Red} \dot{a_{0_1}} } & {\color{Red} \dot{a_{1_1}} } & 0 & {\color{Red} \dot{a_{3_1}} } \\
{\color{Red} \dot{a_{0_3}} } & {\color{Red} \dot{a_{Big}} } & 0 & {\color{Red} \dot{a_{3_3}} } \\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
{\color{Red} \dot{a_{0_2}} } & {\color{Red} \dot{a_{1_2}} } & 0 & {\color{Red} \dot{a_{3_2}} }
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
\dot{b_2} \\
\dot{b_3} \\
\dot{b_1}
\end{array}
\right ]
$$

同理，数值变换该对角元值为1:
$$
\left[ 
\begin{array}{c c c c}
\dot{a_{0_1}} & \dot{a_{1_1}} & 0 & \dot{a_{3_1}} \\
\ddot{a_{0_3}} & 1 & 0 & \ddot{a_{3_3}} \\
\dot{a_{0_4}} & \dot{a_{1_4}} & 1 & \dot{a_{3_4}} \\
\dot{a_{0_2}} & \dot{a_{1_2}} & 0 & \dot{a_{3_2}}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\dot{b_0} \\
\ddot{b_2} \\
\dot{b_3} \\
\dot{b_1}
\end{array}
\right ]
$$

同理，变换该列其他元素数值为0：
$$
\left[ 
\begin{array}{c c c c}
\ddot{a_{0_1}} & 0 & 0 & \ddot{a_{3_1}} \\
\ddot{a_{0_3}} & 1 & 0 & \ddot{a_{3_3}} \\
\ddot{a_{0_4}} & 0 & 1 & \ddot{a_{3_4}} \\
\ddot{a_{0_2}} & 0 & 0 & \ddot{a_{3_2}}
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\ddot{b_0} \\
\ddot{b_2} \\
\ddot{b_3} \\
\ddot{b_1}
\end{array}
\right ]
$$

##### 对其他列分别进行同样的变换处理
在上述示例中，对第1列和第4列进行相似的变换处理：
$$
\left[ 
\begin{array}{c c c c}
{\color{Red} \ddot{a_{0_1}} } & 0 & 0 & {\color{Red} \ddot{a_{3_1}} } \\
\ddot{a_{0_3}} & 1 & 0 & \ddot{a_{3_3}} \\
\ddot{a_{0_4}} & 0 & 1 & \ddot{a_{3_4}} \\
{\color{Red} \ddot{a_{0_2}} } & 0 & 0 & {\color{Red} \ddot{a_{3_2}} }
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\ddot{b_0} \\
\ddot{b_2} \\
\ddot{b_3} \\
\ddot{b_1}
\end{array}
\right ]
$$

最终会得到目标变换形式：
$$
\left[ 
\begin{array}{c c c c}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{array}
\right ]
\times
{
\left[ 
\begin{array}{c} 
x_0 \\
x_1 \\
x_2 \\
x_3
\end{array}
\right ]
}
=
\left [
\begin{array}{c}
\hat{b_0} \\
\hat{b_1} \\
\hat{b_2} \\
\hat{b_3}
\end{array}
\right ]
$$

这样就得到方程的解：$x_0 = \hat{b_0}, x_1 = \hat{b_1},x_2 = \hat{b_2},x_3 = \hat{b_3}$。

---
By GuJun， 2021-5-24
