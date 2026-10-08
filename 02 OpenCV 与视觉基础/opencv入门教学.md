# opencv入门教学

# 前言：

对于视觉组同学来说，其核心任务就是“让计算机看懂图像和视频，代替人的眼睛，从而更加智能”。对于入门计算机视觉（computer vision简称cv），我们通常会让初学者先去学opencv库（一个开源免费且十分强大的库）。下面的教程全程使用C\+\+，与python的原理是一样的。本人水平有限，难免会有纰漏，如果发现哪里不对，望大家指出。



我们不当百科全书，不把所有的函数及其用法都写上，比如图像的旋转，缩放这些函数，查一查就知道不需要写，我们聚焦的，是让读者能快速上手使用opencv做视觉识别算法

# To Images

首先我们需要理解图像是由什么构成的，假设我现在有一张网格

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%201.png)

我想要在网格里写上数字3

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2048.png)

只需要将其中几个网格涂成黑色即可

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2042.png)

为了方便我们让黑色的格子为0，白色的格子为1

如果想要增加更多细节，只需要增加格子数量即可，而这些格子也就是我们通常所说的像素，举个栗子，我们常说的4K=3840\*2160，实际上就是指图像的宽为3840，高为2160个格子或者像素组成（我们通常说宽和高，而不说长和宽）。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2035.png)

而现在我们只有黑白两种颜色，这样的图像我们称之为二值图，现在我们要增加更多细节，这意味着我们不能只有0和1两种取值

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2025.png)

于是我们将使用256种取值，其中0是黑色，255是白色

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2044.png)

于是我们得到了灰度图像

对于彩色图像我们有红色，绿色，蓝色（R,G,B\)三种灰度图像，分别代表其颜色的强度

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%204.png)

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2014.png)

# 配置环境

有了一定的前置知识，接下来正式开始opencv教学，首先我们需要配置环境，这里我们使用vscode，跟着下面这个教程操作即可

<https://blog.csdn.net/zczc66/article/details/145964746?fromshare=blogdetail&sharetype=blogdetail&sharerId=145964746&sharerefer=PC&sharesource=&sharefrom=from_link>



#### For Mac

直接用homebrew ，`brew install opencv`就行，homebrew 会自动配置变量，然后mac 也不用额外配置cmake,直接新建一个项目，加一个CMakeLists\.txt,内容大概是

![截屏2025\-10\-13 14\.35\.05\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2014.35.05.png)

#### For windows

##### 在vscode里面配置c\+\+

首先，需要在你的windows上安装mingw64（这个网上教程很多，这里就不多说了。它包含了c和c\+\+的编译器）,并将其在系统环境变量里面进行配置。（安装好后可以输入g\+\+ \-\-version查看是否已经下载好）

下载好后，在vscode里面把c,c\+\+,cmake的相关插件下载好。

之后就可以新建一个\.cpp为后缀的文件了。


![fc2c13512444b63341f09ff1cdadc866\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/fc2c13512444b63341f09ff1cdadc866.png)

点击运行后，会出现两个选项。（如果你是想以c语言写，那么就选择第一个，它会生成gcc\.exe的文件，如果是用C\+\+写的，就选择第二个）

点击后，如果在文件夹里面有\.vscode文件，那么就算成功。（这个文件夹用来存放特定的配置文件，大都是\.json文件，不用管）

之后，再点击运行C\+\+程序，如果在文件夹里面出现了\.exe文件，那基本上就算成功了。

##### 安装cpp环境下的opencv

由于传统cpp不像python一样，python拥有pip install的包管理器，传统的cpp的依赖管理要比python繁琐很多。主播一开始就是在官网下载opencv包，手动配置环境变量的方法，但最后还是没能成功编译。

但经过查阅资料发现，现代cpp项目也可以借助包管理器来实现类似python的便捷性。

目前cpp的包管理器有两款：
vcpkg,Conan,这里主播有的是vcpkg,它是由微软开发，开箱即用，与cmake集成简单，适合个人项目和原型开发。

这里主播讲解一下如何用vcpkg来配置opencv。

打开cmd或者powershell,使用 Git 将 vcpkg 仓库克隆到本地（例如 `D:\vcpkg`）注：这里需要稳定的网络环境，且有可能一次并不能完全下载成功，可能要多试几次。

```Python
git clone https://github.com/microsoft/vcpkg.git D:\vcpkg
cd D:\vcpkg
```

进入 vcpkg 目录并运行引导脚本，这会编译出 `vcpkg.exe`可执行文件：

```Python
.\bootstrap-vcpkg.bat 
```

使用 vcpkg 安装 OpenCV 的 64 位版本。这个过程会自动下载 OpenCV 源代码及其所有依赖项（如 libjpeg, libpng 等），并进行编译。这需要一些时间

```Python
.\vcpkg install opencv:x64-windows
```

以上是安装vcpkg和opencv,它们都是全局的，只用在新创建的opencv项目的cmakelist下配置好路径就好（具体怎么配后面文档会详细提到）。

好了，接下来让我们新建一个opencv项目。


```Python
#创建新项目目录
mkdir my_new_opencv_project
cd my_new_opencv_project
```

以下提供CMakelist\.txt,这个模板可以重复使用。

```Markdown
cmake_minimum_required(VERSION 3.10)

# 设置 vcpkg 工具链（关键！）
set(CMAKE_TOOLCHAIN_FILE "D:/vcpkg/scripts/buildsystems/vcpkg.cmake")

project(MyNewOpenCVProject)

# 查找 OpenCV 包
find_package(OpenCV REQUIRED)

message(STATUS "OpenCV version: ${OpenCV_VERSION}")

# 添加可执行文件（修改这里的文件名）
add_executable(face_detection face_detection.cpp)

# 链接 OpenCV 库
target_link_libraries(face_detection ${OpenCV_LIBS})

# 包含头文件目录
target_include_directories(face_detection PRIVATE ${OpenCV_INCLUDE_DIRS})

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
```

