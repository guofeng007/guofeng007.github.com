---
layout: post
title: 摄影教训总结
categories: [Blog,Cat]
description: 历次出游摄影的一些失败的点。
keywords: 摄影, 教训
---

### 2014 年 8 月 杭州

1. 以人为主体没错，但是要有远有近，拍出来全是脸和上半身也不好。
2. 偶尔兼顾景色，不是所有情况下虚化背景都是加分。
3. 一般情况下色彩鲜艳的衣服拍出来会更好看。
4. 人像镜头不是万能的，大光圈容易虚焦，取景范围影响构图，这些都需要更好地考虑。

### 2014 年 6 月 武汉

1. 照顾好拍摄对象的心情，她开心和配合了，才容易出好片。
2. 如果她真的不愿意拍，就不拍了吧，否则出来基本也是废的。
3. 蓝天白云绿水真的很容易出好片，但是阴天加浑浊的长江边还是不要轻易挑战了。

### 2014 年 3 月 武汉

1. 气氛要活泼一点，不然模特摆拍容易表情僵硬。
2. 光圈优先在室外光线好的情况下容易过曝。
3. 只挑拍得最好的片发给模特看，如果没有出好片，宁愿不发。

###将 Excel 里的数据使用 VBA 导入 Word 表格中。

**前置工作：**将 Word 文档空表格当作模板文档做好，与 Excel 数据源文件置于同一路径下。

```vb
Sub 分离()
    Application.ScreenUpdating = False
    
    p = ThisWorkbook.Path & "/"
    f = p & "空白模板.doc"
    
    Dim myWS As Worksheet
    Set myWS = ThisWorkbook.Sheets(1) '存有数据的表格
    
    For i = 3 To 54    '遍历数据行
        FileCopy f, p & "test/" & myWS.Cells(i, 2).Text & ".doc"    '复制空模板并以某列数据为名命名新产生的文档
        Set wd = CreateObject("word.application")
        Set d = wd.documents.Open(p & "test/" & myWS.Cells(i, 2).Text & ".doc") '打开新文档
        
        d.tables(1).Cell(1, 2) = myWS.Cells(i, 2).Text '###
        '复制表格每列内容到文档，有多少项就有多少条
        d.tables(1).Cell(5, 4) = myWS.Cells(i, 20).Text '###
        
        d.Close
        wd.Quit
        Set wd = Nothing
    Next
    
    Application.ScreenUpdating = True
End Sub
```

