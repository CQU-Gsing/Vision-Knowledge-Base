# Python 速通课

GSing 视觉组入门资料，参考《C 语言速通课》的章节组织和讲解方式。

> **视觉组入门推荐学习 Python**，便于衔接图像处理、数据处理和模型调用。会一种编程语言即可，C 或 Python 都可以；已经会 C 的同学也可以先用熟悉的语言实践，不要求两种都学。

### 学习目标：学到什么程度可以继续？

你不必把 Python 所有高级特性学完。能做到下面这些，就可以边做视觉项目边补知识了：

- 能自己运行 `.py` 文件，确认解释器和安装库的环境。
- 理解缩进、条件、循环、函数和返回值。
- 能用列表和字典组织数据，知道 `None` 表示没有结果。
- 能分清赋值、修改对象和复制数据。
- 能导入模块、读写文件，并根据报错找到自己的代码位置。
- 能独立解释综合练习的输入、处理过程和输出，而不只是复制运行。

---

### 写在前面

这是一部 Python 速通课，目标是让你能读懂代码、自己写出小程序，并接着学习 OpenCV 和 YOLO。很多知识先讲到“能用、知道为什么这样用”为止，更深的内容可以在实践中补上。

如果你已经学过 C，可以重点看环境、缩进、列表与字典、对象引用和模块。完全没有编程基础的话，就按顺序来。每个例子都建议自己运行，再改一两个数值看看结果；只看完文章，和能自己写出来，还是两回事。

建议先用几天跑通基础，不必给自己规定“必须看完所有语法才能动手”。本课默认使用 Python 3，标记为 `python` 的代码写进 `.py` 文件；标记为 PowerShell 或 Bash 的命令在终端里执行。

### Lesson 1 : 前置基础以及环境搭建

#### Part 1 计算机是如何跑起来的

##### Python 能用来做什么？

先看一个熟悉的任务：摄像头拍下一张图，我们希望程序找出里面的目标，再把结果告诉机器人。拆开来就是：

1. 输入：接收图像。
2. 处理：根据规则或模型分析图像。
3. 输出：给出类别、位置或其他结果。

Python 可以把这些步骤串起来。它也能批量整理文件、处理表格、发送网络请求。对视觉组来说，我们经常用 Python 验证想法、处理数据、训练和调用模型。

但 Python 本身不会“自动识别图片”。语言负责表达流程，OpenCV、PyTorch 等库提供图像处理和计算工具，我们还要写清楚具体怎么做。

##### 代码为什么能运行？

计算机不能直接理解我们写下的 `print("hello")`。还需要一个 Python **解释器**，负责执行 Python 程序。

以常见的 CPython 实现为例，源代码通常会先编译成字节码，再由解释器执行。这里不用背执行细节，只要理解：`.py` 文件保存代码，解释器负责运行它；最终仍由计算机硬件完成运算。

写 Python 入门程序不需要配置 GCC。GCC 是另一套编译工具，不是运行 `.py` 文件的必要步骤。

#### Part 2 我们如何写程序

##### 先分清三个东西

| 工具 | 作用 | 本课使用方式 |
| --- | --- | --- |
| Python 解释器 | 执行 Python 代码 | 用 `python` 命令运行文件 |
| VS Code | 编写代码、查看提示、调试 | 打开项目文件夹并选择解释器 |
| 终端 | 输入命令 | 创建环境、安装库、运行程序 |

VS Code 不等于 Python，安装好编辑器后，还需要让它找到解释器。

##### 下载与验证

