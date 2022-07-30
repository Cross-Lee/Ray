> 原文链接：https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#cleaninguppdfmanagement/handlingsurfaceacne

## 一、前言
- 使用蒙特卡洛积分
- 不做阴影，避免产生阴影
- 更复杂的也不会说（双向反射、Metropolis方法、光子映射）

## 二、简单的蒙特卡洛积分程序
蒙特卡洛积分就是对积分随机采样进行求解，根据pdf（概率密度函数）进行采样
##### 2.1 估计$\pi$的值
通过布冯针的实验原理来估计$\pi$的值
```cpp
#include "rtweekend.h"

#include <iostream>
#include <iomanip>
#include <math.h>
#include <stdlib.h>

int main() {
    int N = 1000;
    int inside_circle = 0;
    for (int i = 0; i < N; i++) {
        auto x = random_double(-1,1);
        auto y = random_double(-1,1);
        if (x*x + y*y < 1)
            inside_circle++;
    }
    std::cout << std::fixed << std::setprecision(12);
    std::cout << "Estimate of Pi = " << 4*double(inside_circle) / N << '\n';
}
```
上述的结果与随机种子相关，但是程序逻辑是估计$\pi$的值的
##### 2.2 收敛表示
作者构建一个死循环来不断逼近$\pi$的值

##### 