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
