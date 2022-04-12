---
title: Python基操-自动化办公(Excel)
layout: post
tags: Python基操
categories: 'Python'
---

### 1.概述

​		前一篇讲了自动化办公中的`Office Word`，这一篇针对`Office Excel`进行解析。主要用到的库是`openpyxl`、`xlwt`等等库，但是每个库里面的专注的功能不一样，需要根据需要使用。

​		Python 中有大量的原生和第三方 Excel 操作包，各有所长，不过对于刚使用 Python 与 Excel 交互的同学来说，可能有点目不暇接，所以先简单梳理一下常见的一些 Excel 包

- **OpenPyXL** 是个读写 Excel 2010 xlsx/xlsm/xltx/xltm 的 Python 库，简单易用，功能广泛，单元格格式/图片/表格/公式/筛选/批注/文件保护等等功能应有尽有，图表功能是其一大亮点
- **xlwings** 是一个基于 BSD 授权协议的 Python 库，可以轻松的使用 Python 操作 Excel，也可以在 Excel 中调用 Python，以接近 VBA 语法的实现 Excel 编程，支持 Excel 宏，并且可以作为 Web 服务器，提供 REST API 接口
- **pandas** 数据处理是 pandas 的立身之本，Excel 作为 pandas 输入/输出数据的容器
- **win32com** 从命名上就可以看出，这是一个处理 windows 应用的扩展，Excel 只是该库能实现的一小部分功能。该库还支持 office 的众多操作。需要注意的是，该库不单独存在，可通过安装 pypiwin32 或者 pywin32 获取
- **Xlsxwriter** 拥有丰富的特性，支持图片/表格/图表/筛选/格式/公式等，功能与 openpyxl 相似，优点是相比 openpyxl 还支持 VBA 文件导入，迷你图等功能，缺点是不能打开/修改已有文件，意味着使用 xlsxwriter 需要从零开始
- **DataNitro** 一个 Excel 的付费插件，内嵌到 Excel 中，可完全替代 VBA，在 Excel 中使用 python 脚本。既然被称为 Excel 中的 python，同时可以与其他 python 库协同。
- **xlutils** 基于 xlrd/xlwt，老牌 python 包，算是该领域的先驱，功能特点中规中矩，比较大的缺点是仅支持 xls 文件。

概括一下：

- 不想使用 GUI 而又希望赋予 Excel 更多的功能，openpyxl 与 xlsxwriter，二者可选其一；
- 需要进行科学计算，处理大量数据，建议 pandas+xlsxwriter 或者 pandas + openpyxl，是不错的选择；
- 想要写 Excel 脚本，会 Python 但不会 VBA，可考虑 xlwings 或 DataNitro；
- win32com 功能还是性能都很强大，不过需要一定的 windows 编程经验才能上手，它相当于是 windows COM 的封装，另外文档不够完善

![image-20220411211816141](../../assets/images/20200401python_excel/image-20220411211816141.png)



### 2.openpyxl库

```sh
>>> pip install openpyxl
>>> pip show openpyxl
Name: openpyxl
Version: 3.0.9
Summary: A Python library to read/write Excel 2010 xlsx/xlsm files
Home-page: https://openpyxl.readthedocs.io
Author: See AUTHORS

```

在操作`Excel`文件之前需要了解`Exce工作簿`的基本概念

| 基本概念 | 说明                                              |
| -------- | ------------------------------------------------- |
| workbook | 工作簿，即xls文件，每个工作簿都是一个Workbook对象 |
| sheet    | Excel中的表单，每个文档至少一个sheet              |
| cell     | 单元格，每个sheet中都要若干个cell，存储数据       |

#### 2.1 创建工作簿

```python
from openpyxl import Workbook
import datetime
# 创建一个 workbook
wb = Workbook()
# 获取被激活的 worksheet
ws = wb.active
# 设置单元格内容
ws['A1'] = 42
# 设置一行内容
ws.append([1, 2, 3])
# 设置A2的值，前面已经在A2写下的值会被覆盖
ws['A2'] = datetime.datetime.now()
# 保存 Excel 文件
wb.save("第一个工作簿v1.1.xlsx")
```

```python
# 打开一个工作簿
from openpyxl import load_workbook

wb = load_workbook('第一个工作簿v1.1.xlsx')

# 显示文档中包含的 表单 名称
print(wb.sheetnames)
```

`load_workbook` 除了参数 `filename` 外为还有一些有用的参数：

- `read_only`：是否为只读模式，对于超大型文件，要提升效率有帮助
- `keep_vba` ：是否保留 vba 代码，即打开 Excel 文件时，开启并保留宏
- `guess_types`：是否做在读取单元格数据类型时，做类型判断
- `data_only`：是否将公式转换为结果，即包含公式的单元格，是否显示最近的计算结果
- `keep_links`：是否保留外部链接

