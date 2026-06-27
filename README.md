# 贝塞尔曲线交互绘制实验
## 实验简介
本项目使用 Python + Taichi 高性能计算框架，实现了一个交互式贝塞尔曲线绘制系统。通过 De Casteljau 算法实时计算曲线坐标，并利用 GPU 并行加速进行像素级光栅化渲染，展示了计算机图形学中曲线生成、GPU 编程和交互设计的核心原理。

## 实验目标
+ **理解贝塞尔曲线的几何意义**：掌握控制点对曲线形状的影响
+ **实现 De Casteljau 算法**：通过递归线性插值计算曲线上的点
+ **<font style="color:rgb(15, 17, 21);">掌握光栅化基础</font>**<font style="color:rgb(15, 17, 21);">：在像素缓冲区中直接操作和点亮像素</font>
+ **<font style="color:rgb(15, 17, 21);">熟悉图形交互</font>**<font style="color:rgb(15, 17, 21);">：处理鼠标点击与键盘事件，实现实时绘制</font>

<font style="color:rgb(15, 17, 21);"></font>

## <font style="color:rgb(15, 17, 21);">核心算法：De Casteljau</font>
### <font style="color:rgb(15, 17, 21);">数学原理</font>
<font style="color:rgb(15, 17, 21);">贝塞尔曲线由一组控制点</font>$ P_0,P_1,.......,P_{n-1} $<font style="color:rgb(15, 17, 21);">定义，参数 t∈[0,1]</font>_<font style="color:rgb(15, 17, 21);">t</font>_<font style="color:rgb(15, 17, 21);">∈[0,1] 控制曲线上的位置。</font>

<font style="color:rgb(15, 17, 21);">De Casteljau 算法通过递归线性插值计算曲线点：</font>

1. **<font style="color:rgb(15, 17, 21);">第一层插值</font>**<font style="color:rgb(15, 17, 21);">：对相邻控制点进行线性插值</font>

<font style="color:rgb(15, 17, 21);">  		   </font>$ P_i^{(1)}=(1−t)⋅P_i+t⋅P_{i+1},i=0,1,...,n−2 $

2. **<font style="color:rgb(15, 17, 21);">递归计算</font>**<font style="color:rgb(15, 17, 21);">：对得到的新点重复插值，直到只剩 1 个点</font>

<font style="color:rgb(15, 17, 21);">  		  </font>$ P_i^{(k)}=(1−t)⋅P_i^{(k−1)}+t⋅P_{i+1}^{(k−1)} $

3. **<font style="color:rgb(15, 17, 21);">终止条件</font>**<font style="color:rgb(15, 17, 21);">：最终得到的唯一点即为曲线在参数 t</font>_<font style="color:rgb(15, 17, 21);">t</font>_<font style="color:rgb(15, 17, 21);"> 处的位置</font>

### <font style="color:rgb(15, 17, 21);">代码实现</font>
```plain
def de_casteljau(points, t):
    """纯 Python 递归实现 De Casteljau 算法"""
    if len(points) == 1:
        return points[0]
    next_points = []
    for i in range(len(points) - 1):
        p0 = points[i]
        p1 = points[i+1]
        x = (1.0 - t) * p0[0] + t * p1[0]
        y = (1.0 - t) * p0[1] + t * p1[1]
        next_points.append([x, y])
    return de_casteljau(next_points, t)
```



## <font style="color:rgb(15, 17, 21);">光栅化流程</font>
### 坐标映射
<font style="color:rgb(15, 17, 21);">曲线点坐标是归一化浮点数 [0, 1]，需映射到屏幕像素坐标</font>

```plain
x_pixel = int(pt[0] * WIDTH)
y_pixel = int(pt[1] * HEIGHT)
```

### 像素点亮
<font style="color:rgb(15, 17, 21);">在 800×800 的像素缓冲区中，将对应位置设为绿色：</font>

```plain
pixels[x_pixel, y_pixel] = ti.Vector([0.0, 1.0, 0.0])
```

### CPU并行绘制
<font style="color:rgb(15, 17, 21);">利用 Taichi 的 </font>`<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">@ti.kernel</font>`<font style="color:rgb(15, 17, 21);"> 实现 GPU 并行加速：</font>

```plain
@ti.kernel
def draw_curve_kernel(n: ti.i32):
    for i in range(n):  # GPU 自动并行执行
        pt = curve_points_field[i]
        x_pixel = ti.cast(pt[0] * WIDTH, ti.i32)
        y_pixel = ti.cast(pt[1] * HEIGHT, ti.i32)
        if 0 <= x_pixel < WIDTH and 0 <= y_pixel < HEIGHT:
            pixels[x_pixel, y_pixel] = ti.Vector([0.0, 1.0, 0.0])
```



## 实验效果展示
下列是一些实验效果动图展示，从中可以看到单曲线绘制、二次曲线绘制、高阶曲线绘制以及实时交互。
<img width="804" height="848" alt="work3" src="https://github.com/user-attachments/assets/7ccb80bb-99d0-4ef0-a8cd-3e9d81ecac1b" />
<img width="804" height="848" alt="work3(1)" src="https://github.com/user-attachments/assets/97ab14c1-1c00-4eea-8a16-ac5de2bfb959" />
<img width="804" height="848" alt="work3 (2)" src="https://github.com/user-attachments/assets/d4610077-dae4-4b6d-8828-a505bbb00b2f" />