如果后面你想修改模板，只用修改add\_executable\(face\_detection face\_detection\.cpp\)；和你的project名就可以了。

接下来我们来写一个示例文件，名字就叫face\_detection\.cpp吧：


```C++
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    // 创建一个简单的图像处理示例
    cv::Mat image = cv::Mat::zeros(400, 600, CV_8UC3);
    
    // 绘制一个笑脸
    cv::circle(image, cv::Point(300, 200), 100, cv::Scalar(0, 255, 255), -1); // 脸
    cv::circle(image, cv::Point(270, 170), 10, cv::Scalar(0, 0, 0), -1);     // 左眼
    cv::circle(image, cv::Point(330, 170), 10, cv::Scalar(0, 0, 0), -1);     // 右眼
    cv::ellipse(image, cv::Point(300, 230), cv::Size(40, 20), 0, 0, 180, cv::Scalar(0, 0, 0), 3); // 嘴
    
    cv::putText(image, "New OpenCV Project!", 
                cv::Point(150, 350), 
                cv::FONT_HERSHEY_COMPLEX, 1.0, 
                cv::Scalar(255, 255, 255), 2);
    
    cv::imshow("Face Detection", image);
    std::cout << "新项目运行成功！" << std::endl;
    cv::waitKey(0);
    return 0;
}
```

确保运行成功，最好在运行CMake之前设置环境变量：


```Shell
# 设置环境变量
$env:OpenCV_DIR = "D:/vcpkg/installed/x64-windows/share/opencv4"
$env:CMAKE_PREFIX_PATH = "D:/vcpkg/installed/x64-windows"

```

OK,万事俱备，只欠运行：


```Python
# 1. 配置项目（在项目目录中运行）
cmake -B build -DCMAKE_TOOLCHAIN_FILE=D:/vcpkg/scripts/buildsystems/vcpkg.cmake

# 2. 编译项目
cmake --build build

# 3. 运行程序
.\build\Debug\face_detection.exe
```

注：每次如果重新修改了代码，需要重新编译项目，再运行程序。

最后运行出来是这样的：


![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2037.png)

注：
这里主播有报错，但是程序可以运行，uu们不用担心，这个错误是 IDE/编辑器（如 VS Code）的智能提示问题，不是编译错误。你的程序能正常运行说明 OpenCV 已经正确安装和链接了。（但也记得调 vscode 的 json 配置文件，把opencv的路径include进去就行，不然会一直弹报错影响编码）

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2032.png)

参考配置（这个放进\.vscode文件夹里面）：


```Bash
{
    "configurations": [
        {
            "name": "Win32",
            "includePath": [
                "${workspaceFolder}/**",
                "D:/vcpkg/installed/x64-windows/include",
                "D:/vcpkg/installed/x64-windows/include/opencv4"
            ],
            "defines": [
                "_DEBUG",
                "UNICODE",
                "_UNICODE"
            ],
            "windowsSdkVersion": "10.0.22621.0",
            "compilerPath": "C:/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.41.34120/bin/Hostx64/x64/cl.exe",
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "windows-msvc-x64"
        }
    ],
    "version": 4
}
```

如果修改了文件夹路径需要重新编译，则需要清理并重新配置：



```Markdown
# 删除旧的 build 目录和缓存
rm -r build

# 重新创建配置
cmake -B build -DCMAKE_TOOLCHAIN_FILE=D:/vcpkg/scripts/buildsystems/vcpkg.cmake

# 编译所有程序
cmake --build build
```



如果一个文件夹下有多个cpp文件，那么也要修改CMakelist的配置：


```Markdown
# 项目1：人脸检测演示
add_executable(face_detection face_detection.cpp)
target_link_libraries(face_detection ${OpenCV_LIBS})

# 项目2：Sobel边缘检测演示
add_executable(sobel_demo sobel_demo.cpp)
target_link_libraries(sobel_demo ${OpenCV_LIBS})

# 包含头文件目录（对所有目标生效）
target_include_directories(face_detection PRIVATE ${OpenCV_INCLUDE_DIRS})
target_include_directories(sobel_demo PRIVATE ${OpenCV_INCLUDE_DIRS})

```

不同的cpp文件需要不同的add\_executable和target\_link\_libraries



# 3\.第一个Opencv程序\(using cmake\)



opencv 是模块化组织的，第一步我们一般先用`core`和`highgui`这两个模块，opencv 的"hello world"长这样



![截屏2025\-10\-13 15\.07\.22\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2015.07.22.png)

运行结果是

![截屏2025\-10\-13 14\.47\.18\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2014.47.18.png)

注意我这里先自己准备了"example\.png",然后为了各位想直接copy代码的，就没用绝对路径，而是`cv::Mat image = cv::imread("example.png");`,这个相对路径，如果想直接运行，你需要将你准备的example\.png放在编译完成后的可执行程序所在的目录下，如我这里就是`cmake-build-debug`这个文件夹

以上代码中，我们使用 `imread` 函数读取图像，使用 `imshow` 函数显示图像，并使用 `waitKey` 函数等待用户按键

`imread`这是文件I/O 的函数，属于`core`模块，`imshow`和`waitKey`属于`highgui`模块

`cv::Mat`是opencv 的基础数据结构，基本上就是由opencv 定义的一个类，用于储存和操作多维数组，这里来存图形的数组,这个数据结构具体包含什么，可以自己去了解

