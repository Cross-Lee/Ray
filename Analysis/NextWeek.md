> 原文链接：https://raytracing.github.io/books/RayTracingTheNextWeek.html
### 1. Overview
- 添加纹理
- 体积（雾）
- 矩形
- 实例
- 灯光
- BVH加速


### 2. Motion Blur
- 模糊可能由于多次采样而获得，但是运动模糊是由于长曝光而产生
#### 2.1 SpaceTime 光线追踪介绍
- 快门打开时的随机时间发送光线来获得随机估计
- 为`ray`添加`time`属性

#### 2.2 更新相机来模拟运动模糊
- 为相机添加 `time1` 和 `time2` 属性，表示相机的快门时间为 `t1` 到 `t2`

#### 2.3 添加移动的spheres
- `spheres` 会在 `t1` 和 `t2` 时刻从 `center1` 运动到 `center2`
- 创建 `moving_sphere.h` 头文件 声明 `moving _sphere::hit` 函数

#### 2.4 找到相交的时间
- 为每一种材质添加 `ray` 发射的时间

#### 2.5 整合代码
- 创建动态模糊场景

### 3. BVH包围盒
- 重构了 `hittable` 代码
- 判断与物体的求交是光追的主要性能瓶颈
- 划分时间 / 划分空间

#### 3.1 关键Idea
- 创建包围盒减少判断时间

#### 3.2 包围盒阶级
- 二叉树结构

#### 3.3 轴对齐包围盒
- 建立ray与包围盒的关系等式

#### 3.4 射线与AABB盒子相交
- 比较相交时间t是否与盒子范围有重叠
```cpp
#ifndef AABB_H
#define AABB_H

#include "rtweekend.h"

class aabb {
    public:
        aabb() {} // 默认构造函数
        aabb(const point3& a, const point3& b) { minimum = a; maximum = b;} // 最大最小值构造函数

        // 成员变量接口
        point3 min() const {return minimum; } 
        point3 max() const {return maximum; }

        bool hit(const ray& r, double t_min, double t_max) const {
            for (int a = 0; a < 3; a++) {
                auto t0 = fmin((minimum[a] - r.origin()[a]) / r.direction()[a],
                               (maximum[a] - r.origin()[a]) / r.direction()[a]);
                auto t1 = fmax((minimum[a] - r.origin()[a]) / r.direction()[a],
                               (maximum[a] - r.origin()[a]) / r.direction()[a]);
                t_min = fmax(t0, t_min);
                t_max = fmin(t1, t_max);
                if (t_max <= t_min) // 如果t_max <= t_min，说明没有交点
                    return false;
            }
            return true;
        }

        point3 minimum;
        point3 maximum;
};

#endif
```
- 只有进入所有的面，才算进入AABB盒子
- 只要离开任意一个界面，就算离开AABB盒子
- Tenter = max{Tmin}; Texit = min{Tmax}
- ray 与 AABB有交点，当且仅当 T_enter < T_exit && T_exit >= 0

#### 3.5 优化的 AABB Hit方法

#### 3.6 为hittables类型创建bounding_box方法
- 为`sphere`构建AABB包围盒
- 为`moving_sphere`构建AABB包围盒

#### 3.7 为一系列物体创建包围盒
```cpp
bool hittable_list::bounding_box(double time0, double time1, aabb& output_box) const {
    if (objects.empty()) return false;

    aabb temp_box;
    bool first_box = true;

    for (const auto& object : objects) {
        if (!object->bounding_box(time0, time1, temp_box)) return false; // 将object 储存到temp_box中？
        output_box = first_box ? temp_box : surrounding_box(output_box, temp_box);
        // - 第一个物体创建temp_box
        // - 第二个物体建立surrounding_box
        // - 后续一直创建surrounding_box
        first_box = false;
    }

    return true;
}
```

#### 3.8 BVH结点类
- 创建类储存BVH树

