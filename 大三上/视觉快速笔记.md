# 视觉 - 实验

[TOC]

##### matlab函数

- []：用于存储矩阵和向量。行间元素可以用逗号或空格分开；列间使用分号分开

- ()：用于引用数组的元素。X([1,2,3])就是X的头三个元素

- varagin：提供了一种函数可变参数列表机制，允许调用者调用该函数时根据需要来改变输入参数的个数

- persistent：标志全局变量（仅在函数中可见）

- nargin：判断输入变量个数

- makelut, applylut与lut：lut是查找表，是预先计算每个可能邻域形状的像素值，然后把这些值储存到一个向量中；makelut用于构造lut查找表；applylut将查找表应用于图像

- norm()：获取矩形范数

- ndims()：获取维度（一维二维等）

- end：相对于break。条件和循环语句都需要

- double：将数据集强制转为double类型

- zeros：创建全0矩阵
  - zeros(n)：返回n*n的全零矩阵
  - zeros(n1,n2)：返回n1*n2的全零矩阵
  
- sum()：按列添加

- \~：表示忽略输出参数（等号右边的值只会赋给非~的值）

- ~=：不等于

- :  ：所有（如`d(top, :)`表示top行的所有数据）

- rgb2gray()：将RGB图像转为灰度图像（此时每个点R=G=B=亮度值）

- im2bw()：把灰度图像转换成二值图像（只有纯黑（0）和纯白（255）两种颜色的图像）

- fspecial()：生成滤波器（也叫算子）

- imfilter()：使用滤波器对图像进行滤波

- imcrop()：剪切图像，四个参数为左上右下

- .* 、 .\：矩阵相乘、相除

- [filepath, name, ext] = fileparts(filename)：返回指定文件的路径名称、文件名和扩展名

- fullfile()：合成完整文件名

- imread()：根据文件名获取照片

- [r,c] = find(sq==1)：找出居正中为1的值的全部索引（行和列）

- rectangle()：画长方形

- plot()：画线

- repmat(A, m, n)：将矩阵A复制m*n块

- .^  ：幂运算

- stem(x,y)：绘制序列

- exp(x)：e^x^

- filter(b,a,x)：

  - 一维数字滤波器，用于滤除向量X中的数据
  - 可用于实现差分方程。a表示差分方程输出y的系数，b表示输入x的系数，x表示输入序列

- conv(x,h)：用来实现卷积。对x序列和h序列进行卷积

- impz(b,a,N)：用来实现冲激响应。a表示差分方程输出y的系数，b表示输入x的系数，x表示输出序列的个数

- axis([xmin xmax ymin ymax])：设置坐标轴范围

- bwmorph()：对二值图像进行形态学操作

- 循环

  ```matlab
  for i = 1 : length(sq)
  	% ..
  end
  ```

  





##### 获取图像模块

```matlab
% 获取图像
[filename, pathname, filterindex] = uigetfile({'*.jpg;*.tif;*.png;*.gif','All Image Files';...
    '*.*','All Files' }, '选择待处理图像', ...
    'images\car.jpg');

% 图像加载、图像大小适应
if filename == 0
    return;
end
file = fullfile(pathname, filename); 
Img = imread(file); 
[y, ~, ~] = size(Img); 
if y > 800
    rate = 800/y;
    Img1 = imresize(Img, rate);
else
    Img1 = Img;
end

% 将图像加载到ui中
axes(handles.axes1);
imshow(Img1); title('原图像', 'FontWeight', 'Bold');
handles.Img = Img;
handles.file = file;
guidata(hObject, handles);
```



##### 标定特定区域模块

- 找出车牌区域
- 行过滤》列过滤》分割结果