> 对于不知道图形是以数组储存的读者来说，这里做个介绍，计算机中图形都是以数组的形式储存的，一般有RGB 三个通道，有些还有一个A 的通道，说是数组，其实是矩阵
> 
> 

每个函数具体的，大家在自己的IDE 中鼠标悬停在想了解的函数上就会自动浮现介绍\(其实就是注释\),如

![截屏2025\-10\-13 15\.06\.07\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2015.06.07.png)

# 4\.图片处理

### cv::Mat 与 图像 IO

在OpenCV中，所有图像、视频帧都存储在一个核心数据结构中：`cv::Mat`,它可以存储图像的像素值，图像的宽度、高度、通道数、数据类型等元信息

直接用一个练习程序来讲



![截屏2025\-10\-13 15\.52\.44\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2015.52.44.png)

运行结果

![截屏2025\-10\-13 15\.33\.13\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2015.33.13.png)

这个程序就是将彩色图像转换成灰色图像，我们来解析一下代码

我建了一个`cv::Mat`的对象`graycolor`,用`cv::cvtColor(image, grayimage, cv::COLOR_BGR2GRAY);`这个函数来转换色彩空间

![截屏2025\-10\-13 15\.53\.40\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/截屏2025-10-13%2015.53.40.png)

这是它的函数签名，这个函数属于Imgproc 模块，这个模块提供了大量的图像处理函数。这些函数可以用于图像的滤波、几何变换、颜色空间转换、直方图计算等

- 图像滤波：包括均值滤波、高斯滤波、中值滤波等，用于去除图像中的噪声

- 几何变换：如图像的缩放、旋转、仿射变换等

- 边缘检测：如 Canny 边缘检测、Sobel 算子等

- 直方图均衡化：用于增强图像的对比度

这个函数的核心功能就是将图像从一个色彩空间转换到另一个色彩空间，`cv::COLOR_BGR2GRAY` 是一个颜色空间转换代码常量，它指定了转换的类型

- `COLOR_`: 指示这是一个颜色转换常量

- `BGR`: 源色彩空间（OpenCV默认的蓝色\-绿色\-红色通道顺序）

- `2`: “转换到”（to）

- `GRAY`: 目标色彩空间（灰度）

### 图像预处理

图像预处理是我们做检测算法前对图像的提纯操作，因为一张图片会有大量无关信息和噪声，我们要做的就是减少干扰、提取我们感兴趣的信息

##### 滤波

来简单介绍一下滤波种类

##### **均值滤波**

均值滤波是一种简单的线性滤波方法，它将图像中每个像素的值替换为其邻域内所有像素值的平均值。这种方法可以有效去除噪声，但也会使图像变得模糊

```Plain Text
cv::Mat src = cv::imread("image.jpg", cv::IMREAD_GRAYSCALE);
cv::Mat dst;
cv::blur(src, dst, cv::Size(3, 3));  // 3x3的均值滤波
```

##### **高斯滤波**

高斯滤波是一种非线性滤波方法，它使用高斯函数来计算邻域内像素的权重，从而对图像进行平滑处理。高斯滤波在去除噪声的同时，能够更好地保留图像的边缘信息

```Plain Text
cv::GaussianBlur(src, dst, cv::Size(5, 5), 0);  // 5x5的高斯滤波
```

##### **中值滤波**

中值滤波是一种非线性滤波方法，它将图像中每个像素的值替换为其邻域内所有像素值的中值。这种方法在去除椒盐噪声时效果非常好

```Plain Text
cv::medianBlur(src, dst, 5);  // 5x5的中值滤波
```

##### **双边滤波**

双边滤波也是一种非线性滤波器，是一种边缘保留的平滑滤波算法，它既能去除噪声，也能保证图像的边缘得到保留。

```Plain Text
bilateralFilter(img, result, 15, 80, 80);
```

##### **自定义滤波器**

OpenCV允许用户自定义滤波器核，通过`cv::filter2D`函数可以实现自定义滤波操作

```Plain Text
cv::Mat kernel = (cv::Mat_<float>(3, 3) << 1, 0, -1, 0, 0, 0, -1, 0, 1);
cv::filter2D(src, dst, -1, kernel);
```

##### 形态学操作

###### 什么是形态学？

形态学主要是从图像内提取分量信息，该分量通常是图像理解所使用的的最本质的形状特征，形态学在视觉检测、文字识别、医学图像处理等有重要的应用。

###### 腐蚀

它是最基本的形态学操作之一，具体作用就是：能够消除图像的边界点，使图像沿着边界向内收缩，也可以去除小于指定结构元的部分。

以下是图解：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%205.png)

那么，可以看出，设置合理的结构元就十分重要了，如何设置合理的大小，充分利用资源，是关键。