#### 3.9 实现BVH二叉树
```cpp
bvh_node::bvh_node(
    std::vector<shared_ptr<hittable>>& src_objects,
    size_t start, size_t end, double time0, double time1
) {
    auto objects = src_objects; // Create a modifiable array of the source scene objects

    int axis = random_int(0,2);
    auto comparator = (axis == 0) ? box_x_compare
                    : (axis == 1) ? box_y_compare
                                  : box_z_compare;

    size_t object_span = end - start;

    if (object_span == 1) {
        left = right = objects[start];
    } else if (object_span == 2) {
        if (comparator(objects[start], objects[start+1])) {
            left = objects[start];
            right = objects[start+1];
        } else {
            left = objects[start+1];
            right = objects[start];
        }
    } else {
        std::sort(objects.begin() + start, objects.begin() + end, comparator);

        auto mid = start + object_span/2;
        left = make_shared<bvh_node>(objects, start, mid, time0, time1);
        right = make_shared<bvh_node>(objects, mid, end, time0, time1);
    }

    aabb box_left, box_right;

    if (  !left->bounding_box (time0, time1, box_left)
       || !right->bounding_box(time0, time1, box_right)
    )
        std::cerr << "No bounding box in bvh_node constructor.\n";

    box = surrounding_box(box_left, box_right);
}
```

#### 3.10 盒子比较函数
```cpp
inline bool box_compare(const shared_ptr<hittable> a, const shared_ptr<hittable> b, int axis) {
    aabb box_a;
    aabb box_b;

    if (!a->bounding_box(0,0, box_a) || !b->bounding_box(0,0, box_b))
        std::cerr << "No bounding box in bvh_node constructor.\n";

    return box_a.min().e[axis] < box_b.min().e[axis];
}


bool box_x_compare (const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
    return box_compare(a, b, 0);
}

bool box_y_compare (const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
    return box_compare(a, b, 1);
}

bool box_z_compare (const shared_ptr<hittable> a, const shared_ptr<hittable> b) {
    return box_compare(a, b, 2);
}
```

### 4. Solid Texture
- 纹理的产生可能是程序化生成，也可以是图像查找

#### 4.1 texure.h类
- 增加 `hit_record` 中击中点的uv坐标

#### 4.2 球体的纹理坐标
- 建立球上的点 与 uv坐标的映射

#### 4.3 创建棋盘格纹理

#### 4.4 使用棋盘格纹理渲染球体

### 5. Perlin 噪声
#### 5.1 使用随机数创建blocks
#### 5.2 平滑噪声结果
#### 5.3 使用Hermitian平滑改进
#### 5.4 降低噪声频率
#### 5.5 使用随机向量
#### 5.6 引入湍流
#### 5.7 调整相位

### 6 图像贴图映射
- 引入`stb_image`库
#### 6.1 储存Image数据
- 构造 `image_texture` 类

#### 6.2 使用Image贴图
- 创建地球

### 7. 矩形和灯光
- 创建有形状的灯光，就是创建会发光的材质

#### 7.1 自发光材料
- 创建自发光材料，不需要发光的父类中`emit`属性设置为黑色

#### 7.2 添加背景
- 将背景默认设置为黑色

#### 7.3 添加矩形物体
- 矩形面片的厚度为0，用k代替
- 创建 `aacect.h` 头文件
- aarect中，`t = (k-r.origin().z()) / r.direction().z()`
- 矩形面片中，我们需要将其想象为一个厚度很小的盒子，所以
```cpp
// The bounding box must have non-zero width in each dimension, so pad the Z
// dimension a small amount.
output_box = aabb(point3(x0,y0, k-0.0001), point3(x1, y1, k+0.0001));
```
- 这里的k值实际上就是z值

#### 7.4 将物体转变为灯光
- 材质类中，我们需要添加一个发出的函数（我们也可以将它添加到 hit_record 中——这是设计品味的问题）。和背景一样，它只是告诉光线它是什么颜色，不进行反射。


### 8. 实例


### 9. 体积
- 体积问题可以转化为随机的曲面

#### 9.1 常量密度介质
- 体积的密度代表为散射的可能性
- 