---
title: Python基操-自动化办公(Pdf)
layout: post
tags: Python基操
categories: 'Python'
---

### 1.概述

​		PDF（Portable Document Format）是日常办公中常用的文件格式，使用.pdf作为扩展名，用于可靠的呈现和交换文档，与软件、硬件和操作系统无关。

### 2.PyPDF2库

​		日常办公中，对于`PDF`文件的制作，一般通过在`Office Word`中进行文档的编辑工作，随后通过`另存为pdf`生成得到。本节主要讲解对生成后的`PDF`文件的处理，包括合并、拆分、读取、旋转等功能。

​		主要用到的第三方库为`PyPDF2`

```sh
# 安装PyPDF2
>>> pip install PyPDF2
>>>pip show pypdf2
Name: PyPDF2
Version: 1.26.0
Summary: PDF toolkit
Home-page: http://mstamy2.github.com/PyPDF2
...
```

`PyPDF2`库主要包括 `PdfFileReader`、`PdfFileWriter`、`PageObject`和`PdfFileMerge`共四个大类，分别负责pdf文件的读取、写出、页码管理和合并

#### 2.1读取PDF

​		读取pdf文件用到 `PdfFileReader`类，详解一下这个类下面的方法

| 方法                                    | 说明                                         |
| --------------------------------------- | -------------------------------------------- |
| decrypt (*password*）                   | 加密文档                                     |
| getDestinationPageNumber(*destination*) | 获取目标页（destination为 int类型）          |
| getDocumentInfo()                       | 获取文档基本信息                             |
| getIsEncrypted()                        | 判断是否加密                                 |
| getNumPages()                           | 获取总页数                                   |
| getPage()                               | 获取页面信息，返回一个页面对象**PageObject** |

```python
from PyPDF2 import PdfFileReader

file = r"./sources/Deep Forecast Deep Learning-based Spatio-Temporal Forecasting.pdf"
reader = PdfFileReader(file)

# 获取文档信息
print(reader.getDocumentInfo())
# 获取是否加密信息
print(reader.getIsEncrypted())
# 获取总页数
print(reader.getNumPages())
# 获取最后一页的内容信息
print(reader.getPage(reader.getNumPages() - 1))
print(reader.getFields(tree=True))
# 获取索引内容
print(reader.getOutlines())
# 导出第一页内容
page = reader.pages[0]
print(page.extractText())
```

#### 2.2PDF拆分

​		拆分PDF主要用到了`PdfFileWriter`类，简单梳理下这个类的方法

| 方法                                     | 说明                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| addPage(*page*)                          | 添加页，page是**page** ([*PageObject*](https://pypdf2.readthedocs.io/en/latest/modules/PageObject.html#PyPDF2.pdf.PageObject))对象，可以通过PdfFileReader获得 |
| addBlankPage(*width=None*,*height=None*) | 添加空白页                                                   |
| ...                                      | 有些方法用不到，可直接在word中完成                           |

```python
from PyPDF2 import PdfFileReader, PdfFileWriter

input_file = r"./sources/Deep Forecast Deep Learning-based Spatio-Temporal Forecasting.pdf"

pdf_file = PdfFileReader(input_file)

# 获取总页数
page_sum = pdf_file.numPages

for num in range(page_sum):
    output_file = r'./sources/splited_pdf/拆分文件第' + str(num + 1) + '页.pdf'
    add_page = pdf_file.getPage(num)
    pdf_output = PdfFileWriter()
    pdf_output.addPage(add_page)
    with open(output_file, 'wb') as f:
        pdf_output.write(f)
```

#### 2.3PDF合并

​		合并PDF主要用到 `PdfFileMerge`类，基本方法介绍下

| 方法                                                         | 说明                                                 |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| close()                                                      | 关闭内存                                             |
| merge (*position, fileobj, bookmark=None, pages=None, import_bookmarks=True*) |                                                      |
| append（*fileobj, bookmark=None, pages=None, import_bookmarks=True*) | 一般参数不设置，直接添加就行，每个子页命名顺序就行了 |

```python
from PyPDF2 import PdfFileMerger
import os

# 三个输入项，分别为输入路径、输出路径和合并后的名字
input_file_path = r'./sources/splited_pdf'
output_file_path = r'./sources/merged_pdf'
merge_name = 'merge_file'

files = os.listdir(input_file_path)  # 列出目录中的所有文件
merger = PdfFileMerger()

for file in files:  # 从所有文件中选出pdf文件合并
    if file.endswith('pdf'):
        file = os.path.join(input_file_path, file)
        merger.append(open(file, 'rb'))

output_file = os.path.join(output_file_path, merge_name) + '.pdf'
with open(output_file, 'wb') as fout:  # 输出文件为newfile.pdf
    merger.write(fout)

```

### 参考链接：

1.[`PyPDF2官网说明`](https://pypdf2.readthedocs.io/en/latest/user/installation.html)