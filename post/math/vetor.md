# 向量(vector)
也称矢量, 在数学中，矢量常采用更为抽象的矢量空间（也称为线性空间）来定义，而定义具有物理意义上的大小和方向的矢量概念则需要引进了范数和内积的欧几里得空间。

## 矢量表示方法
- 符号: 适量有大小和方法, $\vec{a}$
- 与纸面垂直的符号:
    1. 圆圈中带点的记号（⊙）表示由纸下方指向纸上方的矢量，
    2. 而圆圈中带叉的记号（⊗）则表示由纸的上方指向纸下方的矢量。
- 值表示: $(a, b, c) = a\vec{i} + b\vec{j} + c\vec{k}$

## 矢量种类
按空间分:
1. 欧几里得空间: 物理学和几何学中的矢量被称为欧几里得矢量或几何矢量,包括大小和方向
2. 线性空间: 线性代数中，矢量是所谓矢量空间中的基本构成元素

按自由度分:
1. 自由矢量: 不区分起点和终点, 数学一般默认研究这种
1. 非自由矢量: 区分起点和终点, 会被视为不同的矢量

# 矢量运算

## 数量积(内积, dot product)
数量积也叫点积/内积，它是矢量与矢量的乘积，其结果为一个标量（非矢量）。几何上，数量积可以定义如下:

$A·B = |A||B|cos(θ)$    
$A·B = A_B|B|=B_A|A|$

### 标量投影
欧氏空间中向量A在向量B上的标量投影是指

$A_B=|\mathbf A|\cos\theta$,

### 性质
点积满足交换律： $\vec{a} \cdot \vec{b} = \vec{b} \cdot \vec{a} \;$

点积满足分配律: $\vec{a} \cdot (\vec{b} + \vec{c}) = \vec{a} \cdot \vec{b} + \vec{a} \cdot \vec{c}$ 

>相当于b在a上的投影叠加c在a上的投影(等于b+c在a上的投影), 再乘以a的长度

## 向量积(vetor product)
叉积(cross product), 也称作外积（英语：Outer product）或向量积（英语：Vector product）

叉积可以定义为：

$a \times b =|a||b|\sin \theta \;\hat{n}$

> 在这里 ø 表示 a与b 之间的角度。而 $\hat{n}$ 是一个与a,b所构成的平面垂直的单位向量。

![math/vetor-1.png](/img/math/vetor-1.png)

### 性质
1. 几何性质: 与点积类似a乘b, 相当于b向a的垂线(即距离, 平行分量抵消了), 再乘以a的长度
2. $a \times {b} =0$ 當且僅當 a 平行於 b 
3. $a×b=-b×a$（反交换律）
4. $a×(b+c)=a×b+a×c$（加法的左分配律）
5. $(a+b)×c=-c×(a+b)=a×c+b×c$（加法的右分配律）
6. $a×b+c×d=(a-c)×(b-d)+a×d+c×b$

#### 图形证明分配律
$z=a×(b+c)=a×b+a×c$（加法的左分配律）

1. 设a的模长是1, b重直于a(平行分量不影响大小和方向), c也垂直于a(同样忽略平行分量)
2. 大小相等: |z_b|=|b|, |z_c|=|c|, 重点: |z_(b+c)|=|z_b+z_c|
3. 方向相同: z_(b+c) 的方向完全相当于$z_b+z_c$

![math/vetor-2.png](/img/math/vetor-2.png)

### 三維坐標
通过分配律可以得到
![math/vetor-3.png](/img/math/vetor-3.png)

## 混合积(三重积)
三个向量\vec{a}、\vec{b}和\vec{c}的混合积定义为，物理意義為三向量始於同點時所構成的體積：

$\vec{a}\cdot(\vec{b}\times \vec{c})=
\vec{b}\cdot(\vec{c}\times \vec{a})=
\vec{c}\cdot(\vec{a}\times \vec{b})$