```C++
#include <opencv2/opencv.hpp>
#include <iostream>

int main() {
    // 1. 读取图像并转为灰度图
    cv::Mat src = cv::imread("C:\\Users\\lenovo\\Desktop\\5935.jpg_wh860.jpg", cv::IMREAD_GRAYSCALE);
    if (src.empty()) {
        std::cerr << "无法读取图像!" << std::endl;
        return -1;
    }
    
    // 2. 二值化处理（为演示腐蚀效果）
    cv::Mat binary;
    cv::threshold(src, binary, 128, 255, cv::THRESH_BINARY);
    
    // 3. 创建不同的结构元素
    cv::Mat kernel_rect = cv::getStructuringElement(cv::MORPH_RECT, cv::Size(5, 5));
    cv::Mat kernel_cross = cv::getStructuringElement(cv::MORPH_CROSS, cv::Size(5, 5));
    cv::Mat kernel_ellipse = cv::getStructuringElement(cv::MORPH_ELLIPSE, cv::Size(5, 5));
    
    // 4. 应用腐蚀操作
    cv::Mat eroded_rect, eroded_cross, eroded_ellipse;
    
    // 基础腐蚀（矩形核，迭代1次）
    cv::erode(binary, eroded_rect, kernel_rect);
    
    // 十字形核腐蚀
    cv::erode(binary, eroded_cross, kernel_cross);
    
    // 椭圆形核腐蚀
    cv::erode(binary, eroded_ellipse, kernel_ellipse);
    
    // 5. 多次迭代腐蚀效果
    cv::Mat eroded_iter3;
    cv::erode(binary, eroded_iter3, kernel_rect, cv::Point(-1,-1), 3);
    
    // 6. 显示结果
    cv::imshow("1. 原图", src);
    cv::imshow("2. 二值图像", binary);
    cv::imshow("3. 矩形核腐蚀", eroded_rect);
    cv::imshow("4. 十字核腐蚀", eroded_cross);
    cv::imshow("5. 椭圆核腐蚀", eroded_ellipse);
    cv::imshow("6. 3次迭代腐蚀", eroded_iter3);
    
    std::cout << "观察不同结构元素和迭代次数对腐蚀效果的影响：" << std::endl;
    std::cout << "- 矩形核：各向同性腐蚀" << std::endl;
    std::cout << "- 十字核：主要腐蚀水平和垂直方向" << std::endl;
    std::cout << "- 椭圆核：近似圆形腐蚀" << std::endl;
    std::cout << "- 多次迭代：腐蚀效果更明显" << std::endl;
    
    cv::waitKey(0);
    return 0;
}
```

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%203.png)

运行后可发现，不同的腐蚀效果是不同的，这就启示我们，需要设置合理的结构元，使特征体现得到最大化。

###### 膨胀

膨胀相当于是腐蚀反向操作，图像中较亮的物体尺寸会变大，较暗的物体尺寸会减小。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2022.png)

# 5\.图像特征提取

### 边缘检测专题

#### sobel算子原理与实现

##### 什么是sobel算子？

用于边缘检测的离散微分算子。

具体数学原理部分可以参考csdn：<https://blog.csdn.net/great_yzl/article/details/119709699>

这里主播只给大家讲解应该如何去使用。

##### 具体算法的代码实现：

1、边缘检测: Gx 用于检测纵向边缘, Gy 用于检测横向边缘。



示例图片原图：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2010.png)

简单来说：

**对x取微分，得到的是y方向的边缘；**

```C++
//对x方向微分
    Sobel(gray, grad_x, CV_16S, 1,0,3);   
    // x方向差分阶数  y方向差分阶数 核大小
    convertScaleAbs(grad_x, abs_grad_x);//可将任意类型的数据转化为CV_8UC1
    imshow("【边缘图x】", abs_grad_x); 
    //边缘与梯度方向垂直，所以输出的边缘是和我们所计算的某一方向的梯度是垂直的
    
```

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2049.png)

**对y取微分，得到的是x方向的边缘。**

```C++
//对y方向微分
    Sobel(gray, grad_y, CV_16S, 0, 1, 3);
    // x方向差分阶数 y方向差分阶数 核大小
    convertScaleAbs(grad_y, abs_grad_y);//可将任意类型的数据转化为CV_8UC1
    imshow("边缘图y", abs_grad_y);
```

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2036.png)

**线性混合：**

```C++
//图像的线性混合
    addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 0, dst);
    imshow("线性混合", dst);
```

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2045.png)

可以很明显看出，通过对x方向求微分和y方向求微分后，都没有将它们线性融合后得到的边缘检测效果好。

总代码：

```C++
//Sobel算子（微分）
#include<opencv2/opencv.hpp>
#include<opencv2/imgproc/imgproc.hpp>
using namespace cv;
 
int main()
{
    Mat src, dst, gray, grad_x, grad_y, abs_grad_x, abs_grad_y;
    src = imread("Resource/test12.jpg");
    imshow("原图", src);
    cvtColor(src, gray, COLOR_RGB2GRAY);   //转变为灰度图
    imshow("灰度图", gray);
 
    //对x方向微分
    Sobel(gray, grad_x, CV_16S, 1,  0,3);   
    //  x方向差分阶数   y方向差分阶数   核大小
    convertScaleAbs(grad_x, abs_grad_x);     //可将任意类型的数据转化为CV_8UC1
    imshow("边缘图x", abs_grad_x); 
    //边缘与梯度方向垂直，所以输出的边缘是和我们所计算的某一方向的梯度是垂直的
 
    //对y方向微分
    Sobel(gray, grad_y, CV_16S, 0, 1, 3);
    //  x方向差分阶数   y方向差分阶数   核大小
    convertScaleAbs(grad_y, abs_grad_y);//可将任意类型的数据转化为CV_8UC1
    imshow("边缘图y", abs_grad_y);
 
    //图像的线性混合
    addWeighted(abs_grad_x, 0.5, abs_grad_y, 0.5, 0, dst);
    imshow("线性混合", dst);
    waitKey(0);
}
```

计算法线: Gx 用于计算法线的横向偏移, Gy 用于计算法线的纵向偏移\. Sobel边缘检测算法比较简单，实际应用中效率比canny边缘检测效率要高，但是边缘不如Canny检测的准确，但是很多实际应用的场合，sobel边缘却是首选，Sobel算子是高斯平滑与微分操作的结合体，所以其抗噪声能力很强，用途较多。尤其是效率要求较高，而对细纹理不太关心的时候。

#### Canny边缘检测算法

