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

##### 2.3 分层采样（抖动）
蒙特卡洛积分的问题在于，每一次新的采样都比上一次采样贡献的更少，因此，通过分层采样来避免这样的问题。
```cpp
#include "rtweekend.h"
#include <iostream>
#include <iomanip>

int main() {
    int inside_circle = 0;
    int inside_circle_stratified = 0;
    int sqrt_N = 10000;
    for (int i = 0; i < sqrt_N; i++) {
        for (int j = 0; j < sqrt_N; j++) {
            auto x = random_double(-1,1);
            auto y = random_double(-1,1);
            if (x*x + y*y < 1)
                inside_circle++;
            x = 2*((i + random_double()) / sqrt_N) - 1;
            y = 2*((j + random_double()) / sqrt_N) - 1;
            if (x*x + y*y < 1)
                inside_circle_stratified++;
        }
    }

    auto N = static_cast<double>(sqrt_N) * sqrt_N;
    std::cout << std::fixed << std::setprecision(12);
    std::cout
        << "Regular    Estimate of Pi = "
        << 4*double(inside_circle) / (sqrt_N*sqrt_N) << '\n'
        << "Stratified Estimate of Pi = "
        << 4*double(inside_circle_stratified) / (sqrt_N*sqrt_N) << '\n';
}
```
实验结果表明，分层方法结果更好，收敛更好；但是这种优势随着维度升高而降低。本书中不进行分层，但是可以在2D中进行分层。

## 三、一维蒙特卡洛积分

##### 3.1 $x^2$积分
一个经典的积分为：
$$
I = \int_0^2{x^2}\, \mathrm{d}x
$$

科学表达可以将其更改为：
$$
I = area(x^2, 0, 2)
$$

也可以将其写为：
$$
I = 2 \cdot average(x^2, 0, 2)
$$

使用cpp可以写为：
```cpp
#include "rtweekend.h"

#include <iostream>
#include <iomanip>
#include <math.h>
#include <stdlib.h>

int main() {
    int N = 1000000;
    auto sum = 0.0;
    for (int i = 0; i < N; i++) {
        auto x = random_double(0,2);
        sum += x*x;
    }
    std::cout << std::fixed << std::setprecision(12);
    std::cout << "I = " << 2 * sum/N << '\n';
}
```

通过概率采样，得到了正确的答案。这实际上与`ray_color()`函数的方法一样。
随机采样会引入噪声，可以通过增加采样数量减少噪声，但是过多的采样，可能会引入不正确的结果，因此引入重要性采样，即根据特定的pdf（概率密度）进行采样。

##### 3.2 密度函数
密度函数实际上就是将区域内分为很多个`bin`，就是概率直方图

##### 3.3 构建一个PDF
PDF函数应除以采样点的数值上，即：
```cpp
inline double pdf(double x) {
    return 0.5*x;
}

int main() {
    int N = 1000000;
    auto sum = 0.0;
    for (int i = 0; i < N; i++) {
        auto x = sqrt(random_double(0,4));
        sum += x*x / pdf(x);
    }
    std::cout << std::fixed << std::setprecision(12);
    std::cout << "I = " << sum/N << '\n';
}
```

##### 3.4 重要性采样
通过更改PDF将采样集中在我们需要的地方采样
总结：
- 有一个在[a, b]上积分的函数$ f(x) $
- 在[a, b]上选择一个非0的概率密度函数PDF $p$
- 获得最终的值$\frac{f(r)}{p(r)}$


## 球面方向上的MC积分
在所有方向上积分就是：
$$
\int cos^2(\theta)
$$


## 光线散射

##### 5.1 Albedo反照率
光线散射的概率：$A$
光线被吸收的概率：$1 - A$
这里A就代表反照率，会随着入射方向发生变化

##### 5.2 散射
大多数基于物理的渲染器中，使用波长代表光色，而不是RGB。
如果光确实发生散射，它将具有方向分布，我们可以将其描述为立体角上的PDF。我们将其称为它的散射
物体表面的颜色就可以表示为：
$$
Color = \int A \cdot s(direction) \cdot color(direction)
$$
现在，A和s()项都随视角方向发生变化，所以输出色彩就与入射方向有关。

##### 5.3 散射PDF
使用蒙特卡洛积分计算Color就为：
$$
Color = \frac{A \cdot s(direction) \cdot color(direction)}{p(direction)}
$$
$p(direction)$就是朝着方向direction随机采样的概率函数，对于Lambertian表面，我们已经隐式使用了$p=cosine$的概率密度函数。
对于Lambertian表面来说，它的$BRDF = A / \pi$

## 6. 重要性采样材质
PDF可以进行线性混合，我们在这里将$p(direction)$分为2种成分。
$$
p(direction) = \frac{1}{2} \cdot \mathrm{pLight}(direction) + \frac{1}{2} \cdot \mathrm{pSurface}(direction)
$$
所有PDF函数都会使得结果收敛为正确，

##### 6.1 重构Cornell Box
通过构建一个集中在灯光上的PDF来减少noise，对于Lambetian材质来说，我们将其的$p(direction) = cos(\theta)/\pi$

##### 6.2 随机半球采样
将$p(direction) = \frac{1}{2\pi}$

