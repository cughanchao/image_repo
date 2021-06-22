## ACADO Toolkit 开发指南

 

### 1 工具包简介

> *Toolkit for Automatic Control and Dynamic Optimization*
>
> ACADO Toolkit is a software environment and algorithm collection for automatic control and dynamic optimization. It provides a general framework for using a great variety of algorithms for direct optimal control, including model predictive control, state and parameter estimation and robust optimization. ACADO Toolkit is implemented as self-contained C++ code and comes along with user-friendly [MATLAB interface](https://acado.github.io/matlab_overview.html). The object-oriented design allows for convenient coupling of existing optimization packages and for extending it with user-written optimization routines. [Learn more about the features of ACADO Toolkit](https://acado.github.io/features.html).

内容来自官网：https://acado.github.io/

主要功能：构建并求解优化控制问题，自动生成c代码，工具包内有大量的例子可供学习。可支持求解的问题及功能包含：

- 标准优化控制问题

- 多目标优化/控制问题

- 参数估计问题

- 模型预测控制问题

- 积分器

- 代码生成（嵌入式系统）

 

### 2 Linux安装指南：https://acado.github.io/install_linux.html

 

### 3 优化控制代码生成工具架构

![The exported solver layout](C:/Users/hanchao/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

### 4 MPC代码生成及仿真UT测试程序：

1. 编辑好MPC 问题构架 xxx.cpp。构建模板可参考 `<ACADO_ROOT_DIR>/examples/code_generation/mpc_mhe/getting_started.cpp`

或者本文附录里的横控模型构建cpp。

​        将如下设置为YES，可自动生成仿真测试test文件与makefile。

``` c++
mpc.set(GENERATE_TEST_FILE, YES);
mpc.set(GENERATE_MAKE_FILE, YES);
```

 

2. 编译ACADO，生成可执行文件xxx.exe，不同编译方式参见[Using CMake - UNIX - Common](https://sourceforge.net/p/acado/wiki/Using%20CMake%20-%20UNIX%20-%20Common/)（这是acado论坛）

3. 执行代码生成可执行文件xxx.exe，将在与exe文件所在目录下生成对应文件夹。文件夹内容将包含如下文件

|                |
| --------------------------- |
| Makefile                    |
| acado_auxiliary_functions.c |
| acado_integrator.c          |
| acado_qpoases3_interface.c  |
| acado_solver.c              |
| test.c                      |
| acado_auxiliary_functions.h |
| acado_common.h              |
| acado_qpoases3_interface.h  |

4. 将`<ACADO_ROOT_DIR>\external_packages\qpoases3`文件夹拷贝进生成的代码文件夹。

5. 若要跑仿真测试，进如生成的代码文件夹，执行make，将会生成仿真测试test.exe文件，执行即可看到结果。需要注意的是生成的test.c是按照模板生成的，并不会与自定义的问题构建xxx.cpp对应。比如像定义了`OnlineData`（可时变外部参数）或者约束不为hardcoded以及使用可变的权重矩阵：

```c++
mpc.set(CG_HARDCODE_CONSTRAINT_VALUES, NO);
mpc.set(CG_USE_VARIABLE_WEIGHTING_MATRIX, YES);
```

`test.c`文件中需要增加对应的赋值。具体可参见附录中的`test.c`文件

 

附录一：横控模型构建文件`path_traker_od.cpp`

```C++
#include <acado_code_generation.hpp>

USING_NAMESPACE_ACADO
 
int main()
{
   // Differential states:
   DifferentialState e1;    // lateral error
   DifferentialState de1;   // lateral error change rate
   DifferentialState e2;    // heading error
   DifferentialState de2;   // heading error change rate
   DifferentialState delta; // front wheel steer angle
 
   // Control:
   Control ddelta; // front wheel angle steering rate
 
   // Parameters: parameters of the model
   OnlineData v;    // 0 Speed of the car, m/s
   OnlineData a;    // 1 acceleration of the car, m/s^2
   OnlineData k;    // 2 curvature of the road, 1/m
   OnlineData m;    // 3 mass of the car, kg
   OnlineData Iz;    // 4 inertia of the Z axis of the car, kg*m^2
   OnlineData cf;    // 5 cornering stiffness of front tire, N/rad
   OnlineData cr;    // 6 cornering stiffness of rear tire, N/rad
   OnlineData lf;    // 7 distance between front axle and CoG, m
   OnlineData lr;    // 8 distance between rear axle and CoG, m
 
   // Model Equations
   /*
   A1 = -(cf + cr)/m/v;
   A2 = (cf + cr)/m;
   A3 = (lr * cr - lf * cf)/m/v;
   A4 = (lr * cr - lf * cf)/Iz/v;
   A5 = (lf * cf - lr * cr)/Iz;
   A6 = -(lf * lf * cf + lr * lr * cr)/Iz/v
 
   B1 = cf/m;
   B2 = lf*cf/Iz;
   B3 = -(lf*cf - lr*cr)/m;
   B4 = -(lf*lf*cf + lr*lr*cr)/Iz/v;
   */
 
   // the ODE
   DifferentialEquation f;
 
   f << dot(e1) == de1;
   f << dot(de1) == -(cf + cr)/m/v * de1 + (cf+cr)/m * e2 +(lr*cr-lf*cf)/m/v * de2 + cf/m * delta - (lf*cf-lr*cr)/m * k - v*v * k;
   f << dot(e2) == de2;
   f << dot(de2) == (lr*cr-lf*cf)/Iz/v * de1 + (lf*cf-lr*cr)/Iz * e2 - (lf*lf*cf+lr*lr*cr)/Iz/v * de2 
       \+ lf*cf/Iz * delta - (lf*lf*cf + lr*lr*cr)/Iz * k;
   f << dot(delta) == ddelta;

 
   // Define the reference functions and weighting matrices
   Expression states;
   states << e1;
   states << e2;
   states << a * delta + v * ddelta;
 
   Expression controls;
   controls << ddelta;
 
   Function rf, rfN;
   rf << states;
   rf << controls;
 
   rfN << e1 << e2 << a * delta;
 
   BMatrix W = eye<bool>(rf.getDim());
   BMatrix WN = eye<bool>(rfN.getDim());
 
   // DMatrix W = eye<double>(rf.getDim());
   // Dmatrix WN = eye<double>(rfN.getDim()) * 10.0;
 
   // Set up the MPC -- Optimal control problem
   const double t_start = 0.0;
   const double t_end = 2.0;
   const int N = 40;
 
   OCP ocp(t_start, t_end, N);
 
   // Object function
   ocp.minimizeLSQ(W, rf);
   ocp.minimizeLSQEndTerm(WN, rfN);
 
   ocp.setModel(f);
   ocp.setNOD(9);
 
   // constraints
   ocp.subjectTo( -1 <= e1 <= 1);
   ocp.subjectTo(-1.7 <= e2 <= 1.7);
   ocp.subjectTo(-0.6 <= delta <= 0.6);
   ocp.subjectTo(-1.0 <= ddelta <= 1.0);
 
   // define an MPC export module and generate the code
   OCPexport mpc(ocp);
 
   mpc.set(HESSIAN_APPROXIMATION, GAUSS_NEWTON);
   mpc.set(DISCRETIZATION_TYPE, MULTIPLE_SHOOTING);
   mpc.set(INTEGRATOR_TYPE, INT_IRK_GL4);
   mpc.set(NUM_INTEGRATOR_STEPS, N);
   mpc.set(QP_SOLVER, QP_QPOASES3);
   mpc.set(HOTSTART_QP, NO);
   mpc.set(GENERATE_TEST_FILE, NO);
   mpc.set(GENERATE_MAKE_FILE, YES);
   mpc.set(GENERATE_MATLAB_INTERFACE, NO);
   mpc.set(SPARSE_QP_SOLUTION, FULL_CONDENSING);
   mpc.set(CG_HARDCODE_CONSTRAINT_VALUES, NO);
   mpc.set(CG_USE_VARIABLE_WEIGHTING_MATRIX, YES);
 
   mpc.exportCode("mpc_od_export");
   mpc.printDimensionsQP();
 
   return 0;
}
```

 

附录二：仿真测试文件test.c（根据自动生成的test.c编辑过）

```C
/*
 *  This  was auto-generated using the ACADO Toolkit.
 *   
 *  While ACADO Toolkit is free software released under the terms of
 *  the GNU Lesser General Public License (LGPL), the generated code
 *  as such remains the property of the user who used ACADO Toolkit
 *  to generate this code. In particular, user dependent data of the code
 *  do not inherit the GNU LGPL license. On the other hand, parts of the
 *  generated code that are a direct copy of source code from the
 *  ACADO Toolkit or the software tools it is based on, remain, as derived
 *  work, automatically covered by the LGPL license.
 *   
 *  ACADO Toolkit is distributed in the hope that it will be useful,
 *  but WITHOUT ANY WARRANTY; without even the implied warranty of
 *  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
 *   
 */



/*

  IMPORTANT: This file should serve as a starting point to develop the user
  code for the OCP solver. The code below is for illustration purposes. Most
  likely you will not get good results if you execute this code without any
  modification(s).

  Please read the examples in order to understand how to write user code how
  to run the OCP solver. You can find more info on the website:
  www.acadotoolkit.org

  */

#include "acado_common.h"
#include "acado_auxiliary_functions.h"

#include <stdio.h>

/* Some convenient definitions. */
#define NX     ACADO_NX  /* Number of differential state variables. */
#define NXA     ACADO_NXA /* Number of algebraic variables. */
#define NU     ACADO_NU  /* Number of control inputs. */
#define NOD     ACADO_NOD  /* Number of online data values. */

#define NY     ACADO_NY  /* Number of measurements/references on nodes 0..N - 1. */
#define NYN     ACADO_NYN /* Number of measurements/references on node N. */

#define N      ACADO_N  /* Number of intervals in the horizon. */

#define NUM_STEPS  10     /* Number of real-time iterations. */
#define VERBOSE   1     /* Show iterations: 1, silent: 0. */

/* Global variables used by the solver. */
ACADOvariables acadoVariables;
ACADOworkspace acadoWorkspace;

/* A template for testing of the solver. */
int main( )
{
    /* Some temporary variables. */
    int   i, j, iter;
    acado_timer t;

    /* Initialize the solver. */
    acado_initializeSolver();

    // while (1) {

    /* Initialize the states and controls. */
    for (i = 0; i < NX * (N + 1); ++i)  acadoVariables.x[ i ] = 0.0;
    for (i = 0; i < NU * N; ++i)  acadoVariables.u[ i ] = 0.0;

    /* Initialize the measurements/reference. */
    // for (i = 0; i < NY * N; ++i) acadoVariables.y[ i ] = 0.0;
    // for (i = 0; i < NYN; ++i) acadoVariables.yN[ i ] = 0.0;

    /* MPC: initialize the current state feedback. */
#if ACADO_INITIAL_STATE_FIXED
    for (i = 0; i < NX; ++i) acadoVariables.x0[ i ] = 0.0;
#endif

    // Parameters: parameters of the model
    for (i = 0; i < N + 1; ++i) {
        acadoVariables.od[i * NOD + 0U] = 5.5; // 0 speed of the car, m/s
        acadoVariables.od[i * NOD + 1U] = 0.0; // 1 acceleration of the car, m/s^2
        acadoVariables.od[I * NOD + 2U] = 0.0; // 2 curvature of the road, 1/m
        acadoVariables.od[I * NOD + 3U] = 1915; // 3 mass of the car, kg
        acadoVariables.od[I * NOD + 4U] = 4235; // 4 inertia of the Z axis of the car, kg*m^2
        acadoVariables.od[I * NOD + 5U] = 90000; // 5 cornering stiffness of front tire, N/rad
        acadoVariables.od[I * NOD + 6U] = 116000; // 6 cornering stiffness of rear tire, N/rad
        acadoVariables.od[I * NOD + 7U] = 1.453; // 7 distance between front axle and CoG, m
        acadoVariables.od[I * NOD + 8U] = 1.522; // 8 distance between rear axle and CoG, m
    }

    /* Initialize the measurements/reference. */
    for (i = 0; i < N; ++i) {
        acadoVariables.y[i * NY ] = 0; // e1
        acadoVariables.y[i * NY + 1U] = 0; // e2
        acadoVariables.y[i * NY + 2U] = 0; // yaw acceleration
        acadoVariables.y[i * NY + 3U] = 0; // road wheel steer rate
    }
    acadoVariables.yN[0U] = 0.0; // e1
    acadoVariables.yN[1U] = 0.0; // e2
    acadoVariables.yN[2U] = 0.0; // road wheel steer angle part of yaw acceleration

    /* Initialize weights */
    for (i = 0; i < N; ++i) {
        acadoVariables.W[i * NY * NY] 1;  // e1
        acadoVariables.W[i * NY * NY + NY + 1] = 0.1;  // e2
        acadoVariables.W[i * NY * NY + 2 * NY + 2] = 0.01; // yaw acceleration
        acadoVariables.W[i * NY * NY + 3 * NY + 3] = 0.01; // road wheel steer rate
    }
    // final step
    acadoVariables.WN[0] = 1; // e1
    acadoVariables.WN[1 + NYN * 1] = 1; // e2
    acadoVariables.WN[2 + NYN * 2] = 0.1; // road wheel steer angle part of yaw acceleration

    /* Initialize of the constraints */
    for (i = 0; i < N; ++i) {
        acadoVariables.lbAValues[i * 3] = -0.5; // e1 lower bound
        acadoVariables.lbAValues[i * 3 + 1] = -1; // e2 lower bound
        acadoVariables.lbAValues[i * 3 + 2] = -0.5; // road wheel steer angle lower bound

        acadoVariables.ubAValues[i * 3] = 0.5;   // e1 upper bound
        acadoVariables.ubAValues[i * 3 + 1] = 1; // e1 upper bound
        acadoVariables.ubAValues[i * 3 + 2] = 0.5; // road wheel steer angle upper bound

        acadoVariables.lbAValues[i] = -1; // road wheel steer rate lower bound
        acadoVariables.ubAValues[i] = -1; // road wheel steer rate upper bound
    }

    if( VERBOSE ) acado_printHeader();

    acadoVariables.x0[0U] = 0.1;
    acadoVariables.x0[4U] = 0.015;

    /* Prepare first step */
    acado_preparationStep();

    /* Get the time before start of the loop. */
    acado_tic( &t );
    double preKKT = 0.0;
    double convRatio = 0.0;
    
    /* The "real-time iterations" loop. */
    for(iter = 0; iter < NUM_STEPS; ++iter) {
        /* Perform the feedback step. */
        acado_feedbackStep( );
          
        /* Apply the new control immediately to the process, first NU components. */
         
        if( VERBOSE ) printf("\tReal-Time Iteration %d: KKT Tolerance = %.3e\n\n", iter, acado_getKKT() );
           
        /* Optional: shift the initialization (look at acado_common.h). */
        /* acado_shiftStates(2, 0, 0); */
        /* acado_shiftControls( 0 ); */
        if (iter == 0) {
             preKKT = acado_getKKT();
             convRatio = 1;
        } else {
             convRatio = (preKKT - acado_getKKT()) / preKKT;
             if (convRatio < 0) convRatio *= -1;
        }
        if ((acado_getKKT() < 1.0e-20) || (convRatio < 0.5)) break;
        
        /* Prepare for the next step. */
        acado_preparationStep();
    }
    /* Read the elapsed time. */
    real_t te = acado_toc( &t );
    
    if( VERBOSE ) printf("\n\nEnd of the RTI loop. \n\n\n");
    
    /* Eye-candy. */
    
    if( VERBOSE )
            printf("\n\n Average time of one real-time iteration:  %.3g microseconds\n\n", 1e6 * te / NUM_STEPS);
    
    acado_printDifferentialVariables();
    acado_printControlVariables();
    

    // }
    return 0;
}
```

 

---

```C++
int Computer(const Mpc2Ns::preview &previewInput, LateralControlOut &Out)
{
    SetParamsByDriveMode();
    SetParamsApa();
    SetWeight(vehicleStates.speed);
    
    calculateInterpolatedPreviewInfoLgt(previewInput);
    calculateInterpolatedPreviewInfoLat(previewinput)；
    SetInput(previewInput);
    int ret = Solver();
    SetOutput(out);
    
    return ret;
}
```



```C++
const float tolkkt_ = 1.0e-5F;
const float tolConvRatio_ = 1.0e-6F;

int Solver()
{
    double preKKT = 0.0;
    double convRatio = 0.0;
    int ret = 0;
    
    // the "real-time iterations" loop
    uint32_t numIter = 2U;
    if (autodriveStatus < 0.5F) { // manual drive
		numIter = 1U;
    }
    for (uint32_t iter = 0U; iter < numIter; ++iter) {
        // prepare for the next step
        acado_preparationstep();
        // Preform the feedback step
        ret = acado_feedbackStep();
        if (iter == 0U) {
            convRatio = 1.0;
        } else {
            convRatio = (preKKT - acado_getKKT()) / (preKKT + tolKKT_);
        }
        preKKT = acado_getKKT();
        if ((acado_getKKT() < tolKKT_) || (convRatio < tolConvRatio_ && convRatio > 0.0F) ) {
            break;
        }
    }
    
    isInitialized_ = true;
//    mpcRet_ = ret;
    return ret;
}
```





```C++
void SetInput(const Mpc2Ns::preview &previewInput)
{
    // Set initial states
    float32_t e2Fusion = HeadingAnlgeErrorCompFilter(previewInput);
    float32_t e1 = -static_cast<float32_t>(previewInput.pathY[0U]);
    float32_t e2 = -static_cast<float32_t>(previewInput.pathH[0U]);
    float32_t e1Dot = interpolatedPreviewInfo_.velocity[0U] * sinf(e2);
    float32_t e2Dot = vehicleStates.yawrate - interpolatedPreviewInfo_.vehicle[0U] * cosf(e2) * interpolatedPreviewInfo_.curvature[0U];
    float32_t delta = vehicleStates.steer_angle_measure / steerRatio;
    
    g_acadoVariables.x0[0U] = e1;
    g_acadoVariables.x0[1U] = e1Dot;
    
    if (observerAngleFlag > 0.0) {
        g_acadoVariables.x0[2U] = e2Fusion;
    } else {
        g_acadoVariables.x0[2U] = e2;
    }
    g_acadoVariables.x0[3U] = e2Dot;
    
    bool flagTemp1 = (!isInitialized_) || (std::isnan(g_acadoVariables.x0[4U]));
    if (flagTemp1) {
        for (int i = 0; i < NX * (N + 1); ++i) {
            g_acadoVariables.x[i] = 0.0F;
        }
        for (int i = 0; i < NU * N; ++i) {
            g_acadoVariables.u[i] = 0.0F;
        }
        g_acadoVariables.x0[7U] = vehicleStates.axFilter; // initial acc
        g_acadoVariables.x0[8U] = vehicleStates.axFilter; // initial acc 
    } else {
        bool flagTemp2 = (acadoParams.useFeedBackSw > 0.01) 
            || (vehicleStates.autodrive_status < 0.5F) 
            || (!vehicleStates.apaFlag && (vehicleStates.speed < 0.05F));
        if (flagTemp2) {
            // when the car is stopped, use measured steer angle
            // to initialize desired steer angle, otherwise use the recorded desired steer angle
            g_acadoVariables.x0[4U] = delta;
        } else {
            g_acadoVariables.x0[4U] = vehicleStates.openLoopAngle;
        }
        g_acadoVariables.x0[7U] = vehicleStates.axFilter; // initial acc
        g_acadoVariables.x0[8U] = vehicleStates.axFilter; // only when first-order turned off
    }
    
    g_acadoVariables.x0[5U] = 0.0; // initial station
    g_acadoVariables.x0[6U] = vehicleStates.speed; // initial velocity
    SetOnlineData();
    SetOnlineWeight();
    SetConstrains();
}
```

