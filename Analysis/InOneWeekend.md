> 原文链接：https://raytracing.github.io/books/RayTracingInOneWeekend.html

### 1. Overview
### 2. Output an Image
#### 2.1 图片格式
- 本教程中将渲染图输出为ppm格式
- ppm文件组成简单说明：文件头由3行组成：
  1. 文件类型（p2/p6/p3）在PPM彩色文件(P3)中，每个象素占12个字节，依次为红绿蓝通道，每个通道为4字节ASCII码表示的整数，高字节在前。左上角为坐标原点。
  2. 图像的宽度和高度
  3. 最大的像素值为255
#### 2.2 输出图片
- 通过重定向运算符实现 `>`
```cpp
build\Release\inOneWeekend.exe > image.ppm
```
- 输出的图片可以在visual studio code中通过ppm插件观察

#### 2.3 添加进度指示
- 通过`std`库中的`cerr`与`flush`实现
- 每生成一行像素，输出当前剩余的像素行数
- 结束后输出Done
```cpp
    for (int j = image_height-1; j >= 0; --j) {
        std::cerr << "\rScanlines remaining: " << j << ' ' << std::flush;
        for (int i = 0; i < image_width; ++i) {
            auto r = double(i) / (image_width-1);
            auto g = double(j) / (image_height-1);
            auto b = 0.25;

            int ir = static_cast<int>(255.999 * r);
            int ig = static_cast<int>(255.999 * g);
            int ib = static_cast<int>(255.999 * b);

            std::cout << ir << ' ' << ig << ' ' << ib << '\n';
        }
    }

    std::cerr << "\nDone.\n";
```

### 3. vec3类
#### 3.1 变量和方法
- 自己构建三维向量运算库
  - 默认构造`vec3() 为 {0，0，0}`
  - 构造函数`vec3(double e0, double e1, double e2) : e{e0, e1, e2} {}`
  - 提取分量`x()` `y()` `z()`
  - 减号运算符重载
  - 加号运算符重载
  - 索引[]运算符重载
  - 乘法运算符重载
  - 除法运算符重载
  - 求模长
  - 求3个分量平方和
  - 为vec3设置别名