##### 什么是canny边缘检测？

canny边缘检测是一种多级边缘检测算法，核心目标是实现高检测率，高定位精度和低边缘误报。

以下是它的几个步骤：
1\.**高斯滤波**：减少噪点对边缘检测的干扰。
2\.**计算梯度**：识别边缘，使用Sobel等算子计算图像在水平和垂直方向的梯度，得到梯度的幅值和方向。梯度方向通常被近似到0°、45°、90°、135°等有限方向。
3\.**非极大值抑制**：使粗边缘变为单像素宽度的细边缘。

4\.**双阈值检测**：设置一个高阈值和一个低阈值，梯度幅值高于高阈值的点为强边缘，低于低阈值的点被抑制，介于两者之间的为弱边缘。

5\.**滞后阈值**：仅当它们与强边缘点相连才被保留为有效边缘。

##### canny函数

```C++
void Canny(InputArray image, OutputArray edges, double threshold1, double threshold2, int apertureSize=3, bool L2gradient=false)
```

第一个参数：InputArray类型的image，输入图像，Mat对象节课，需为单通道8位图像。

第二个参数：OutputArray类型的edges，输出的边缘图，需要和输入图像有相同的尺寸和类型。

第三个参数：double类型的threshold1（低阈值）

第四个参数：double类型的threshold2（高阈值）

（推荐高低阈值比在2:1和3:1之间）

第五个参数：int类型的apertureSize，表示算子的孔径的大小，默认值时3，**孔径越大，对噪声抑制越强，但边缘定位可能稍模糊**。

第六个参数：bool类型的L2gradient，一个计算图像梯度复制的标识，默认false。（默认`false`使用L1范数（计算快）；设为`true`使用更精确的L2范数（计算稍慢））

##### 代码实现

```C++
#include <iostream>
#include <opencv2/opencv.hpp>
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>
#include <opencv2/imgproc/imgproc.hpp>
 
using namespace std;
using namespace cv;
int main() {
    Mat srcImage, grayImage;
    srcImage = imread("C:\\Users\\lenovo\\Desktop\\GSing--opencv\\my_new_opencv_project\\20251015200422_102_6.png");//读取图像
    Mat srcImage1 = srcImage.clone();//克隆图像
    cvtColor(srcImage, grayImage, COLOR_BGR2GRAY);//灰度化
    Mat dstImage, edge;
 
    blur(grayImage, grayImage, Size(3,3));//平滑滤波
    Canny(grayImage, edge, 150, 100, 3);//canny边缘检测，高阈值150，低阈值100，sobel孔径：3
 
    dstImage.create(srcImage1.size(), srcImage1.type());
    dstImage = Scalar::all(0);//创建输出掩膜
    srcImage1.copyTo(dstImage, edge);//叠加边缘
    imwrite("canny.jpg", dstImage);
 
    return 0;
}
```

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%207.png)

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2033.png)

这里只是讲解了各种算法的基本使用，如果想通过canny进行一些实际处理（比如对赛道的识别），那就需要设置相应的ROI，然后对赛道的特征进行一个限定。（具体的代码怎么写可以询问ai）

# 6\.颜色空间与分割

1\.颜色空间需要掌握3种表示方法：

RGB:在前面我们已经介绍过，在此不再赘述，简单说明一下RGB的弊端：对亮度变化不是很敏感，颜色分割不如以下介绍的另一种方式。（注意：在opencv中默认读取顺序为BGR）

HSV:由色调（Hue），饱和度（Saturation），明度（Value）组成

色调：表示颜色种类,取值范围通常是0\-360°，在opencv中为0\-179，例如蓝色在240°左右

饱和度：表示颜色的纯度，取值范围是0 \- 100%（在OpenCV等计算机视觉库中为0 \- 255）。饱和度越高，颜色越鲜艳；当饱和度为0时，颜色变为灰度，即没有颜色成分，只有亮度信息。

明度：描述颜色的明亮程度，取值范围也是0 \- 100%（在计算机视觉库中为0 \- 255）。明度为0时表示黑色，明度越高，颜色越亮。

Lab：由亮度Light和有关色彩的a\&b共同构成。a表示从绿色到红色（数值对应从负到正，后面同理），b表示从蓝色到黄色的范围。其中L的值域为\[0, 255\], a的值域为\[\-128, 127\], b的值域为\[\-128, 127\]\.其色域是最广的，囊括了RGB\(屏幕输出\)和CMYK\(设备打印\)\.

## 常用颜色空间有哪些？


### RGB颜色空间


定义：是一种通过红，绿，蓝三种基本色光的不用比例叠加产生各种颜色模拟系统。

基本原理：加色混合。

核心构成：三种颜色通道，红，绿，蓝，每个通道的亮度由0\-255的整数表示。

主要的缺点：三个颜色分量高度相关，改变亮度会使所有分量一同变化，不利于图像处理中单独调整色彩属性；均匀性差，即空间中两点间的几何距离不能准确反映人眼感知的色彩差异。**因此，RGB适合显示系统，不适合图像处理。**

### HSI/HSV/HSL颜色空间

### HSI：


HSI是一种非常符合人类视觉直觉的色彩描述方式：H \(Hue, 色调\)、S \(Saturation, 饱和度\)、I \(Intensity, 亮度\)

**色调：**表示颜色的种类（如红、黄、绿），用角度表示（0\-360°）。例如，红色为0°，绿色为120°，蓝色为240° 
**饱和度：**表示颜色的纯度或鲜艳程度。从0%（灰色）到100%（完全饱和的纯色）
**亮度：**表示颜色的明暗程度。从0%（黑）到100%（白）

