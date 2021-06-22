# gtest 安装与使用

@[toc]

## 1. 下载并安装[googletest](https://github.com/google/googletest)

目前最新的release版本是`googletest-release-1.10.0`，下载压缩包，解压并按照下面的命令编译安装。

```bash
$ cd googletest-release-1.10.0/
$ mkdir build && cd build
$ cmake .. -DBUILD_GMOCK=OFF # 关闭 gmock
$ make
$ sudo make install

...
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/lib/cmake/GTest/GTestTargets.cmake
-- Installing: /usr/local/lib/cmake/GTest/GTestTargets-noconfig.cmake
-- Installing: /usr/local/lib/cmake/GTest/GTestConfigVersion.cmake
-- Installing: /usr/local/lib/cmake/GTest/GTestConfig.cmake
-- Up-to-date: /usr/local/include
-- Installing: /usr/local/include/gtest
-- Installing: /usr/local/include/gtest/gtest-spi.h
-- Installing: /usr/local/include/gtest/gtest-death-test.h
-- Installing: /usr/local/include/gtest/gtest-param-test.h
-- Installing: /usr/local/include/gtest/gtest-matchers.h
-- Installing: /usr/local/include/gtest/gtest-typed-test.h
-- Installing: /usr/local/include/gtest/gtest-test-part.h
-- Installing: /usr/local/include/gtest/gtest_prod.h
-- Installing: /usr/local/include/gtest/internal
-- Installing: /usr/local/include/gtest/internal/gtest-param-util.h
-- Installing: /usr/local/include/gtest/internal/gtest-internal.h
-- Installing: /usr/local/include/gtest/internal/gtest-string.h
-- Installing: /usr/local/include/gtest/internal/gtest-port.h
-- Installing: /usr/local/include/gtest/internal/custom
-- Installing: /usr/local/include/gtest/internal/custom/gtest-port.h
-- Installing: /usr/local/include/gtest/internal/custom/gtest-printers.h
-- Installing: /usr/local/include/gtest/internal/custom/README.md
-- Installing: /usr/local/include/gtest/internal/custom/gtest.h
-- Installing: /usr/local/include/gtest/internal/gtest-death-test-internal.h
-- Installing: /usr/local/include/gtest/internal/gtest-type-util.h.pump
-- Installing: /usr/local/include/gtest/internal/gtest-type-util.h
-- Installing: /usr/local/include/gtest/internal/gtest-port-arch.h
-- Installing: /usr/local/include/gtest/internal/gtest-filepath.h
-- Installing: /usr/local/include/gtest/gtest-message.h
-- Installing: /usr/local/include/gtest/gtest-printers.h
-- Installing: /usr/local/include/gtest/gtest.h
-- Installing: /usr/local/include/gtest/gtest_pred_impl.h
-- Installing: /usr/local/lib/libgtest.a
-- Installing: /usr/local/lib/libgtest_main.a
-- Installing: /usr/local/lib/pkgconfig/gtest.pc
-- Installing: /usr/local/lib/pkgconfig/gtest_main.pc
```
这样就完成了本地安装gtest。



## 2. cmake 工程使用 gtest

