## 常用命令

### 基本操作

```matlab
help helpwin clc clear
% F5 保存并运行
```

### 向量和矩阵

``` matlab
% matlab 的矩阵是按列存储的
A = fix(rand(3)*10)  % 3阶随机矩阵，向 0 取整
inv(A)  % 方阵的逆
A'      % 方阵的转置
det(A)  % 方阵A的行列式
[V, D] = eig(A) % 方阵的特征向量与特征值，每列代表一个
zeros(m, n)  ones(m, n) eye(m, n) % m行n列的全0，全1，单位矩阵
A(end:-1:i1, :)  % 逆序提取 A 的第 i1 ~ 最后一行
A(i1:i2, :) = [] % 删除 A 的 i1 ~ i2 行
sum(A) % 对每一列求和，得一个行向量
sum(A, 2) % 对每一行求和
max(A) min(A) % 与 sum 用法类似
find(A) % 找 A 中不为 0 的元素的下标
find(A == 2) % 找 A 中等于 2 的元素的下标
```

### 分支判断等

```matlab
% if switch while 等最后要加一行 end
if 3 > 2
	disp('3 > 2')
end
```

### 符号运算

+ 微分

  ```matlab
  syms x y;
  z = x*x*y + sin(x*y);
  dx = diff(z, x, 1);  % 对 x 求 1 阶偏导
  dxy = diff(dx, y, 1);
  ```

+ 积分

  ```matlab
  syms x a b
  int(x*x*sin(x), x)  % 求不定积分
  int(exp(x)*x*x, x, a, b) % 求 [a, b] 的定积分
  ```

+ 求解方程（组）

  ```matlab
  syms x y
  [a, b] = solve(2*x+4*y==1, 5*x-6*y==4); % x = a, y = b
  ```

### 其他操作

```matlab
% matlab 很多接口与 c 类似
% 字符串操作，还有 strcmp，sprintf, sscanf等
num2str(A)
str2num(A)
% 文件操作
fd = fopen("in.txt", "rt")
fclose(fd)
fprintf(fd, ...)

```

