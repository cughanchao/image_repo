



![](https://cdn.jsdelivr.net/gh/cughanchao/image_repo@main/20200912-buck600%20-%20%E5%89%AF%E6%9C%AC.jpg)





[gitextensions](https://github.com/gitextensions/gitextensions)





![image_test](https://cdn.jsdelivr.net/gh/cughanchao/test_repo@test/img/image_test.png)





----

2020.12-至今 仿真器车辆模型开发



2020.7-2020.12 控制新版MPC重构



2020.4-2020.7  控制模块C++重构

控制模块原本是由Simulink生成的代码，由于去A的需求，需要将所有代码用C++重构。

本人负责完成了迟滞比较器功能、车辆三自由度运动学模型、二自由度动力学模型、特定QP问题求解器、横控MPC问题求解器。





入职5个月以来主要从事的工作包括：指定模型的Cpp重构工作（Mat2Cpp）、代码整改、控制自闭环验证工具control_st开发以及横控、纵控的调试等工作。保质保量的完成了各项工作的交付。下面具体总结。
1.	Mat2Cpp工作：该项目是团队去A的一项重要工作。我负责完成了迟滞比较器功能、车辆三自由度运动学模型、二自由度动力学模型、特定QP问题求解器、横控MPC问题求解器、横控角度仲裁功能以及ARB和APA中的4个函数，总计提交代码2000余行。完成代码codestyle检查，完成UT用例，代码整体覆盖率95%以上。参与团队代码检视，同时根据大家的意见修改代码。
2.	代码整改工作：负责文件LateralParamIdentity.cpp和LateralParamIdentity.h文件的整改，包括codestyle检查、 CMatirx检查、Codex检查和UT用例编写。解决200多个Codex问题，增加5个UT用例，文件UT覆盖率从74%提成到96%。将 control 包里面的所有 new 换成 make_uniqe 函数，并将包含make_unique 的头文件合并到联合开发仓的common分支方便其他包使用。
3.	control_st开发：完成了车辆运动学模型，增加了实车转向特性数据设置转向幅度和转向速率的限制，并与轨迹生成器的集成跑通。实现了Rviz显示规划轨迹和仿真车运动轨迹的功能。完成了直线、圆弧、变道、s曲线、调头等基本测试轨迹类型，整理各类基本测试用例共61个。构建左右连续转向的测试的用例，用于调试连续左右转时侧偏严重的问题。完成脚本自动从obs上根据路测数据构造特定轨迹生成测试用例，并已针对ICA和AVP横控提取轨迹构造用例各一个用于调试。完成脚本根据用例log自动生成Excel测试报告。将ST合入联合开发仓pnc主分支，合入BU仓。完成说明文档readme的写作；完成使用说明分享；制定了第二阶段开发工作的目标和安排。
4.	横控调试：定位并修复高速段横控两种 ay 融合的一个问题，解决了车辆在从50公里时速降速到35公里时速时的异常转向的问题；支持821演示公关，辨识和验证侧偏刚度系数；实车测试多次。
5.	纵控调试：接手辛博纵控的起停功能AR开发，完成该算法UT；完成算法的实车调试和适配，上车测试4次，完成交付。
6.	safestop功能开发：基于C语言，完成轨迹坐标变换代码开发，完成基于纯跟踪的转向控制算法代码开发。后续要需要补充UT和联合测试。
7.	其他工作：参与评审文档写作，输出《ADSV 1.0.0 Control算法性能测试方案》；ADS1.0轮值看护一周，上库5辆A6车控制参数；B828版本控制测试，实车测试2次。



```cpp
// linear interpolation
double CKinematics::LinearInterpolation(const std::map<double, double> &dataMap, const double queryValue)
{
    auto it = dataMap.lower_bound(queryValue);
    double interpolatedVal;
    if (it == dataMap.begin()) {
        interpolatedVal =  it->second;
    } else if (it == dataMap.end()) {
        auto itPrev = --dataMap.end();
        interpolatedVal =  itPrev->second;
    } else {
        auto itPrev = it;
        --itPrev;
        double k = (queryValue - itPrev->first) / (it->first - itPrev->first);
        interpolatedVal = k * it->second + (1.0 - k) * itPrev->second; // 1.0 is double
    }
    return interpolatedVal;
}

void CKinematics::MakeInterpolationMap(const std::vector<double> &key, const double keyRatio,
    const std::vector<double> &value, const double valueRatio, std::map<double, double> &outputMap)
{
    uint32_t keyLen = key.size();
    uint32_t valueLen = value.size();
    if (keyLen == 0U || valueLen == 0U || keyLen != valueLen) {
        ROS_ERROR_STREAM("Error: Input vector size error!\n");
    }
    outputMap.clear();
    for (uint32_t i = 0U; i < keyLen; ++i) {
        outputMap.insert(std::map<double, double>::value_type(key.at(i) * keyRatio, value.at(i) * valueRatio));
    }
}
```

```cpp
void CKinematics::Set2DofModel()
{
    double minSpeedProtect = 0.1;
    double speedTmp = fabs(speed);
    if (speedTmp < minSpeedProtect) {
        speedTmp = minSpeedProtect;
    }
    const double speedDir = (speed >= 0.0 ? 1.0 : -1.0);
    const double tmpSpeedX = speed * speed - speedY * speedY;
    const bool speedFlag = (fabs(speed) < 0.1) || (fabs(speed) < fabs(speedY)) || (tmpSpeedX < 1e-6);
    speedX = speedFlag ? speed : (speedDir * sqrt(tmpSpeedX));

    float lcf = lf_ * cf_;
    float lcr = lr_ * cr_;
    float mom = mass_ * speedTmp; // momentum
    float izv = inertiaZ_ * speedTmp;
    aMat_(0U, 0U) = -(lf_ * lcf + lr_ * lcr) / izv;
    aMat_(0U, 1U) = (lcr - lcf) / izv;
    aMat_(1U, 0U) = (lcr - lcf) / mom - speedTmp;
    aMat_(1U, 1U) = -(cr_ + cf_) / mom;
    bMat_ << (lcf / inertiaZ_), (cf_ / mass_);
}

void CKinematics::Discrete2DofModel()
{
    Eigen::MatrixXf aMatdt = aMat_ * dt;
    uint32_t statesNum = aMat_.rows();
    Eigen::EigenSolver<Eigen::MatrixXf> eigenDecompose(aMatdt);
    Eigen::VectorXcf expEigen(statesNum);
    for (uint32_t i = 0U; i < statesNum; ++i) {
        expEigen(i) = std::exp(eigenDecompose.eigenvalues()(i));
    }
    Eigen::MatrixXcf eigValDiag = expEigen.asDiagonal();
    Eigen::MatrixXcf eigVec = eigenDecompose.eigenvectors();
    adMat_ = (eigVec * eigValDiag * eigVec.inverse()).real();
    bdMat_ = aMat_.inverse() * (adMat_ - Eigen::MatrixXf::Identity(statesNum, statesNum)) * bMat_;
}

// 2-DoF dynamics model
void CKinematics::CalcPoseWith2DOFModel()
{
    Eigen::Vector2f states(yaw_rate, speedY);
    Eigen::Vector2f output(0.0F, 0.0F);
    double xCog = x + lf_ * cos(heading_angle);
    double yCog = y + lf_ * sin(heading_angle);

    Set2DofModel();
    Discrete2DofModel();
    output = adMat_ * states + bdMat_ * steering_angle;

    yaw_rate = output(0U);
    speedY = output(1U);
    
    const double speedDir = (speed >= 0.0 ? 1.0 : -1.0);
    heading_angle += (speedDir * yaw_rate * dt);

    front_right_angle =
            atan(2.0f * wheelBase_ * tan(steering_angle) / (2.0f * wheelBase_ + frontTrack_ * tan(steering_angle)));
    front_left_angle =
            atan(2.0f * wheelBase_ * tan(steering_angle) / (2.0f * wheelBase_ - frontTrack_ * tan(steering_angle)));
    speed_rr = speed + yaw_rate * rearTrack_ / 2.0f;
    speed_rl = speed - yaw_rate * rearTrack_ / 2.0f;
    double cosFrontRightAngle = cos(front_right_angle);
    double cosFrontLeftAngle = cos(front_left_angle);
    if (fabs(cosFrontRightAngle) < 0.001f) {
        cosFrontRightAngle = 1.0;
    }
    if (fabs(cosFrontLeftAngle) < 0.001f) {
        cosFrontLeftAngle = 1.0;
    }
    speed_fr = speed_rr / cosFrontRightAngle;
    speed_fl = speed_rl / cosFrontLeftAngle;
    mileage += speed * dt;

    const double speedYDir = (speedDir * yaw_rate >= 0) ? 1.0 : -1.0;
    float absVelX = speedX * cos(heading_angle) - speedYDir * fabs(speedY) * sin(heading_angle);
    float absVelY = speedX * sin(heading_angle) + speedYDir * fabs(speedY) * cos(heading_angle);
    xCog += absVelX * dt;
    yCog += absVelY * dt;

    x = xCog - lf_ * cos(heading_angle);
    y = yCog - lf_ * sin(heading_angle);
}
```

```c++
void CKinematics::CalcPoseWithout8DOFModel()
{
    double dPsi = speed * tan(steering_angle) / wheelBase_;

    front_right_angle =
        atan(2.0f * wheelBase_ * tan(steering_angle) / (2.0f * wheelBase_ + frontTrack_ * tan(steering_angle)));
    front_left_angle =
        atan(2.0f * wheelBase_ * tan(steering_angle) / (2.0f * wheelBase_ - frontTrack_ * tan(steering_angle)));
    speed_rr = speed + dPsi * rearTrack_ / 2.0f;
    speed_rl = speed - dPsi * rearTrack_ / 2.0f;
    double cosFrontRightAngle = cos(front_right_angle);
    double cosFrontLeftAngle = cos(front_left_angle);
    if (fabs(cosFrontRightAngle) < 0.001f) {
        cosFrontRightAngle = 1.0;
    }
    if (fabs(cosFrontLeftAngle) < 0.001f) {
        cosFrontLeftAngle = 1.0;
    }
    speed_fr = speed_rr / cosFrontRightAngle;
    speed_fl = speed_rl / cosFrontLeftAngle;

    yaw_rate = dPsi;
    mileage += speed * dt;
    heading_angle += dPsi * dt;
    double dx = speed * cos(heading_angle);
    double dy = speed * sin(heading_angle);

    x += dx * dt;
    y += dy * dt;
    /* ***********End Orig Model************ */
}

```

判断线性系统的稳定性，状态空间方程的系统矩阵所有特征值小于1.

```C++
bool JudgeStability(const MatrixXf &matA)
{
    bool stabilityFlag = true;
    uint32_t dimNum = 4U;
    EigenSolver<MatrixXf> eigenDecompose(matA);
    for (uint32_t i = 0U; i < dimNum; ++i) {
        if (eigenDecompose.eigenvalues().cwiseAbs()(i) >= 1.0F) {
            stabilityFlag = false;
        }
    }
    return stabilityFlag;
}
```

