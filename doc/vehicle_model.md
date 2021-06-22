## 车辆动力学模型



### 运动学模型（阿克曼转向）

基于Ackerman转向的车辆运动学模型
$$
\begin{bmatrix} 
    \dot{X}_{r} \\ 
    \dot{Y}_{r} \\ 
    \dot{\varphi}
\end{bmatrix} = 
\begin{bmatrix}
	cos \varphi \\
    sin \varphi \\
    \frac{tan \delta_f}{L} 
\end{bmatrix} 
v_r
$$

其中，$\varphi$ 为车辆横摆角（航向角，左正右负），$\delta_f$ 为车辆前轮转角（左正右负），$L$为轴距，$v_r$ 为车辆后轴中心处的速度，$X_r, Y_r$ 为车辆后轴中心在定位坐标系中的坐标。



### 运动学模型（质心）





### 二自由度动力学模型

二自由度动力学模型
$$
\begin{bmatrix} 
	\dot{v}_y \\
	\dot{\phi}
\end{bmatrix} =
\begin{bmatrix}
	- \frac{C_f + C_r}{m v_x}  & - v_x - \frac{C_f l_f - C_r l_r}{m v_x} \\
	- \frac{C_f l_f - C_r l_r}{I_z v_x} & - \frac{C_f l_f^2 + C_r l^2_r}{I_z v_x}
\end{bmatrix}
\begin{bmatrix}
	v_y \\
	\phi
\end{bmatrix} + 
\begin{bmatrix}
	\frac{C_f}{m} \\
	\frac{C_f l_f}{I_z}
\end{bmatrix} 
\delta_f
$$

令，车辆横摆角（航向角）为$\varphi$，轴距为$L$。则，$\phi$ 为车辆横摆角速率，$\phi = \dot{\varphi}$ 。$l_f, l_r$ 分别代表车辆质心到前轮轴和后轮轴的距离，且$l_f + l_r = L$，$C_f, C_r$ 分别代表前后轮等效侧偏刚度。$v_x, v_y$分别代表车辆在车体坐标轴上的速度。



另一种写法（状态的顺序不一样）：
$$
\begin{bmatrix}
	\dot{\phi} \\
	\dot{v}_y \\
\end{bmatrix} = 
\begin{bmatrix}
	\frac{-(l^2_f C_f + l^2_r C_r)}{I_z v_x}  &  \frac{l_r C_r - l_f C_f}{I_z v_x} \\
 	\frac{l_r C_r - l_f C_f}{m v_x} - v_x  & \frac{-(C_f + C_r)}{m v_x}  \\
\end{bmatrix} 
\begin{bmatrix}
	\phi  \\
	v_y \\
\end{bmatrix} + 
\begin{bmatrix}
	\frac{l_f C_f}{I_z}\\
	\frac{C_f}{m} \\
\end{bmatrix} \delta_f
$$

则在固定车体坐标系下的运动速度：
$$
V_x = v_x cos \varphi- v_y sin \varphi \\
V_y =v_x sin \varphi+v_y cos \varphi
$$

参考：

[汽车二自由度模型公式推导及simulink模型——传递函数、状态空间](https://www.codenong.com/cs105888247/)

[车辆工程（1）——线性二自由度汽车模型的运动方程](https://blog.csdn.net/Mr_ZZTC/article/details/100580687)



### 道路曲率的动力学模型


$$
\dot{x} = Ax + Bu + M
$$
其中，$x=[e_1, \dot{e}_1, e_2, \dot{e}_2]$，$\dot{\varphi}_{des} = \frac{v_x}{R}$，$u=\delta$，$C_f,C_r$为前后轮胎侧偏刚度的两倍。
$$
A=
\begin{bmatrix}
	0 & 1 & 0 & 0 \\
	0 &   -\frac{C_f + C_r}{m v_x} & \frac{C_f + C_r}{m} & \frac{-C_f l_f + C_r l_r}{m v_x} \\
	0 & 0 & 0 & 1 \\
	0 & -\frac{-C_f l_f - C_r l_r}{I_z v_x} & \frac{C_f l_f - C_r l_r}{I_z} & -\frac{C_f l_f^2 + C_r l_r^2}{I_z v_x}
\end{bmatrix}, 
B = 
\begin{bmatrix}
	0 \\
	\frac{C_f}{m} \\
	0  \\
	\frac{C_f l_f}{I_z}
\end{bmatrix},
M=
\begin{bmatrix}
	0 \\
	-\frac{C_f l_f - C_r l_r}{m v_x} - v_x \\
	0 \\
	-\frac{C_f l_f^2 + C_r l_r^2}{I_z v_x}
\end{bmatrix} \dot{\varphi}_{des}
$$


参考：

[1] R. N. Jazar, *Vehicle Dynamics: Theory and Application (2nd Edition)*. New York, NY: Springer, 2014. Chapter 3.

