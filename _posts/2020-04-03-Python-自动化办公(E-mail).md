---
title: Python基操-自动化办公(E-mail)
layout: post
tags: Python基操
categories: 'Python'
---

### 1.概述

​		在日常的工作中，涉及到自动化数据采集、处理并形成文档，且需要将信息以邮件的方式告知相关工作人员的场景。对于远端的机器，需要执行无人值守，并自动完成数据采集、处理、分析、形成文档和自动邮件发送的功能。这个时候，需要用代码去实现邮件的编写和发送，本节介绍Python的两个内置库`smtplib`和`email`，来实现邮件功能。

### 2.smtplib库

​		SMTP（Simple Mail Transfer Protocol）即简单邮件传输协议,它是一组用于由**源地址**到**目的地址**传送邮件的规则，由它来控制信件的中转方式。python实现发邮件也是基于此基础上进行封装的。SMTP协议只能用来发送邮件，不能用来接收邮件。大多数的邮件发送服务器 (Outgoing Mail Server) 都是使用SMTP协议。**SMTP协议的默认TCP端口号是25**。

​		`smtplib`用法相对来说很简单，就是分为三步。

- 创建SMTP的操作对象并连接smtp目标服务器，可以是126、163、QQ等
- 根据自己的账号登录目标服务器（自己的邮箱地址和邮箱授权码）
- 调用对象中的方法，发送邮件到目标地址

```python
import smtplib
from email.mime.text import MIMEText

mail_host = "smtp.163.com"
mail_sender = "zhengkan1993@163.com"
mail_receivers = ["846774350@qq.com", "1175234362@qq.com", "188261720@qq.com"]
mail_license = "HBUFQCFRNAIJINJC"
msg = MIMEText('邮件发送测试内容')  # 邮件内容

# 创建SMTP对象
stp = smtplib.SMTP()
# 设置发件人邮箱的域名和端口，端口地址为25
stp.connect(mail_host, 25)
# set_debuglevel(1)可以打印出和SMTP服务器交互的所有信息
stp.set_debuglevel(1)
# 登录邮箱，传递参数1：邮箱地址，参数2：邮箱授权码
stp.login(mail_sender, mail_license)
# 发送邮件，传递参数1：发件人邮箱地址，参数2：收件人邮箱地址，参数3：把邮件内容格式改为str
stp.sendmail(mail_sender, mail_receivers, msg.as_string())
print("邮件发送成功")
# 关闭SMTP对象
stp.quit()
```

### 3.email库

​		除简单文本外，很多时候邮件中还会包含HTML、图片、音频、附件等。MIME（Multipurpose Internet Mail Extensions，多用途互联网邮件扩展）作为一种新的扩展邮件格式很好地补充了这一点。

​		可以将email.mime理解成smtplib模块邮件内容主体的扩展，从原先默认只支持纯文本格式扩展到HTML，同时支持附件、音频、图像等格式，smtplib只负责邮件的投递即可。

| 类                                                           |               说明               |
| :----------------------------------------------------------- | :------------------------------: |
| **email.mime.multipart.MIMEMultipart**([`_subtype[,boundary[,_subparts[,_params`]]]]) | **包含多个部分邮件体的MIME对象** |
| **email.mime.audio.MIMEAudio (`_audiodata[,_subtype[,_encoder[,_params`]]])** |   **创建包含音频数据的邮件体**   |
| **email.mime.image.MIMEImage(`_imagedata[,_subtype[,_encoder[,_params`]]])** |   **创建包含图片数据的邮件体**   |
| **email.mime.text.MIMEText (`_text[,_subtype[,_charset`]])** |   **创建包含文本数据的邮件体**   |

说明：

```sh
# MIMEMultipart中的_subtype类型
mixed（默认）：构建一个带附件的邮件体
related：构建内嵌资源的邮件体
alternative：构建纯文本与超文本共存的邮件体

# MIMEAudio中的_audiodata
_audiodata包含原始二进制音频数据的字节字符串

# MIMEImage中的_imagedata
_imagedata是包含原始图片数据的字节字符串

# MIMEText中的_text
_text是包含消息负载的字符串；_subtype指定文本类型，支持plain（默认值）或html类型的字符串
```