gtest 的编程语法请参考[Googletest Primer](https://google.github.io/googletest/primer.html)，文末的参考中有网友翻译的中文版。下面以一个完整的C++工程为例详细介绍如何在自己的cmake工程中使用gtest做单元测试用例。

示例工程目录结构如下

```xml
gtest_study/
 ├── CMakeLists.txt
 ├── src/
 │   ├── limiter.cpp
 │   ├── limiter.h
 │   └── main.cpp
 └── test/
     ├── CMakeLists.txt
     ├── main_test.cpp
     └── ut/
         └── test_limiter.cpp
```



其中，`limiter.cpp`和`limiter.h`只是为了说明gtest的使用写的一个简单类。各文件的内容如下。主 `CMakeLists.txt`：

```cmake
cmake_minimum_required(VERSION 3.1)
project(gtest_study)

# enable c++ 11
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include_directories(
    src/
)

add_executable(main src/main.cpp src/limiter.cpp)

# CTest related
enable_testing()
add_subdirectory(test)
```

`limiter.h`

```cpp
#ifndef LIMITER_H_
#define LIMITER_H_

class Limiter {
public:
    double AmplitudeLimit(double input, double minValue, double maxValue);

}; // end of class

#endif // end of LIMITER_H_
```

`limiter.cpp`

```cpp
#include"limiter.h"

double Limiter::AmplitudeLimit(double input, double minValue, double maxValue)
{
    double output;
    if (input < minValue) {
        output = minValue;
    } else if (input > maxValue) {
        output = maxValue;
    } else {
        output = input;
    }

    return output;
}
```

`main.cpp`

```cpp
#include <iostream>
#include "limiter.h"

int main(int argc, char** argv)
{
    Limiter valueLimiter;
    double inputValue = 2.0;
    double minLimiter = -1.0;
    double maxLimiter = 1.0;

    double output = valueLimiter.AmplitudeLimit(inputValue, minLimiter, maxLimiter);
    std::cout << "output = " << output << "\n";

    return 0;
}
```

test目录下的 `CMakeLists.txt`

```cmake
find_package(GTest REQUIRED)     # 包含gtest
find_package(Threads REQUIRED)

include_directories(
    ${GTEST_INCLUDE_DIR}         # 包含gtest头文件
    ${PROJECT_SOURCE_DIR}/src/
    ${PROJECT_SOURCE_DIR}/test/
)

add_executable(main_test 
    main_test.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/ut/test_limiter.cpp
    ${PROJECT_SOURCE_DIR}/src/limiter.cpp
)

target_link_libraries(main_test 
    ${GTEST_BOTH_LIBRARIES}   # 链接gtest库文件
    pthread
)

add_test(
    NAME main_test
    COMMAND ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}/main_test
)
```

单元测试用例`ut/test_limiter.cpp`如下。这个文件可以作为单元测试用例的模板。

```cpp
#include "gtest/gtest.h"
#include "limiter.h"

// The fixture for testing class Foo.
class LimiterTest : public ::testing::Test {
protected:
    // You can remove any or all of the following functions if their bodies would
    // be empty.

    LimiterTest() {
        // You can do set-up work for each test here.
    }

    ~LimiterTest() override {
        // You can do clean-up work that doesn't throw exceptions here.
    }

    // If the constructor and destructor are not enough for setting up
    // and cleaning up each test, you can define the following methods:

    void SetUp() override {
        // Code here will be called immediately after the constructor (right
        // before each test).
    }

    void TearDown() override {
        // Code here will be called immediately after each test (right
        // before the destructor).
    }

    // Class members declared here can be used by all tests in the test suite
    // for Foo.
    Limiter testValueLimiter;
};

// Tests that the Foo::Bar() method does Abc.
TEST_F(LimiterTest, LimiterMax) 
{
    
    double minLimiter = -0.9;
    double maxLimiter = 0.9;
    double inputValue = 1.1;
    double outputValue = testValueLimiter.AmplitudeLimit(inputValue, minLimiter, maxLimiter);
    std::cout << "[TEST] outputValue = " <<  outputValue << "\n";

    EXPECT_EQ(outputValue, maxLimiter);
}

```

`main_test.cpp`

```cpp
#include "gtest/gtest.h"

int main(int argc, char **argv) {
  ::testing::InitGoogleTest(&argc, argv);
  return RUN_ALL_TESTS();
}
```

编译并运行工程如下：

```bash
$ cd gtest_study/
$ mkdir build && cd build
$ cmake ..
$ make 
# 编译成功运行主节点和测试节点，如下
$./main 
output = 1
$ ./test/main_test 
[==========] Running 1 test from 1 test suite.
[----------] Global test environment set-up.
[----------] 1 test from LimiterTest
[ RUN      ] LimiterTest.LimiterMax
[TEST] outputValue = 0.9
[       OK ] LimiterTest.LimiterMax (0 ms)
[----------] 1 test from LimiterTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test suite ran. (0 ms total)
[  PASSED  ] 1 test.

```



## 3. 将 gtest 编译为动态链接库添加到cmake工程使用

有的时候我们会将所使用的第三方依赖库作为工程的一部分，这样工程就可以不依赖运行环境了，避免在每台运行工程的电脑上都要安装依赖程序。下面介绍将gtest编译为动态连接库添加到cmake工程中使用的方法。

工程目录结构如下;

```xml
gtest_lib_study/
 ├── CMakeLists.txt
 ├── lib/
 │   └── googletest-1.10.0/
 │       ├── include/
 │       │   └── ...
 │       └── lib/
 │           ├── libgtest_main.so
 │           └── libgtest.so
 ├── src/
 │   ├── limiter.cpp
 │   ├── limiter.h
 │   └── main.cpp
 └── test/
     ├── CMakeLists.txt
     ├── main_test.cpp
     └── ut/
         └── test_limiter.cpp
```

首先，将googletest编译为动态连接库，然后按照上面的结构将生成的库文件和头文件添加到工程，编译的方式跟上面一样。

```c++
$ cd googletest-release-1.10.0/
$ mkdir build && cd build
$ cmake .. -DBUILD_GMOCK=OFF -DBUILD_SHARED_LIBS=ON # 关闭 gmock, 生成动态链接库so文件，默认生成.o文件
$ make  # 这样就会生成一个lib目录，里面有libgtest.so和libgtest_main.so两个库文件。
```

然后，修改tests下`CMakeLists.txt `文件如下，其他文件保持不变。

```shell
find_package(Threads REQUIRED)

include_directories(
    ${PROJECT_SOURCE_DIR}/lib/googletest/include/  # 包含gtest头文件
    ${PROJECT_SOURCE_DIR}/src/
    ut/
)
link_directories(${PROJECT_SOURCE_DIR}/lib/googletest/lib) #包含gtest库文件所在的路径

add_executable(gtest_study 
    main_test.cpp 
    ${CMAKE_CURRENT_SOURCE_DIR}/ut/limiter_test.cpp
    ${PROJECT_SOURCE_DIR}/src/limiter.cpp
)
target_link_libraries(gtest_study 
    gtest        # 链接gtest库文件
    gtest_main   # 链接gtest库文件
    pthread
)

add_test(
    NAME gtest_study
    COMMAND ${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}/gtest_study
)
```



编译与运行工程的方式跟上面一样，结果也一样。



参考：

[Googletest Primer](https://google.github.io/googletest/primer.html)

[GTEST做C++单元测试初级教程（GTEST Prime译文）](https://www.jianshu.com/p/c7c702c0abb9)

[第一个gtest程序（Linux）](https://www.jianshu.com/p/778f835cc18c)

[关于C ++：如何在Linux上将googleTest设置为共享库](https://www.codenong.com/13513905/)

[CMake 与 Google Test](https://zjuturtle.com/2017/11/29/cmake-gtest/)

[cmake + googletest 之一 入门](https://blog.csdn.net/joelcat/article/details/90766192)

[现代CMake编程指南：集成GTest](https://zhuanlan.zhihu.com/p/164482548)

[yecharlie/gtest-demo](https://github.com/yecharlie/gtest-demo)