```matlab
function [Plate, bw, Loc] = Pre_Process(Img, parm, flag)

if nargin < 1
    Img = imread(fullfile(pwd, 'images/car.jpg'));
end
if nargin < 2 || isempty(parm)
    if size(Img, 2) > 900
        parm = [0.35 0.9 90 0.35 0.7 90 2];
    end
    if size(Img, 2) > 700 && size(Img, 2) < 900
        parm = [0.6 0.9 90 0.6 0.8 90 0.5];
    end
    if size(Img, 2) > 500 && size(Img, 2) < 700
        parm = [0.5 0.54 50 0.6 0.7 50 3];
    end
    if size(Img, 2) < 500
        parm = [0.8 0.9 150 0.8 0.9 150 3];
    end
end
if nargin < 3
    flag = 1;
end
% 以下语句所有条件皆会运行
% 图片大小适应
I = Img;
[y, ~, ~] = size(I); 
if y > 800
    rate = 800/y;
    I = imresize(I, rate);
end
[y, x, ~] = size(I); 

% 行过滤
myI = double(I); 
bw1 = zeros(y, x);
bw2 = zeros(y, x);
Blue_y = zeros(y, 1);
for i = 1 : y
    for j = 1 : x
        rij = myI(i, j, 1)/(myI(i, j, 3)+eps);
        gij = myI(i, j, 2)/(myI(i, j, 3)+eps);
        bij = myI(i, j, 3);
        if (rij < parm(1) && gij < parm(2) && bij > parm(3)) ...
                || (gij < parm(1) && rij < parm(2) && bij > parm(3))
            Blue_y(i, 1) = Blue_y(i, 1) + 1; 
            bw1(i, j) = 1;
        end
    end
end

[~, MaxY] = max(Blue_y);
Th = parm(7);
PY1 = MaxY;
while ((Blue_y(PY1,1)>Th) && (PY1>1))
    PY1 = PY1 - 1;
end
PY2 = MaxY;
while ((Blue_y(PY2,1)>Th) && (PY2<y))
    PY2 = PY2 + 1;
end
PY1 = PY1 - 2;
PY2 = PY2 + 2;
if PY1 < 1
    PY1 = 1;
end
if PY2 > y
    PY2 = y;
end
IY = I(PY1:PY2, :, :);

% 列过滤
Blue_x = zeros(1,x);
for j = 1:x
    for i = PY1:PY2
        rij = myI(i, j, 1)/(myI(i, j, 3)+eps);
        gij = myI(i, j, 2)/(myI(i, j, 3)+eps);
        bij = myI(i, j, 3);
        if (rij < parm(4) && gij < parm(5) && bij > parm(6)) ...
                || (gij < parm(4) && rij < parm(5) && bij > parm(6))
            Blue_x(1,j) = Blue_x(1,j) + 1; 
            bw2(i, j) = 1;
        end
    end
end

PX1 = 1;
while (Blue_x(1,PX1)<Th) && (PX1<x)
    PX1 = PX1 + 1;
end
PX2 = x;
while (Blue_x(1,PX2)<Th) && (PX2>PX1)
    PX2 = PX2 - 1;
end
PX1 = PX1 - 2;
PX2 = PX2 + 2;
if PX1 < 1
    PX1 = 1;
end
if PX2 > x
    PX2 = x;
end

% 准备返回的数据，包括：分割结果、分割结果坐标
IX = I(:, PX1:PX2, :);
Plate = I(PY1:PY2, PX1:PX2, :);
Loc.row = [PY1 PY2];
Loc.col = [PX1 PX2];
bw = bw1 + bw2;
bw = logical(bw);
bw(1:PY1, :) = 0;
bw(PY2:end, :) = 0;
bw(:, 1:PX1) = 0;
bw(:, PX2:end) = 0;
% 如果flag==1， 展示一下结果
if flag
    figure;       
    subplot(2, 2, 3); imshow(IY); title('行过滤结果', 'FontWeight', 'Bold');
    subplot(2, 2, 1); imshow(IX); title('列过滤结果', 'FontWeight', 'Bold');
    subplot(2, 2, 2); imshow(I); title('原图像', 'FontWeight', 'Bold');
    subplot(2, 2, 4); imshow(Plate); title('分割结果', 'FontWeight', 'Bold');
end
```