- Python 下载：[Python 官方下载页面](https://www.python.org/downloads/)。选择 Python 3 的稳定版本，按页面提供的安装方式完成安装。
- 编辑器下载：[VS Code 官网](https://code.visualstudio.com/)。安装后，在扩展中安装 Microsoft 发布的 Python 扩展。

如果使用传统 Windows 安装包，留意将 Python 加入 PATH 的选项；如果使用官方安装管理器，则按其引导安装解释器。以后学习 PyTorch、YOLO 时，还要确认这些库支持你选择的 Python 版本。

打开新终端，检查：

```powershell
python --version
```

如果 `python` 找不到，Windows 上也可以检查 `py --version`。如果 `py` 可用，下文创建环境时可以把 `python -m venv .venv` 换成 `py -m venv .venv`；不是所有电脑都同时有这两个命令。

##### 给项目建一个独立环境

不同项目可能需要不同的库版本。虚拟环境相当于给每个项目准备自己的工具箱，避免把所有依赖混在一起。

先新建一个练习文件夹，比如 `D:\python_practice`，用 VS Code 打开它，再打开终端。以下是 **Windows PowerShell** 命令：

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -c "import sys; print(sys.executable)"
```

最后输出的路径应指向练习文件夹里的 `.venv\Scripts\python.exe`。激活后，新终端通常会显示 `(.venv)`。

如果 PowerShell 阻止激活脚本，不必为了运行代码立即修改系统设置，可以直接指定环境中的解释器：

```powershell
.\.venv\Scripts\python.exe --version
```

未激活环境时，后续命令中的 `python` 都应替换成 `.\.venv\Scripts\python.exe`，包括安装库时的 `-m pip` 命令，确保工具装进这个项目的环境。

在 Linux/macOS 上，创建和激活命令是：

```bash
python3 -m venv .venv
source .venv/bin/activate
```

如果已经使用 Conda，也可以沿用自己的独立 Conda 环境，不需要在里面再建一层 `.venv`。

在 VS Code 中按 `Ctrl + Shift + P`，选择 **Python: Select Interpreter**，选中这个项目的解释器。选择后打开新终端，并再次检查 `sys.executable`，确认运行代码和安装库使用的是同一环境。

##### 跑起来第一个程序

新建 `hello.py`，写入：

```python
print("hello world")
print("你好，视觉组！")
```

保存后，在这个文件所在的目录运行：

```powershell
python hello.py
```

如果未激活环境，则运行 `.\.venv\Scripts\python.exe hello.py`。

`>>>` 是 Python 交互窗口的提示符，不是终端提示符。`python hello.py` 这样的命令不能输入到 `>>>` 后面。若误进了交互窗口，输入 `exit()` 返回终端。

**小练习：**把输出改成自己的名字，再增加一行输出“我准备学习图像识别”。

### Lesson 2 : 语法规则

#### Part 1 如何写第一个程序

##### 注释与缩进

注释用来解释代码，以 `#` 开头。程序不会执行注释里的内容。

```python
# 记录一次检测的置信度
confidence = 0.85

if confidence >= 0.5:
    print("保留这个目标")
    print("这两行都在 if 内部")

print("这一行不属于 if，无论条件是否成立都会执行")
```

Python 用**缩进**表示代码属于哪个块，常用四个空格。`if` 后面的冒号表示接下来有一个代码块，缩进不是为了好看，而是语法的一部分。

不要在同一段代码中混用 Tab 和空格。写错缩进时，先看编辑器的对齐，不要只盯着变量名。

##### 输入、输出与类型转换

`print()` 把结果显示出来，`input()` 从键盘读取一行内容。要注意：**`input()` 得到的是字符串，即使你输入了数字。**

```python
name = input("你的名字：")
count = int(input("今天标注了多少张图片："))

print(f"{name} 今天标注了 {count} 张，明天再标 {count + 10} 张。")
```

`int()` 把合适的输入转换成整数，`float()` 可以转换成小数。输入“十张”再执行 `int()` 会报错，后面会讲怎样处理。

`f"..."` 是格式化字符串，花括号中的表达式会被替换成结果：

```python
score = 0.8567
print(f"置信度：{score:.2f}")  # 输出：置信度：0.86
```

#### Part 2 基础语法

##### 变量：给数据起个名字

```python
count = 3
confidence = 0.92
label = "cup"
found = True
target = None

print(type(count))
print(type(confidence))
print(type(label))
```

变量不需要先写 `int` 或 `float` 声明。Python 会在运行时根据对象知道它的类型；这不意味着类型可以不管，例如字符串和整数不能随意相加。

| 类型 | 常见写法 | 用途 |
| --- | --- | --- |
| `int` | `3` | 数量、编号 |
| `float` | `0.92` | 置信度、距离等数值 |
| `str` | `"cup"` | 名称、路径、消息 |
| `bool` | `True`、`False` | 表示条件是否成立 |
| `None` | `None` | 表示没有结果或尚未赋予具体值 |

同一个名字可以重新绑定不同类型的对象，但不要为了展示这个能力而乱改变量含义。`count` 一会儿是数量、一会儿是文件名，会让代码很难读。

变量名区分大小写，`score` 和 `Score` 是两个名字。建议使用 `image_width` 这样的名称，不要把自己的变量命名为 `list`、`str`、`print`，否则会遮住原本的工具。

##### 运算符和表达式

```python
print(7 + 2)   # 9
print(7 - 2)   # 5
print(7 * 2)   # 14
print(7 / 2)   # 3.5：普通除法
print(7 // 2)  # 3：向下取整除法
print(-7 // 2) # -4：不是简单去掉小数部分
print(7 % 2)   # 1：余数
print(2 ** 3)  # 8：乘方
```

`=` 是赋值，`==` 是比较。写 `score = 0.5` 是给变量赋值；写 `score == 0.5` 才是在问它是否等于 0.5。

```python
score = 0.8
distance = 1.2

usable = score >= 0.5 and distance < 2.0
print(usable)          # True
print(not usable)     # False
print(score < 0.5 or distance >= 2.0)
```

比较还能写成 `0 <= x < width`。逻辑运算用 `and`、`or`、`not`；`&` 和 `|` 是另一类运算符，不要在普通条件中随便替换。

##### 字符串：处理名字和文字

```python
filename = "image_001.jpg"
print(filename.endswith(".jpg"))
print(filename.replace(".jpg", ".txt"))

line = "0 0.5 0.4 0.2 0.1"
fields = line.split()
print(fields)      # ['0', '0.5', '0.4', '0.2', '0.1']
print(fields[0])   # '0'，仍然是字符串
print(int(fields[0]))
```

`split()` 在这里按空白分开字符串。字符串不可变，`replace()` 返回的是新字符串，不会原地修改原来的字符串。

##### 列表：把一组数据放在一起

列表可以装多个值，也可以增加、删除或修改元素。索引从 0 开始。

```python
scores = [0.2, 0.8, 0.95]
print(scores[0])    # 0.2
print(scores[-1])   # 0.95，最后一个
print(scores[1:3])  # [0.8, 0.95]，包含起点，不包含终点

scores.append(0.7)
scores[0] = 0.3
print(len(scores))  # 4
print(scores)
```

列表长度是 4 时，合法的非负索引是 0、1、2、3，访问 `scores[4]` 会抛出 `IndexError`。切片却允许终点超过长度，它会取到列表结尾。

##### 元组：一组不打算修改的位置数据

```python
center = (120, 80)
x, y = center
print(x, y)

single = (120,)  # 一个元素的元组需要这个逗号
print(type(single))
```

元组不能修改自己的元素槽位，所以不能直接写 `center[0] = 200`。不过，元组里如果装了一个列表，那个列表本身仍然可以修改；“元组不可变”不意味着里面所有对象都不可变。

##### 字典：给一条记录加上字段名

```python
target = {
    "label": "cup",
    "confidence": 0.91,
    "center": (120, 80),
}

print(target["label"])
print(target.get("distance", "尚未测距"))
target["confidence"] = 0.95
```

相比记住列表第几个元素代表什么，字典中的 `"confidence"` 明确说明了字段含义。直接访问不存在的键会抛出 `KeyError`，`get()` 则可以给出默认值。

##### 集合：去重与判断是否出现过

```python
labels = ["cup", "bottle", "cup"]
unique_labels = set(labels)
print(sorted(unique_labels))  # ['bottle', 'cup']
print("cup" in unique_labels) # True
```

集合中的元素不会重复，也不能像列表那样用位置索引。不要依赖它的遍历顺序。空集合写成 `set()`，`{}` 是空字典。

#### Part 3 控制流

##### if：满足条件才做

```python
score = 0.72

if score >= 0.8:
    print("较高置信度")
elif score >= 0.5:
    print("保留，继续检查")
else:
    print("过滤掉")
```

程序按顺序判断，只执行第一个满足条件的分支。把 `score` 改成 0.3 或 0.95，再观察输出。

##### for：遍历一组数据

```python
scores = [0.2, 0.8, 0.95]

for index, score in enumerate(scores):
    if score < 0.5:
        continue
    print(f"第 {index} 个目标：{score}")

for i in range(3):
    print(i)  # 依次是 0、1、2
```

`enumerate()` 同时给出位置和数值，`range(3)` 不包含 3。`continue` 跳过当前这次循环，接着处理下一个元素。

##### while 与 break：条件成立就继续

```python
attempt = 0

while attempt < 5:
    attempt += 1
    print(f"第 {attempt} 次检查")
    if attempt == 3:
        break
```

`break` 退出当前循环。摄像头程序经常会持续读取画面，直到用户按键退出；那时也会用到类似结构。

写 `while` 时想一想：什么时候结束？如果条件永远不变，又没有退出路径，程序就会一直执行。

##### 列表推导式：简单的筛选可以短一点

```python
scores = [0.2, 0.8, 0.95]
usable_scores = [score for score in scores if score >= 0.5]
print(usable_scores)  # [0.8, 0.95]
```

它等价于建立空列表，再用循环筛选和追加。逻辑复杂时，正常写多行循环更容易看懂，不用追求把所有事情塞到一行里。

#### Part 4 函数

##### 把重复操作取个名字

```python
def box_center(x1, y1, x2, y2):
    return (x1 + x2) / 2, (y1 + y2) / 2


cx, cy = box_center(10, 20, 50, 60)
print(cx, cy)  # 30.0 40.0
```

`def` 定义函数，参数是调用时传入的数据，`return` 将结果交回调用者。这里返回一个包含两个值的元组，可以用 `cx, cy` 接住。

**打印结果和返回结果不是同一件事。**只调用 `print()` 而没有 `return` 的函数，执行完默认返回 `None`。

##### 默认参数与关键字参数

```python
def keep_scores(scores, threshold=0.5):
    return [score for score in scores if score >= threshold]


print(keep_scores([0.2, 0.6, 0.9]))
print(keep_scores([0.2, 0.6, 0.9], threshold=0.8))
```

默认参数允许调用者省略某个值，关键字参数则让调用更容易读。不过，列表这种可变对象不要随手作为默认参数，原因在下一课讲。

##### 局部变量与作用范围

```python
threshold = 0.5


def choose_threshold():
    threshold = 0.8
    return threshold


print(choose_threshold())  # 0.8
print(threshold)           # 0.5
```

函数内部赋值创建的是局部绑定，不会自动改变外部同名变量。入门时优先通过参数传入数据、通过 `return` 返回结果，不要到处用全局变量传递状态。

##### 类型提示：帮人和工具看懂代码

```python
def double(value: float) -> float:
    return value * 2


print(double(1.5))
```

`value: float` 和 `-> float` 是类型提示。它们能帮助编辑器检查代码，但不会自动做运行时类型转换；输入数据仍需要你验证。

**这一课的小练习：**写一个函数，接收一组置信度，返回高于指定阈值的目标数量。先用循环写出来，再试试用列表推导式。

### Lesson 3 : 对象、引用与内存

#### Part 1 变量不是一份数据的独占盒子

Python 里的名字绑定到对象。执行一次赋值，不一定会复制对象。看这个例子：

```python
scores_a = [0.2, 0.8]
scores_b = scores_a

scores_b.append(0.9)
print(scores_a)  # [0.2, 0.8, 0.9]
print(scores_b)  # [0.2, 0.8, 0.9]
```

两个名字指向同一个列表，所以通过其中一个名字修改列表，另一个名字看到的内容也改变了。列表和字典是常见的**可变对象**。

再看整数：

```python
count_a = 3
count_b = count_a
count_b += 1

print(count_a)  # 3
print(count_b)  # 4
```

整数是不可变对象，这里的 `+= 1` 让 `count_b` 绑定到另一个整数对象，而不是把原来的整数 3 改成 4。字符串也属于不可变对象。

#### Part 2 复制一份，和起另一个名字，有什么不同？

```python
scores_a = [0.2, 0.8]
scores_b = scores_a.copy()
scores_b.append(0.9)

print(scores_a)  # [0.2, 0.8]
print(scores_b)  # [0.2, 0.8, 0.9]
```

`.copy()` 创建一个新的外层列表。不过这是**浅拷贝**，列表中如果装着其他可变对象，里面的对象仍然可能共享：

```python
from copy import deepcopy

original = [[1, 2], [3, 4]]
shallow = original.copy()
shallow[0][0] = 99
print(original)  # [[99, 2], [3, 4]]

independent = deepcopy(original)
independent[0][0] = 100
print(original)     # [[99, 2], [3, 4]]
print(independent)  # [[100, 2], [3, 4]]
```

遇到嵌套数据，先判断哪些内容需要共享、哪些内容需要独立。不是所有地方都要深拷贝，盲目复制图像这类大数据也会增加开销。

#### Part 3 函数到底会不会改变外面的数据？

不要只记“Python 是传值”或“Python 是传引用”这样的口号。看看函数是在**修改对象**，还是在**给局部变量重新绑定**，就容易理解了。

```python
def append_score(scores):
    scores.append(0.9)


def replace_scores(scores):
    scores = [1.0]
    return scores


values = [0.2]
append_score(values)
print(values)          # [0.2, 0.9]

new_values = replace_scores(values)
print(values)          # [0.2, 0.9]
print(new_values)      # [1.0]
```

第一个函数修改了共享的列表。第二个函数只让自己的局部名字指向新列表，并把新列表返回，没有替换外面的 `values`。

##### 可变默认参数为什么容易出问题？

函数默认值在定义时求值，不是每次调用都重新建立。把列表写成默认值，会导致不同调用共用它。入门时用下面的写法：

```python
def add_record(label, records=None):
    if records is None:
        records = []
    records.append(label)
    return records


print(add_record("cup"))     # ['cup']
print(add_record("bottle"))  # ['bottle']，没有混入上一条
```

不要把这里的默认值直接改成 `records=[]`，除非你确实理解并希望共享这个对象。

##### == 和 is 也不是一回事

```python
left = [1, 2]
right = [1, 2]

print(left == right)  # True：内容相等
print(left is right)  # False：不是同一个对象

result = None
print(result is None)
```

判断普通数值或字符串的内容用 `==`。判断是否为 `None`，通常写 `is None`。

Python 通常不需要你手动申请和释放普通对象的内存，但文件、摄像头等资源仍需要及时关闭。后面会用 `with` 和 `finally` 来保证清理。

#### Part 4 类：把数据和相关操作组织在一起

先看一个小类，不必急着学继承或复杂框架：

```python
class Target:
    def __init__(self, label, confidence):
        self.label = label
        self.confidence = confidence

    def is_usable(self, threshold=0.5):
        return self.confidence >= threshold


target = Target("cup", 0.91)
print(target.label)
print(target.is_usable())
```

`Target` 是类，`target` 是一个实例；`__init__` 在实例创建时初始化属性。`self` 指向调用这个方法的实例，用它访问该实例的数据。

一条简单记录可以先用字典表示。当同类数据有共同的操作、需要长期维护状态时，再考虑用类来组织。看懂这种结构，就能进一步理解 YOLO 模型对象或 ROS2 节点代码。

### Lesson 4 : 写一个真正能用的小项目

#### Part 1 import、模块与程序入口

##### 别人写好的功能怎么拿来用？

```python
import math
from pathlib import Path

print(math.sqrt(9))
print(Path("image_001.jpg").suffix)
```

`math`、`pathlib` 是标准库，安装 Python 后就可以使用。OpenCV、NumPy、Ultralytics 等第三方库则需要另外安装到当前环境。

```powershell
python -m pip install numpy opencv-python
```

`python -m pip` 表示用这个 Python 解释器对应的 pip 安装包。只输入 `pip` 时，在装有多个 Python 的电脑上更容易装错环境。

`import cv2` 对应安装包名 `opencv-python`，安装名和导入名不一定相同。不要把自己的文件命名为 `cv2.py`、`json.py`、`torch.py`，否则可能挡住你原本想导入的库。

##### 自己写的文件也能成为模块

在同一个练习文件夹中建立 `utils.py`：

```python
def box_center(x1, y1, x2, y2):
    return (x1 + x2) / 2, (y1 + y2) / 2
```

再建立 `main.py`：

```python
from utils import box_center


def main():
    print(box_center(10, 20, 50, 60))


if __name__ == "__main__":
    main()
```

运行 `python main.py`。`__name__ == "__main__"` 用来判断这个文件是不是被直接运行；如果它被其他文件导入，这个分支不会执行。Python 不强制你定义一个叫 `main` 的函数，但这样组织项目通常更清楚。

#### Part 2 异常：出错时也要知道发生了什么

```python
text = input("请输入图片数量：")

try:
    count = int(text)
except ValueError:
    print("请输入整数，例如 10。")
else:
    print(f"下一步准备处理 {count} 张图片。")
```

`try` 放可能出错的操作，`except ValueError` 只处理这类转换错误。不要用一个空的 `except` 把所有错误吞掉，否则程序失败了，你却不知道原因。

阅读报错时，从最后一行看错误类型和说明，再沿着调用信息找到自己代码的位置。常见的错误：

| 错误 | 先检查什么 |
| --- | --- |
| `SyntaxError` / `IndentationError` | 冒号、括号、缩进是否正确 |
| `NameError` | 是否拼错名字，或变量还没定义 |
| `TypeError` | 数据类型是否符合操作要求 |
| `ValueError` | 值是否能转换，或是否满足要求 |
| `IndexError` / `KeyError` | 索引或键是否存在 |
| `ModuleNotFoundError` | 当前解释器及环境中是否安装了这个包 |
| `FileNotFoundError` | 文件实际位置和程序使用的路径 |

#### Part 3 文件、路径与 JSON

##### 相对路径是相对谁？

`open("result.txt")` 通常从**当前工作目录**寻找文件，不是自动从代码文件所在的位置找。这就是“明明图片就在旁边，程序却找不到”的常见原因。

下面代码保存为 `path_demo.py` 再运行；它用 `__file__` 找到脚本所在目录，不能原样贴到没有 `__file__` 的交互窗口中：

```python
from pathlib import Path

root = Path(__file__).resolve().parent
output = root / "output"
output.mkdir(parents=True, exist_ok=True)

file_path = output / "result.txt"
with file_path.open("w", encoding="utf-8") as file:
    file.write("今天成功运行了 Python 程序。\n")

with file_path.open("r", encoding="utf-8") as file:
    print(file.read())
```

`with` 会在代码块结束时关闭文件，包括发生异常时。`"w"` 会覆盖原文件；想追加内容时用 `"a"`，不要写入后才发现覆盖了自己的记录。

Windows 路径可以写成 `Path("D:/data/images")` 或 `Path(r"D:\data\images")`。不要把反斜杠路径里的 `\n` 意外当成换行符。

##### JSON：保存一份结构化结果

```python
import json
from pathlib import Path

result = {
    "label": "cup",
    "confidence": 0.91,
    "center": [120, 80],
}

path = Path("demo_result.json")
path.write_text(
    json.dumps(result, ensure_ascii=False, indent=2),
    encoding="utf-8",
)
loaded = json.loads(path.read_text(encoding="utf-8"))
print(loaded["label"])
```

JSON 适合保存这类字符串、数字、布尔值、列表和字典组成的数据。不是所有 Python 对象都能直接转成 JSON，比如 NumPy 数组通常要先转成普通列表。

#### Part 4 综合练习：处理一组模拟检测结果

现在把前面的列表、字典、函数、循环和文件处理接起来。

**注意：下面处理的是手工准备的模拟结果，不会自己识别图像。**真实项目里，可以将数据来源替换成 YOLO 的检测输出。

将完整代码保存为 `target_demo.py`，运行 `python target_demo.py`：

```python
import json
from pathlib import Path


def select_target(detections, threshold=0.5):
    candidates = []
    for item in detections:
        if item["confidence"] < threshold:
            continue
        x1, y1, x2, y2 = item["box"]
        if x2 <= x1 or y2 <= y1:
            continue
        candidates.append(item)

    if not candidates:
        return None
    return max(candidates, key=lambda item: item["confidence"])


def main():
    detections = [
        {"label": "cup", "confidence": 0.3, "box": [10, 20, 50, 60]},
        {"label": "bottle", "confidence": 0.91, "box": [100, 80, 180, 200]},
        {"label": "cup", "confidence": 0.75, "box": [220, 90, 260, 150]},
    ]
    chosen = select_target(detections)
    if chosen is None:
        print("没有符合要求的目标。")
        return

    x1, y1, x2, y2 = chosen["box"]
    result = {
        "label": chosen["label"],
        "confidence": chosen["confidence"],
        "center_pixels": [(x1 + x2) / 2, (y1 + y2) / 2],
    }
    output = Path(__file__).resolve().parent / "target_result.json"
    output.write_text(
        json.dumps(result, ensure_ascii=False, indent=2),
        encoding="utf-8",
    )
    print(result)
    print(f"结果已写入：{output}")


if __name__ == "__main__":
    main()
```

正常情况下选中 `bottle`，中心坐标为 `[140.0, 140.0]`。`max(..., key=...)` 用指定字段比较，`lambda` 在这里是一个只取置信度的小函数。

这里选择最高置信度，只是练习 Python。机器人实际选哪个目标，还可能考虑距离、类别、可达性和动作状态，不能简单照搬这个选择规则。

**再改三件事：**

1. 把阈值改成 0.95，确认程序能处理没有目标的情况。
2. 增加一个高置信度但宽高不合法的框，确认它会被过滤。
3. 增加一个函数，统计每种类别有多少个有效目标。

#### Part 5 衔接视觉：用 Python 打开一张图片

学到这里就可以进入 OpenCV 了。先把一张图片命名为 `sample.jpg`，放到下面代码所在目录，保存代码为 `read_image.py`：

```python
from pathlib import Path

import cv2


def main():
    image_path = Path(__file__).resolve().parent / "sample.jpg"
    image = cv2.imread(str(image_path))
    if image is None:
        raise FileNotFoundError(f"图片不存在、无法解码或路径不可读：{image_path}")

    height, width = image.shape[:2]
    print(f"图片宽度：{width}，高度：{height}")
    try:
        cv2.imshow("Sample image", image)
        cv2.waitKey(0)
    finally:
        cv2.destroyAllWindows()


if __name__ == "__main__":
    main()
```

`cv2.imread()` 读取失败通常返回 `None`，不要直接对失败结果访问 `.shape`。默认读取的彩色图像通道顺序为 BGR；传给要求 RGB 的其他工具前，要确认是否需要转换。

显示窗口需要桌面图形环境。没有图形界面的服务器上，不应直接照搬 `imshow()`，可以改为保存结果后查看。

建议继续阅读：[前置](../前置.md)、[opencv入门教学](<../02 OpenCV 与视觉基础/opencv入门教学.md>)、[YOLO实践文档](<../03 YOLO 目标检测/YOLO实践文档.md>)。现有 OpenCV 入门文章主要使用 C++ 示例，Python 学习者可以结合 OpenCV 的 Python 教程理解相同处理步骤。

### 官方资料

- [Python 官方教程](https://docs.python.org/zh-cn/3/tutorial/index.html)：按主题查语法，不必第一次就从头读完。
- [venv 官方说明](https://docs.python.org/3/library/venv.html)：虚拟环境的创建与使用。
- [VS Code Python 环境说明](https://code.visualstudio.com/docs/python/environments)：解释器选择与项目环境。
- [OpenCV Python 教程](https://docs.opencv.org/4.x/d6/d00/tutorial_py_root.html)：图像读取、处理及后续实践。