#### 2.2 操作sheet

​		openpyxl具备对sheet创建、删除、修改、查询的功能

```python
from openpyxl import Workbook
import warnings

warnings.filterwarnings("ignore")
wb = Workbook()
ws = wb.active

ws1 = wb.create_sheet("人员汇总")  # 创建一个名为 人员汇总 的sheet
ws1.title = "人员汇总A"  # 修改 ws1 标题
ws1.sheet_properties.tabColor = "1072BA"  # 设置 sheet 标签背景色

ws2 = wb.create_sheet("收入汇总", 0)  # 创建一个 sheet，插入到最前面 默认插在后面
ws2.title = "收入汇总A"  # 修改 ws2的 标题

# 获取 sheet
ws3 = wb.get_sheet_by_name("收入汇总A")
# ws4 = wb['New Title']

# 复制 sheet
ws1_copy = wb.copy_worksheet(ws1)

# 删除 sheet
wb.remove(ws1)
wb.save("第二个工作簿v2.2.xlsx")

```

- 每个 Workbook 中都有一个被激活的 sheet，一般都是第一个，可以通过 active 直接获取

- 可以通过 sheet 名来获取 sheet 对象

- 创建 sheet时需要提供 sheet 名称参数，如果该名称的 sheet 已经存在，则会在名称后添加 1，再有重复添加 2，以此类推

- 获得 sheet 对象后，可以设置 名称（title），背景色等属性

- 同一个 Workbook 对象中，可以复制 sheet，需要将源 sheet 对象作为参数，复制的新 sheet 会在最末尾

- 可以删除一个 sheet，参数是目标 sheet 对象

  

  对于每个sheet中的行列，可以插入行、插入列、删除行、删除列、移动

```python
# 在第7行之前插入行
ws.insert_rows(7)
# 删除第6列后面的三列，即F:H
ws.delete_cols(6, 3)
# 移动"D4:F10",向上移动1行，向右移动2列，并覆盖原来的值
ws.move_range("D4:F10", rows=-1, cols=2)
# 如果单元格有公式，一起移动需要加上translate；公式这部分放在后面部分讲
 ws.move_range("G4:H10", rows=1, cols=1, translate=True)
```

#### 2.3 操作cell

```python
from openpyxl import Workbook

wb = Workbook()
ws = wb.active

# 创建一个sheet
ws1 = wb.create_sheet("人员数据", 0)


# 通过单元格名称设置
ws1["A1"], ws1["B1"] = "姓名", "年龄"
ws1["A2"], ws1["B2"] = "马三", 20

# 通过行列坐标设置
ws1.cell(row=3, column=1, value="李四")
ws1.cell(row=3, column=2, value=10)
wb.save("Workbook v2.3.1.xlsx")
```

```python
from openpyxl import Workbook,load_workbook

wb = load_workbook("Workbook v2.3.1.xlsx")
ws = wb["人员数据"]
# 操作单列
for cell in ws["A"]:
    print(cell.value)
print("--------------------")
# 操作单行
for cell in ws["1"]:
    print(cell.value)
print("--------------------")
# 操作多列
for column in ws['A:C']:
    for cell in column:
        print(cell.value)
print("--------------------")
# 操作多行
for row in ws['1:3']:
    for cell in row:
        print(cell.value)
print("--------------------")
# 指定范围
for row in ws['A1:C3']:
    for cell in row:
        print(cell.value)
print("--------------------")
```

```python
# 设置整行数据
ws.append((1,2,3))
```

```python
# 合并单元格
ws.merge_cells('A2:D2')
# 解除合并  
# 注意：对于没有合并过单元格的位置调用 unmerge_cells 时回报错
ws.unmerge_cells('A2:D2')

ws.merge_cells(start_row=2,start_column=1,end_row=2,end_column=4)
ws.unmerge_cells(start_row=2,start_column=1,end_row=2,end_column=4)
```

#### 2.4 单元格格式

OpenPyXl 用6中类来设置单元格的样式

| 样式         | 说明 |
| ------------ | ---- |
| NumberFormat | 数字 |
| Alignment    | 对齐 |
| Font         | 字体 |
| Border       | 边框 |
| PatternFill  | 填充 |
| Protection   | 保护 |

