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
```CPP
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

### vec3类
- 自己构建三维向量运算库
  - 默认构造vec3() 为 {0，0，0}
#### 