##### 区域二值化模块

- 是将彩色图转为只有0黑和255白两种颜色的过程

- RGB图像转灰度图像》转二值图像》滤波》边界对象抑制

```matlab
% RGB图像转灰度图像
if ndims(plate) == 3
    plate1 = rgb2gray(plate);
else
    plate1 = plate;
end
Im = plate1;
% 把灰度图像转换成二值图像（只有纯黑（0）和纯白（255）两种颜色的图像）
bw = im2bw(Im);
% 对图像进行滤波操作
h = fspecial('average', 2);             % 均值滤波器
bw1 = imfilter(bw, h, 'replicate');
% 边界对象抑制：删除和图像边界相连的对象
mask = Mask_Process(bw1);
bw2 = bw1 .* mask;
result = bw2;
```



##### 字符分割模块

- 将图像的7各字符分割出来，对于算法来说是找到黑色中的白色区域
- 去除边缘》切割字符》归一化处理

```matlab
% 切割出第一个字符
bw = Segmation(bw);
[m, n] = size(bw);
wideTol = round(n/20);
rateTol = 0.25;
flag = 0;
word1 = [];
while flag == 0
    [m, n] = size(bw);
    left = 1;
    wide = 0;
    while sum(bw(:,wide+1)) ~= 0
        wide = wide+1;
    end
    if wide < wideTol
        bw(:, 1:wide) = 0; 
        bw = Segmation(bw);
    else
        temp = Segmation(imcrop(bw, [1 1 wide m]));
        [m, n] = size(temp);
        tall = sum(temp(:)); 
        two_thirds = sum(sum( temp(round(m/3):2*round(m/3), :) ));
        rate = two_thirds/tall; 
        if rate > rateTol
            flag = 1;
            word1 = temp;  
        end
        bw(:, 1:wide) = 0; 
        bw = Segmation(bw);
    end
end
% 切割其他字符
[word2, bw] = Word_Segmation(bw);
[word3, bw] = Word_Segmation(bw);
[word4, bw] = Word_Segmation(bw);
[word5, bw] = Word_Segmation(bw);
[word6, bw] = Word_Segmation(bw);
[word7, bw] = Word_Segmation(bw);
```



##### 车牌识别模块

- 导入模板》每个字符逐个匹配，取差值最小的模板图片为结果

```matlab
% 导入模板图片，并对模板图片预处理
pattern = [];
dirpath = fullfile(pwd, '标准库/*.bmp');
files = ls(dirpath);
for t = 1 : length(files)
    filenamet = fullfile(pwd, '标准库', files(t,:));
    [~, name, ~] = fileparts(filenamet);
    imagedata = imread(filenamet);
    imagedata = im2bw(imagedata, 0.5);
    pattern(t).feature = imagedata;
    pattern(t).name = name;
end

distance = [];
for m = 1 : 7;
    % 每个字符逐个对比识别
    for n = 1 : length(files);
        switch m
            case 1
                distance(n)=sum(sum(abs(words.word1-pattern(n).feature)));
            case 2
                distance(n)=sum(sum(abs(words.word2-pattern(n).feature)));
            case 3
                distance(n)=sum(sum(abs(words.word3-pattern(n).feature)));
            case 4
                distance(n)=sum(sum(abs(words.word4-pattern(n).feature)));
            case 5
                distance(n)=sum(sum(abs(words.word5-pattern(n).feature)));
            case 6
                distance(n)=sum(sum(abs(words.word6-pattern(n).feature)));
            case 7
                distance(n)=sum(sum(abs(words.word7-pattern(n).feature)));
        end
    end
    [yvalue,xnumber]=min(distance);         % [最小值，index]
    filename = files(xnumber, :);
    [pathstr, name, ext] = fileparts(filename);
    result(m) = name;
end
```