### HSV：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%208.png)

HSV 表达彩色图像的方式由三个部分组成：

Hue（色调、色相）

Saturation（饱和度、色彩纯净度）

Value（明度）

用下面这个圆柱体来表示 HSV 颜色空间，圆柱体的横截面可以看做是一个极坐标系 ，H 用极坐标的极角表示，S 用极坐标的极轴长度表示，V 用圆柱中轴的高度表示。



### HSL：


HSL和 HSV 比较类似，这里一起介绍。HLS 也有三个分量，hue（色相）、saturation（饱和度）、lightness（亮度）。

HLS 和 HSV 的区别就是最后一个分量不同，HLS 的是 light\(亮度\)，HSV 的是 value\(明度\)。

HLS 中的 L 分量为亮度，亮度为100，表示白色，亮度为0，表示黑色；HSV 中的 V 分量为明度，明度为100，表示光谱色，明度为0，表示黑色。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2021.png)

### 其他颜色空间\(科普，仅作了解\)

### CMYK：

核心组成：Cyan（青）、Magenta（品红）、Yellow（黄）、Key（黑），CMYK色彩模式是一种依靠反光的色彩模式，通过色料的三原色混色原理，加上黑色油墨，共计四种颜色混合叠加，形成所谓的“全彩印刷”。

### Lab：

L\*代表明度，取值0\~100

a\*代表从绿色到红色的分量 ，取值\-128\~127

b\*代表从蓝色到黄色的分量，取值\-128\~127

这样规定是根据人类的视觉原理，灵长类动物的视觉都有两条通道：红绿通道和蓝黄通道，大多数动物最多只有一条通道，如果有人缺失其中一条，就是我们所说的色盲。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2043.png)

# 机器学习在opencv中的应用

## 什么是机器学习？

### 定义

机器学习是人工智能的一个核心分支，其目标是研究如何通过**计算手段**，利用经验（通常以数据形式存在）来改善系统自身的性能。机器学习算法是一类能**从数据中自动分析获得规律（模型）**，并利用此规律对未知数据进行预测的算法。

机器学习是一个迭代过程，可能需要多次调整模型参数和特征选择，以提高模型的性能。

下面这张图展示了机器学习的基本流程：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2017.png)

**Labeled Data（标记数据）：**图中蓝色区域显示了标记数据，这些数据包括了不同的几何形状（如六边形、正方形、三角形）。

**Model Training（模型训练）：**在这个阶段，机器学习算法分析数据的特征，并学习如何根据这些特征来预测标签。

**Test Data（测试数据）：**图中深绿色区域显示了测试数据，包括一个正方形和一个三角形。

**Prediction（预测）：**模型使用从训练数据中学到的规则来预测测试数据的标签。在图中，模型预测了测试数据中的正方形和三角形。

**Evaluation（评估）：**预测结果与测试数据的真实标签进行比较，以评估模型的准确性。

## 机器学习的分类

### **监督学习（Supervised Learning）**

- **定义：** 监督学习是指使用带标签的数据进行训练，模型通过学习输入数据与标签之间的关系，来做出预测或分类。

- **应用：** 分类（如垃圾邮件识别）、回归（如房价预测）。

- **例子：** 线性回归、决策树、支持向量机（SVM）。

### **无监督学习（Unsupervised Learning）**

- **定义：** 无监督学习使用没有标签的数据，模型试图在数据中发现潜在的结构或模式。

- **应用：** 聚类（如客户分群）、降维（如数据可视化）。

- **例子：** K\-means 聚类、主成分分析（PCA）。

### **强化学习（Reinforcement Learning）**

- **定义：** 强化学习通过与环境互动，智能体在试错中学习最佳策略，以最大化长期回报。每次行动后，系统会收到奖励或惩罚，来指导行为的改进。

- **应用：** 游戏AI（如AlphaGo）、自动驾驶、机器人控制。

- **例子：** Q\-learning、深度Q网络（DQN）。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2030.png)

### 半监督学习**（ half Supervised Learning）**

由前文得知，有监督学习中所有用来训练的数据都是有标签的，无监督学习中所有用来训练数据都是没有标签的。有时，有条件给部分训练数据有标签的学习方式为半监督学习。

## 机器学习的泛化能力

指机器通过对测试数据的学习，掌握解决新数据问题的能力。

泛化的基础是重复的，大量的，科学的训练和练习。

## opencv的ml模块简介

opencv的ml\(machine learning\)代码库，包含各种机器学习的算法。下面给大家简单列举一些基础的算法以及它们的基本作用，里面有的算法在后续的文档中会讲到：

**1，Normal Bayes Classifier（**[**贝叶斯**](https://so.csdn.net/so/search?q=%E8%B4%9D%E5%8F%B6%E6%96%AF&spm=1001.2101.3001.7020)**分类）**

基于贝叶斯定理，假设特征服从正态分布，可以进行一些文本分类等。

**2，K\-Nearest Neighbour Classifier（K\-邻近算法）**

一个样本的类别由其k个最近邻居的**多数投票决定**。可以进行一些任务分类等。

**3，SVM,support vector machine\(支持向量机\)**

寻找一个超平面，最大化不同类别样本之间的边界（间隔），可以进行图像分类等。

**4，Decision Tree（决策树）**

通过一系列if\-then规则对数据进行划分，形似树结构。需要进行特征重要分析的任务。

**5，Random Trees Classifier（随机森林算法）**

通过Bagging​（自助采样）集成多棵决策树，综合投票结果，用于分类问题。

**6， Boosted tree classifier （Boost树算法）**

通过Boosting​（顺序学习，纠正前一个模型的错误）集成多个弱分类器（通常是树）

**7，ANN,Artificial Neural Networks\(人工神经网络\)**

模拟人脑神经元网络，通过多层连接的结构学习复杂的非线性关系。用于复杂的非线性分类和回归问题。

有点不知道是什么？没关系，这里只是给大家列举一些比较重要的机器学习的算法。具体这些算法使用，我们会在后面的文档中给大家详细的讲解的。

## KNN（K近邻）

### 算法原理

KNN算法是机器学习算法中最基础、最简单的算法之一。它既能用于分类，也能用于回归。KNN通**过测量不同特征值之间的距离来进行分类**。（这里的距离测量一般是**欧式距离**，但也不排除有曼哈顿距离等）。KNN是一种**监督学习**算法，可用于分类和回归任务。在预测时，**算法计算待测样本与训练集中每个样本的距离**，选取距离最近的k个样本，然后根据这k个样本的类别，通过**投票或平均**来预测待测样本的类别或值。

### 核心思想

**根据它距离最近的 K 个样本点是什么类别来判断该新样本属于哪个类别**（多数投票）。
eg:

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2029.png)