```python
from openpyxl import Workbook
from openpyxl.styles import PatternFill, Border, Side, Alignment, Protection, Font, numbers

wb = Workbook()
ws = wb.active

# 创建一个sheet
ws1 = wb.create_sheet("人员数据", 0)

# 通过单元格名称设置
ws1["A1"], ws1["B1"] = "姓名", "年龄"
ws1["A2"], ws1["B2"] = "马三", 20

# 通过行列坐标设置
ws1.cell(row=3, column=1, value="李四")
ws1.cell(row=3, column=2, value=10)

# 设置单元格格式
# 字体
font_style = Font(name='Calibri',  # 字体样式
                  size=11,  # 字体大小
                  bold=False,  # 加粗
                  italic=False,  # 斜体
                  vertAlign=None,  # 垂直对齐
                  underline='none',  # 下划线
                  strike=False,
                  color='00FF0000')
# 填充背景色
fill = PatternFill(fill_type='solid', start_color='FF0000')

# 设置边线
border = Border(left=Side(border_style='thin', color='FF0000'),
                right=Side(border_style='thin', color='FF0000'),
                top=Side(border_style='thin', color='FF0000'),
                bottom=Side(border_style='thin', color='FF0000'),
                )
# 设置数字格式
number_format = numbers.FORMAT_PERCENTAGE

# 设置对齐方式
alignment = Alignment(horizontal='right')

# 设置单元格保护
protection = Protection(locked=True, hidden=False)

ws1.cell(row=1, column=1).font = font_style
ws1.cell(row=1, column=2).fill = fill
ws1.cell(row=2, column=1).border = border
ws1.cell(row=1, column=3, value=0.54).number_format = number_format
ws1.cell(row=2, column=2, value='右对齐').alignment = alignment
ws1.cell(row=2, column=4, value='保护').protection = protection

# 遍历范围内的单元格
for row in ws1['A1:C3']:
    for cell in row:
        cell.font = font_style

# 设置整行
row = ws1.row_dimensions[1]
row.font = font_style

# 设置整列
column = ws1.column_dimensions["A"]
column.font = font_style

wb.save("Workbook v2.4.xlsx")
```

#### 2.5 柱状图

![image-20220412093119700](../../assets/images/20200401python_excel/image-20220412093119700.png)

```python
from openpyxl import Workbook
from openpyxl.chart import BarChart, Reference

wb = Workbook()
ws = wb.active

rows = [
    ('月份', '苹果', '香蕉'),
    (1, 43, 25),
    (2, 10, 30),
    (3, 40, 60),
    (4, 50, 70),
    (5, 20, 10),
    (6, 10, 40),
    (7, 50, 30),
]
# 对于数据录入，同一采用循环添加 append 方法
for row in rows:
    ws.append(row)

chart1 = BarChart()
chart1.type = "col"
chart1.style = 10
chart1.title = "销量柱状图"
chart1.y_axis.title = '销量'
chart1.x_axis.title = '月份'

data = Reference(ws, min_col=2, min_row=1, max_row=8, max_col=3)
series = Reference(ws, min_col=1, min_row=2, max_row=8)
chart1.add_data(data, titles_from_data=True)
chart1.set_categories(series)
ws.add_chart(chart1, "A10")
```

#### 2.6 饼状图

![image-20220412093409918](../../assets/images/20200401python_excel/image-20220412093409918.png)

```python
from openpyxl import Workbook
from openpyxl.chart import PieChart, Reference

data = [
    ['水果', '销量'],
    ['苹果', 50],
    ['樱桃', 30],
    ['橘子', 10],
    ['香蕉', 40],
]

wb = Workbook()
ws = wb.active

for row in data:
    ws.append(row)

pie = PieChart()
pie.title = "水果销量占比"
labels = Reference(ws, min_col=1, min_row=2, max_row=5)
data = Reference(ws, min_col=2, min_row=1, max_row=5)
pie.add_data(data, titles_from_data=True)
pie.set_categories(labels)

ws.add_chart(pie, "D1")
wb.save("Workbook v2.6.xlsx")
```

#### 2.7 补充说明

​		个人认为，openpyxl适用于对表格样式的加工，旨在为用户提供一个经过美化的效果，而其计算功能相较于pandas而言，就相对复杂和鸡肋了。所以，这里不记录openpyxl中的`公式移植功能`，`数据透视表功能`等使用`pandas`能够轻松解决的问题。

​		在数据处理方面，建议使用`pandas`读取`Excel`文件中的数据，进行处理，最后保存到`Excel`文件中。后通过`openpyxl`读取该`Excel`文档，并对指定位置进行加工（字体样式、单元格样式等），最后呈现出一个适合用户阅读的`Excel`文档。

参考链接：

1.[全网最全 Python 操作 Excel库总结！](https://baijiahao.baidu.com/s?id=1715868951495080857&wfr=spider&for=pc)

2.[Excel 神器 —— OpenPyXl](https://zhuanlan.zhihu.com/p/409954748)