```cpp
// Type aliases for vec3
using point3 = vec3;   // 3D point
using color = vec3;    // RGB color
```
#### 3.2 vec3 Utility Functions
- 根据上下文观察，猜测是根据类中重载的运算符，写了一些后面会用到的函数，添加在头文件的第二部分，vec3类外
> inline关键字：可以解决一些频繁调用的函数大量消耗栈空间（栈内存）的问题。
> [宏，inline，static函数的区别](https://blog.csdn.net/Mr_H9527/article/details/100596854)

#### 3.3 Color 实用函数
- 通过`vec3.h`文件实现颜色写入的函数
`void write_color(std::ostream &out, color pixel_color)`
- 通过`vec.h` 和 `color.h` 优化`main.cpp`


### 4. Rays, a Simple Camera, and Background
#### 4.1 Ray 类
- 使用`p = A + tb`的公式来描述射线
- `ray.h`中包含
  - 空的构造函数
  - `ray`的`orig`与`dir`赋值构造函数
  - 在`t`时刻的成员函数`at()`

![相机空间示意图](../Texture/Camera%20Geometry.jpg)

#### 4.2 Sending Rays Into the Scene
- 构建Image属性——>获得U、V坐标
- 构建相机属性——>获得ray的direction
- `ray_color(const ray &r)`函数，根据射线进行着色
```cpp
auto t = 0.5*(unit_direction.y() + 1.0); // 将y方向归一化
return (1.0-t)*color(1.0, 1.0, 1.0) + t*color(0.5, 0.7, 1.0); // 根据归一化之后的坐标在蓝色与白色之间插值
```

### Adding a Sphere
#### 5.1 Ray-Sphere Intersection
- 通过隐式几何表达式构建相交方程
- `t2b⋅b+2tb⋅(A−C)+(A−C)⋅(A−C)−r2=0`，将问题转化为求方程的根
#### 5.2 Creating Our First Raytraced Image
- `hit_sphere()`函数实现求根，返回boolen
- 根据boolen决定像素颜色

### 6. Surface Normals and Multiple Objects
#### 6.1 根据表面法线着色
- 使用表面法线着色的时候，需要归一化为单位向量
- 将`hit_sphere()`函数返回为较小的根（较大的根不在视线范围内），如果根不存在，则返回-1
- 在`ray_color()`中，将根大于0时，根据法线着色
> 目前的color(x,y,z)中各个分量的范围为[0,1]
#### 6.2 简化射线与几何相交的方程
- 根据方程将`b`转化为`half_b`
#### 6.3 为击中物体构造抽象类
- 创建`hittalbe.h`头文件来记录被击中的物体
- 使用`hit_record`结构体记录交点信息，交点`p`，交点法线`normal`，交点的`t`值
- `hittable`类中声明hit函数，用来接收光线的命中函数
- 创建`sphere.h`头文件，声明`sphere`类作为`hittable`类的子类
- 为`sphere`类创建构造函数（中心，半径）
- 在`sphere`类中实现`hit`函数，找到可接受范围内的最近的根，hit为继承父类中的`hit`函数
#### 6.4 面朝向
- 确定面朝向（法线方向）十分重要，法线决定了着色
- 根据`ray`方向与法线方向之间的点积决定法线朝向
- 在`hit_record`结构体中添加`front_face`成员变量，并实现法线判断
- 在`sphere.h`头文件中实现法线判断

#### 6.5 击中物体列表
- 创建`hittable_list.h`头文件
- 构建`hittable_list`类，声明构造函数，使用`shared_ptr`指针指向`vector`中物体
- `vector`中储存的是指向`hittable`类型的`shared_ptr`
```cpp
class hittable_list : public hittable {
    public:
        hittable_list() {}
        hittable_list(shared_ptr<hittable> object) { add(object); }

        void clear() { objects.clear(); }
        void add(shared_ptr<hittable> object) { objects.push_back(object); }

        virtual bool hit(
            const ray& r, double t_min, double t_max, hit_record& rec) const override;

    public:
        std::vector<shared_ptr<hittable>> objects;
};
```
- 实现`hit`函数，遍历`vector`中的所有物体，只要有击中的，就返回`true`

#### 6.6 新的C++特性
- 智能指针`shared_ptr`
- `vector`容器

#### 6.7 常用变量与实用函数
- 常用变量：无穷大`infinity`，`pi`
- 实用函数：`degrees_to_radians`角度转化为弧度
>疑问：Listing 24中，ray_color(r, world)中的world为什么是hittable_list类型的

### 7. 反走样
#### 7.1 随机数使用函数
- 在`rtweekend.h`中声明随机函数`random_double()`和`random_double(double min, double max)`函数
- 需要`#include <cstdlib>`头文件
- 或者使用c++的内置随机数，需要引入`#include <random>`头文件

#### 7.2 多次采样生成像素
- 创建`camera.h`头文件
- 创建`camera`类
- 在`rtweekend.h`中创建`clamp()`函数
- 在生成颜色的时候，多次采样加和，并除以采样次数来实现多次采样
```cpp
color pixel_color(0, 0, 0);
    for (int s = 0; s < samples_per_pixel; ++s) {
        auto u = (i + random_double()) / (image_width-1);
        auto v = (j + random_double()) / (image_height-1);
        ray r = cam.get_ray(u, v);
        pixel_color += ray_color(r, world);
    }
    write_color(std::cout, pixel_color, samples_per_pixel);
```

### 8.漫反射材质
#### 8.1 简单的漫反射材质
- 漫反射产生的原因是因为光线随机的发射、反射
- 构建反射向量的方法是，在表面切点处进行计算
- 通过构建p+n单位球进行计算
![](../Texture/Diffuse.jpg)
- 在单位球内构建随机反射向量，使用拒绝方法
- 在`ray_color()`中迭代构建射线颜色，并每次只反射50%
```cpp
if (world.hit(r, 0, infinity, rec)) {
    point3 target = rec.p + rec.normal + random_in_unit_sphere(); // s点
    return 0.5 * ray_color(ray(rec.p, target - rec.p), world); // 递归计算
}
```
#### 8.2 限制递归次数
- 目前会产生无限循环
- 添加递归限制
```cpp
if (world.hit(r, 0, infinity, rec)) // 如果击中物体，才会着色
{
    point3 target = rec.p + rec.normal + random_in_unit_sphere();
    return 0.5 * ray_color(ray(rec.p, target - rec.p), world, depth-1); // 如果击中物体，那么就返回0.5倍，所以弹射越多，越黑
}
```
> 目前的着色是光线能否击中物体的概率，与背景颜色的插值？

#### 8.3 Gamma矫正
- 通过Gamma矫正，对抗显示器的差异
```cpp
// Divide the color by the number of samples and gamma-correct for gamma=2.0.
auto scale = 1.0 / samples_per_pixel;
r = sqrt(scale * r);
g = sqrt(scale * g);
b = sqrt(scale * b);
```

#### 8.4 解决阴影粉刺
```cpp
if (world.hit(r, 0.001, infinity, rec))
```

#### 8.5 真正的Lambertian反射
- 前面的拒绝随机分布与法线夹角有关，分布关系为cos的立方
- 真正的L分布更均匀，关系为cos
- 将在球体内生成随机点改为表面生成随机点
- 将单位随机球使用`unit_vector`处理，使其生成在球体表面上

#### 8.6 另一种反射分布原则
- 反射与法线无关
```cpp
vec3 random_in_hemisphere(const vec3& normal) {
    vec3 in_unit_sphere = random_in_unit_sphere();
    if (dot(in_unit_sphere, normal) > 0.0) // In the same hemisphere as the normal
        return in_unit_sphere;
    else
        return -in_unit_sphere;
}
```
- 与最初的区别就是在交点p处随机生成点，而最初是在p+n处生成点
- 每一种分布都值得研究
### 9. Metal
#### 9.1 材质抽象类
- 材质类应该描述散射情况，提供输入光线，描述交点着色属性，描述衰减、描述散射
- 散射是指镜面反射或者diffuse反射

#### 9.2 描述光线与物体相交
- `hit_record`结构体增加`mat_ptr`指针
- `sphere`类中增加材质指针，用来描述光线与物体的交点

#### 9.3 构建光照散射与反射
- 创建`lambertian`材质
- 描述散射方向，散射射线，衰减
- 散射方向为`rec.normal + random_unit_vector()`，有可能出现为0的情况，因此增加判断
```cpp
// Catch degenerate scatter direction
if (scatter_direction.near_zero())
    scatter_direction = rec.normal;
```

#### 9.4 镜像反射
- 在`vec3.h`中增加反射函数
```cpp
if (world.hit(r, 0.001, infinity, rec)) {
    ray scattered; // 声明散射光线
    color attenuation; // 声明散射颜色
    if (rec.mat_ptr->scatter(r, rec, attenuation, scattered)) // 判断是否有散射产生
        return attenuation * ray_color(scattered, world, depth-1);
    return color(0,0,0);
}
```