图中绿色的点就是我们要预测的那个点，假设K=3。那么KNN算法就会找到与它距离最近的三个点（这里用圆圈把它圈起来了），看看哪种类别多一些，比如这个例子中是蓝色三角形多一些，新来的绿色点就归类到蓝三角了。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%206.png)

但是，**当K=5的时候，判定就变成不一样了**。这次变成红圆多一些，所以新来的绿点被归类成红圆。从这个例子中，我们就能看得出K的取值是很重要的。

那么该如何确定K取多少值好呢？从选取一个较小的K值开始，不断增加K的值，然后计算验证集合的方差，最终找到一个比较合适的K值。

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2034.png)

当你增大k的时候，一般错误率会先降低，因为有周围更多的样本可以借鉴了，分类效果会变好。但是，当K值更大的时候，错误率会更高。这也很好理解，比如说你一共就35个样本，当你K增大到30的时候，KNN基本上就没意义了。（**K的值通常取奇数**，以避免平票的现象）

KNN中的距离测量一般是欧式距离：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2023.png)

## SVM（支持向量机）

支持向量机（SVM）是一类按监督学习方式对数据进行二元分类的广义线性分类器，其决策边界是对学习样本求解的最大边距超平面，可以将问题化为一个求解凸二次规划的问题。

具体来说：在线性可分时，在原空间寻找两类样本的最优分类超平面。在线性不可分时，加入松弛变量并通过使用非线性映射将低维度输入空间的样本映射到高维度空间使其变为线性可分，这样就可以在该特征空间中寻找最优分类超平面。

SVM也有一定的适用范围，或者说准则：

1\.对于较少的样本量，我们选用逻辑回归模型或者不带核函数的支持向量机。

2\.对于较少的特征，建议使用高斯核函数的支持向量机，或者创造、增加更多的特征，然后使用逻辑回归或不带核函数的支持向量机。

举个例子：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2019.png)

以上三种分类方式哪一种最好，显然b线最好，因为b线到两类的最近样本点距离最大，换言之，b线能尽可能地分割这两类样本点，这样再有蓝色点落在平面上时，b线能够最大概率的让这点保持在它的下方，而支持向量就是距离决策边界最近的点，而我们的目的就是根据这些支持向量来找到最大间隔，这就是SVM\.

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image.png)

现在来具体研究这个问题，我们要求的是y2，不妨把这3条线按如下表示，

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%202.png)

高中的解析几何内容相信大家掌握的很好，在此不再赘述，我们希望求得决策边界上的点到y2的最大距离，于是乎我们可以得到n维的平行线距离公式：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2047.png)

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2018.png)

对于y1y2y3两边同时除以d，得到例如y1:y/d=wx/d\+b\-1,带入距离公式，得到

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2040.png)

我们现在让所有在y1下的点标记为\-1，y3上的点标记为1，为了让d最大，尽可能让w的模最小，可以转化成求如下问题，我们称其为目标函数

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2031.png)

下面介绍拉格朗日乘法和kkt条件

假设*x=x1,x2,\.\.\.,xn*是一个n维向量，*f\(x\)*和*h\(x\)*含有x的函数，我们需要找到满足*h\(x\)=0*条件下*f\(x\)*最小值，也就是将向量X带入h\(x\)中结果为0，带入f\(x\)中结果要最小，我们可以引入一个可以任意取值的自变量将两个函数h\(x\)和f\(x\)联系起来：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2013.png)

*L\(x，λ\)*叫做Lagrange函数（拉格朗日函数），*λ*叫做拉格朗日乘子（其实就是系数）。令*L\(x，λ\)*对x的导数为0，对*λ*的导数为0，求解出*x，λ* 的值，那么x就是函数*f\(x\)*在附加条件*h\(x\)*下极值点。

其实kkt条件和拉格朗日差不多，差别就在拉格朗日的条件是h\(x\)=0，而kkt是h\(x\)\<=0，而kkt是解决不等式问题，拉格朗日是解决等式问题，也就是假设x=x1,x2,\.\.\.,xn是一个n维向量，f\(x\)和h\(x\)含有x的函数，我们需要找到满足h\(x\)≤0条件下f\(x\)最小值，针对上式，显然是一个不等式约束最优化问题，不能再使用拉格朗日乘数法，因为拉格朗日乘数法是针对等式约束最优化问题。

那我们可以考虑加入一个“松弛变量”a\*\*2让条件h\(x\)≤0来达到等式的效果，即使条件变成h\(x\)\+a2=0，这里加上a2的原因是保证加的一定是非负数，即：a2≥0，但是目前不知道这个a2值是多少，一定会找到一个合适的a2值使h\(x\)\+a2=0成立。我们就把他变成等式约束了，就可以使用拉格朗日来操作了，只是多了a这个参数

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2015.png)