```python
from email.header import Header
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart

mm = MIMEMultipart('related')

# 构建邮件头
subject_content = "琵琶行"
mm["Subject"] = Header(subject_content, 'utf-8')
# 构建邮件内容
content = MIMEText(_text='''
浔阳江头夜送客，枫叶荻花秋瑟瑟。主人下马客在船，举酒欲饮无管弦。醉不成欢惨将别，别时茫茫江浸月。忽闻水上琵琶声，主人忘归客不发。寻声暗问弹者谁，琵琶声停欲语迟。移船相近邀相见，添酒回灯重开宴。千呼万唤始出来，犹抱琵琶半遮面。转轴拨弦三两声，未成曲调先有情。弦弦掩抑声声思，似诉平生不得志。低眉信手续续弹，说尽心中无限事。轻拢慢捻抹复挑，初为《霓裳》后《六幺》。大弦嘈嘈如急雨，小弦切切如私语。嘈嘈切切错杂弹，大珠小珠落玉盘。间关莺语花底滑，幽咽泉流冰下难。冰泉冷涩弦凝绝，凝绝不通声暂歇。别有幽愁暗恨生，此时无声胜有声。银瓶乍破水浆迸，铁骑突出刀枪鸣。曲终收拨当心画，四弦一声如裂帛。东船西舫悄无言，唯见江心秋月白。沉吟放拨插弦中，整顿衣裳起敛容。自言本是京城女，家在虾蟆陵下住。十三学得琵琶成，名属教坊第一部。曲罢曾教善才服，妆成每被秋娘妒。五陵年少争缠头，一曲红绡不知数。钿头银篦击节碎，血色罗裙翻酒污。今年欢笑复明年，秋月春风等闲度。弟走从军阿姨死，暮去朝来颜色故。门前冷落鞍马稀，老大嫁作商人妇。商人重利轻别离，前月浮梁买茶去。去来江口守空船，绕船月明江水寒。夜深忽梦少年事，梦啼妆泪红阑干。
我闻琵琶已叹息，又闻此语重唧唧。同是天涯沦落人，相逢何必曾相识！我从去年辞帝京，谪居卧病浔阳城。浔阳地僻无音乐，终岁不闻丝竹声。住近湓江地低湿，黄芦苦竹绕宅生。其间旦暮闻何物？杜鹃啼血猿哀鸣。春江花朝秋月夜，往往取酒还独倾。岂无山歌与村笛？呕哑嘲哳难为听。今夜闻君琵琶语，如听仙乐耳暂明。莫辞更坐弹一曲，为君翻作《琵琶行》。感我此言良久立，却坐促弦弦转急。凄凄不似向前声，满座重闻皆掩泣。座中泣下谁最多？江州司马青衫湿。
''', _subtype="text", _charset="utf-8")
mm.attach(content)
# 构建插图
pic = MIMEImage(_imagedata=open("./source/琵琶行.jpg", "rb").read(), _subtype="base64")
mm.attach(pic)
# 构建附件
txt_file = MIMEText(open('./source/琵琶行.txt', 'rb').read(), 'base64', 'utf-8')
txt_file["Content-Disposition"] = 'attachment; filename="琵琶行.txt"'
mm.attach(txt_file)
```

### 4.综合一封邮件

```python
import smtplib
from email.header import Header
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart
from email.mime.application import MIMEApplication

mail_host = "smtp.163.com"
mail_sender = "zhengkan1993@163.com"
mail_receivers = ["zhengkan@mywind.com.cn", "846774350@qq.com"]
mail_license = "HBUFQCFRNAIJINJC"

mm = MIMEMultipart('related')

# 构建邮件头
subject_content = "琵琶行"
mm["Subject"] = Header(subject_content, 'utf-8')
mm["FROM"] = "<zhengkan1993@163.com>"
mm["TO"] = "<zhengkan@mywind.com.cn>"
# 构建邮件内容
with open("./source/琵琶行.txt", "r", encoding="utf-8") as f:
    text = f.read()

# 构建插图
with open("./source/琵琶行.jpeg", "rb") as f:
    pic = MIMEImage(f.read(), _subtype="base64")
pic.add_header('Content-ID', '<image1>')
pic["Content-Disposition"] = 'attachment; filename="琵琶行.jpeg"'
mm.attach(pic)

# 整合内容（图片+文字）
content1 = MIMEText('''
<html>
    <body>
    {}\n
       
        <img src="cid:image1" alt="image1">
    </body>
</html>
'''.format(text), 'html', 'utf-8')  # 正文
mm.attach(content1)

# 构建附件
txt_file = MIMEApplication(open('./source/琵琶行.txt', 'rb').read())
txt_file.add_header('content-disposition', 'attachment', filename="琵琶行.txt")
mm.attach(txt_file)
xlsx_file = MIMEApplication(open('./source/琵琶行.xlsx', 'rb').read(), 'base64')
xlsx_file.add_header('content-disposition', 'attachment', filename="琵琶行.xlsx")
mm.attach(xlsx_file)
pic_file = MIMEApplication(open('./source/琵琶行.jpeg', 'rb').read(), 'base64')
pic_file.add_header('content-disposition', 'attachment', filename="琵琶行.jpeg")
mm.attach(pic_file)

# 创建SMTP对象
stp = smtplib.SMTP()
# 设置发件人邮箱的域名和端口，端口地址为25
stp.connect(mail_host, 25)
# set_debuglevel(1)可以打印出和SMTP服务器交互的所有信息
stp.set_debuglevel(1)
# 登录邮箱，传递参数1：邮箱地址，参数2：邮箱授权码
stp.login(mail_sender, mail_license)
# 发送邮件，传递参数1：发件人邮箱地址，参数2：收件人邮箱地址，参数3：把邮件内容格式改为str
stp.sendmail(mail_sender, mail_receivers, mm.as_string())
print("邮件发送成功")
# 关闭SMTP对象
stp.quit()
```

注意：图片暂时无法显示，网上资料说是网易的不行

### 参考链接：

1.[python发送各类QQ邮件 —— smtplib与email模块](https://blog.csdn.net/Hehuyi_In/article/details/103325304)













