[In Next Weekend教程解析](Analysis/NextWeek.md)
[In One Weekend教程解析](Analysis/InOneWeekend.md)


## 过程中需要复习的知识
##### 1. `static_cast`
```cpp
1. static_cast是一个c++运算符，功能是把一个表达式转换为某种类型，但没有运行时类型检查来保证转换的安全性。
2. 向下取值
```


##### 2. `inline`内联函数
```cpp
在 c/c++ 中，为了解决一些频繁调用的小函数大量消耗栈空间（栈内存）的问题，特别的引入了 inline 修饰符，表示为内联函数。

栈空间就是指放置程序的局部数据（也就是函数内数据）的内存空间。

在系统下，栈空间是有限的，假如频繁大量的使用就会造成因栈空间不足而导致程序出错的问题，如，函数的死循环递归调用的最终结果就是导致栈内存空间枯竭。
```
https://www.runoob.com/w3cnote/cpp-inline-usage.html


> 好多采样都是50，所以跟图片跟教程有出入

##### 3. 输出png格式
http://chanhaeng.blogspot.com/2018/12/how-to-use-stbimagewrite.html

##### 4. 加载模型

##### 5. TBB多线程

##### 6. 加速路口检测的八叉树数据结构

##### 7. 生成的Ray
GAMES101中，生成的Ray是[-1, 1]范围内的，但是在One Weekend中，Ray是[0, 1]的，归一化的操作在Image设置中。

## 参考资料
[知乎大佬笔记](https://www.zhihu.com/column/c_1504519612624035840)