然后分别对三个参数求导

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%209.png)

根据6式，我们可以推导出如果λ=0，约束不起作用，根据①式可知*h\(x\)≤0；如果λ≠0，*由于a=0，约束条件起作用，根据⑤式可知，h\(x\)=0。综上两个步骤我们可以得到λ⋅h\(x\)=0，且在约束条件起作用时，λ\>0,h\(x\)=0；约束条件不起作用时，λ=0,h\(x\)≤0。上面方程组中的⑥式可以改写成λ⋅h\(x\)=0。由于a\*\*2≥0，所以将⑤式也可以改写成h\(x\)≤0，所以上面方程组也可以转换成如下：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2012.png)

以上便是不等式约束优化问题的KKT条件。

我们回到最开始要处理的问题上，根据③式可知，我们需要找到合适的x，λ，a值使L\(x，λ，a\)最小，但是合适的x，λ，a必须满足KKT条件。

我们对③式的优化问题可以进行优化如下

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2026.png)

满足最小化⑧式对应的x，λ值一定也要满足KKT条件，假设现在我们找到了合适的参数x值使f\(x\)取得最小值P【注意：这里根据①式来说，计算f\(x\)的最小值，这里假设合适参数x值对应的f\(x\)的值P就是最小值，不存在比这更小的值】，由于⑧式中λh\(x\)一定小于等于零，所以一定有 

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2046.png)

为了找到最优的λ值，我们一定想要L\(x，λ\)接近P，即找到合适的λ最大化L\(x，λ\)，可以写成maxλL\(x，λ\)，所以⑧式求解合适的x，λ值最终可以转换成如下 

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2038.png)

这里我们回到最初的目标，最小化目标函数

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2041.png)

首先应用KKT条件：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2016.png)

且应当满足：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2028.png)

计算w和b

接下来引入软间隔：写成如下形式：

![屏幕截图 2025\-10\-18 135337\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/屏幕截图%202025-10-18%20135337.png)

构造拉格朗日函数：

![屏幕截图 2025\-10\-18 135556\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/屏幕截图%202025-10-18%20135556.png)

上式中*λi,μi*是拉格朗日乘子，*w,b,ξ*是我们要计算的主问题参数。

目标函数是个下凸函数，可以根据强对偶向，将对偶问题转换一下

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2011.png)

分别求导并令其等于0，然后代回原式，整理之后可得：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2020.png)

接下来利用smo算法求解一组合适的λ和μ值：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2027.png)

这里得到的w和b虽然和硬分割结果一样，但是这是加入松弛变量之后得到的w和b的值，根据L式的条件C=λi\+μi和λi≥0,μi≥0，结合h式，C越大，必然导致松弛越小，如果C无穷大，那么就意味着模型过拟合，在训练SVM时，C是我们需要调节的参数。

那么确定w和b之后，我们就能构造出最大分割超平面：wT⋅x\+b=0，新来样本对应的特征代入后得到的结果如果是大于0那么属于一类，小于0属于另一类

总结一下:svm就是使用拉格朗日，kkt条件，对偶问题来求的线或超平面的参数W和b，从而对目标进行分类的一个算法。

决策树与随机森林

### 决策树

决策树是一种以**树形结构进行决策的模型**（通过模拟人的决策过程，构建一颗类似于流程图的树结构，它从根节点开始，对数据特征进行判断，并依据结果将数据划分到不同的子节点）。它通过一系列特征判断，最终得出一个离散或连续的输出结果。

举个例子，下面这棵树可以根据天气情况判断是否适合外出玩耍：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2039.png)

**内部节点**表示对某个特征的判断（如天气、湿度）

**叶子节点**表示最终的预测结果

工作原理简述：

每个叶子节点中都包含一部分训练数据。这些数据都满足从根节点到该叶子节点的所有判断条件。当我们要预测一个新样本时，我们根据它的特征路径走到对应的叶子节点，并使用该节点中训练样本的**多数类（分类）**或**平均值（回归）**作为预测结果。

决策树的一些问题：

**容易过拟合（Overfitting）**

决策树在训练时倾向于尽可能多地分裂节点，以提高训练集的准确率。但这会导致模型过于复杂，学习到了训练数据中的噪声，从而在新数据上表现不佳。

解决方法：限制树的深度、**设置最小样本数等剪枝策略。**

**不稳定性（Instability）**

即使训练数据有微小变化（比如去掉几个样本），生成的决策树结构也可能完全不同。这使得模型缺乏一致性。

踩坑提醒：如果你在做模型解释或部署时依赖某棵特定结构的决策树，这种不稳定性会带来一定风险。

### 随机森林

随机森林是一种**集成学习（Ensemble Learning）**方法，它通过构建多个决策树并将它们的预测结果进行整合，来提升模型的性能。

简单来说，随机森林通过构建大量的决策树，并让每颗树分别进行判断，最终通过投票或平均来得出结果。随机森林通常能更好地处理数据的噪声和过拟合问题，因为它通过集成方法**减少了单棵树的偏差和方差。**

具体工作原理可以看下图：

![image\.png](../media/02%20OpenCV%20与视觉基础/opencv入门教学/image%2024.png)

虽然随机森林在性能上优于决策树，但它有一个显著缺点：**难以解释**。

决策树结构清晰，人类可以理解其判断逻辑。

随机森林由多个树组成，整体决策过程复杂，难以可视化和解释。



作者：张航铭 蓝金桥 姚远洋