## 7. 生成随机的方向
找到哪个Cornell Box是正确的结果
##### 7.1 依据Z轴的随机方向
这里面的推导过程省略了，结论为：
$$
x = cos(2\pi r_1)\cdot 2 \sqrt{r_2(1-r_2)} \\
x = sin(2\pi r_1)\cdot 2 \sqrt{r_2(1-r_2)} \\
z = 1 - 2r_2
$$
单位球内的随机点，用cpp表示为：
```cpp
int main() {
    for (int i = 0; i < 200; i++) {
        auto r1 = random_double();
        auto r2 = random_double();
        auto x = cos(2*pi*r1)*2*sqrt(r2*(1-r2));
        auto y = sin(2*pi*r1)*2*sqrt(r2*(1-r2));
        auto z = 1 - 2*r2;
        std::cout << x << " " << y << " " << z << '\n';
    }
}
```


##### 7.2 在半球内单位采样
$$
\begin{array}{c}
\int \cos ^{3}(\theta) d A \\
=\int_{0}^{2 \pi} \int_{0}^{\pi / 2} \cos ^{3}(\theta) \sin (\theta) d \theta d \phi \\
=2 \pi \int_{0}^{\pi / 2} \cos ^{3}(\theta) \sin (\theta)=\frac{\pi}{2}
\end{array}
$$

## 8. 正交基
上一章，我们根据z轴随机生成向量，本章我们希望根据正交基来随机生成方向

##### 8.1 相关坐标系
目前使用的笛卡尔坐标系就是一种正交坐标系，当原点是$O$，单位向量分别为$x$,$y$,$z$时：
$$
Location\; is\; \mathbf{O} + 3\mathbf{x} - 2\mathbf{y} + 7\mathbf{z}
$$
当原点是$O^\prime$，单位向量分别为$u$,$v$,$w$时：
$$
Location\; is\; \mathbf{O^\prime} + 3\mathbf{u} - 2\mathbf{v} + 7\mathbf{w}
$$

##### 8.2 生成正交基
我们需要生成具有相对于$n$的集合分布的随机方向
构建方式如下：
选择一条非0的，与n不平行的向量$a$。叉乘获得向量$t$，然后获得向量$s$（广告牌原理）
$$
\mathbf{t} = \mathrm{unit\_ vector}(\mathbb{a} \times \mathbf{n}) \\
\mathbf{s} = \mathbf{t} \times \mathbf{n}
$$
为了选取$a$向量，我们需要增加一条逻辑
```cpp
if absolute(n.x > 0.9)
    a ← (0, 1, 0)
else
    a ← (1, 0, 0)
```

##### 8.3 构建ONB类
```cpp
#ifndef ONB_H
#define ONB_H

class onb {
    public:
        onb() {}

        inline vec3 operator[](int i) const { return axis[i]; }

        vec3 u() const { return axis[0]; }
        vec3 v() const { return axis[1]; }
        vec3 w() const { return axis[2]; }

        vec3 local(double a, double b, double c) const {
            return a*u() + b*v() + c*w();
        }

        vec3 local(const vec3& a) const {
            return a.x()*u() + a.y()*v() + a.z()*w();
        }

        void build_from_w(const vec3&);

    public:
        vec3 axis[3];
};


void onb::build_from_w(const vec3& n) {
    axis[2] = unit_vector(n);
    vec3 a = (fabs(w().x()) > 0.9) ? vec3(0,1,0) : vec3(1,0,0);
    axis[1] = unit_vector(cross(w(), a));
    axis[0] = cross(w(), v());
}

#endif
```
重构Lambertian材质，将随机方向替换
```cpp
virtual bool scatter(
    const ray& r_in, const hit_record& rec, color& alb, ray& scattered, double& pdf
) const override {
    onb uvw;
    uvw.build_from_w(rec.normal);
    auto direction = uvw.local(random_cosine_direction());
    scattered = ray(rec.p, unit_vector(direction), r_in.time());
    alb = albedo->value(rec.u, rec.v, rec.p);
    pdf = dot(uvw.w(), scattered.direction()) / pi;
    return true;
}
```

## 9. 直接采样灯光
重要性采样中，我们需要直接采样灯光

##### 9.1 获得光源的PDF
面积为$A\,$的光源，光源表面的PDF为$\displaystyle{\frac{1}{A}}$
如101所说，原方程中的积分更换为光源的积分，则对应关系为：
$$
\mathrm{d}\omega = \displaystyle{\frac{\mathrm{A} \cdot \cos(alpha)}{distance^2(p,q)}}
$$
在$\mathrm{d}\omega$中采样和在$\mathrm{d}A$必须相同，我们有：
$$
p(direction) \cdot \frac{\mathrm{d}A \cdot \cos(alpha)}{distance^{2}(p,q)}=p_{q}(q) \cdot \mathrm{d} A=\frac{\mathrm{d} A}{A}
$$
所以：
$$
p(direction) = \frac{distance^{2}(p,q)}{\cos(alpha)\cdot A}
$$

##### 9.2 光源采样
只采样光源似乎是可行的

##### 9.3 光源转换为单向光
将光源添加一个单向光判断


## 10. 混合密度
我们使用了一个依据$\cos(\theta)$的PDF，和一个只采样光源的PDF，我们对他们进行混合
##### 10.1 平均灯光与反射
$$
\mathrm{mixture}_{pdf}(direction) = \displaystyle{\frac{1}{2}\mathrm{reflection}_{\mathrm{pdf}}(direction) + \frac{1}{2}\mathrm{light}_{\mathrm{pdf}}(direction) }
$$

但是在，光源方向上，也可以是反射方向的PDF。重构PDF得到一个不错的结果

##### 10.2 朝着hittalbe采样
##### 10.3 混合PDF类




























