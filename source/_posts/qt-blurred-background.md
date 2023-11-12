---
title: 用 Qt 实现带有模糊背景的窗体
date: 2022-11-15 13:03:34
tags: ["C plus plus", "Qt", "Windows"]
categories: 编程
description: "实践出真知"
---
## 静态背景
这种背景比较好实现，大致思路就是提前对要模糊的区域截图，再放置一个带有模糊效果的Label显示出来。想要加多层合成的话也很方便。Qt Creator 的示例页面就使用了这种效果。
![图 - Qt Creator 上的模糊效果](https://i2.100024.xyz/2022/11/15/lp5t90.webp "图 - Qt Creator 上的模糊效果")

### 优点
- 只使用了 **Qt 提供的 api**，可以完全不加修改地跨平台使用
- 性能开销**小**，速度快

### 缺点
- *无法动态响应*被模糊区域的更新

### 代码&效果
```cpp
//...

//更新背景
void MainWindow::onTimerCaptureTimeout()
{
	QPixmap pix = QApplication::primaryScreen()->grabWindow(0, x(), y(), width(), height());
	ui->lblBlur->setPixmap(pix);
}
```
以下是运行效果:
![图 - 运行效果](https://i2.100024.xyz/2022/11/18/nlc6tf.webp
)

## 实时更新的背景
由于各平台差异较大，Qt 未提供统一解决方案，需要自行使用**平台特异性方法**解决。主要思路与上面的类似，但难点在于如何获取被一个窗口遮挡的内容的截图。多次尝试和搜索未果后，我在 [Microsoft Learn](https://learn.microsoft.com/zh-cn/windows/win32/api/winuser/nf-winuser-setwindowdisplayaffinity) 上看到了这个函数`SetWindowDisplayAffinity`，它的参数中有这样一个选项:

> WDA_EXCLUDEFROMCAPTURE
>
> 窗口仅显示在监视器上。 在其他地方，窗口根本不显示。
> 此相关性的一个用途是用于显示视频录制控件的窗口，以便捕获中不包含控件。
>
> Windows 10版本 2004 中引入。 请参阅有关早期版本的 Windows 兼容性的备注。

这不正是我想要的吗？经过测试，这种方法完全可行
![图- 前后对比](https://i2.100024.xyz/2022/11/15/mbsdn3.webp "图- 前后对比")
但在 CSDN上，效果应该是这样的
![图 - CSDN 的效果](https://i2.100024.xyz/2022/11/15/md32o9.webp "图 - CSDN 的效果")
原因是，这种方法仅对 Windows 10 2004 以上的版本可用~~(显然那些辣鸡作者并不同意巨硬的规定)~~

### 优点
- 可**动态**更新背景，《稍 加 调 教》后有望高仿亚克力效果

### 缺点
- 系统要求*严格*
- 性能开销**大**
- 目前仅可在 Windows 上实现

### 代码&效果
```cpp
#include <Windows.h>

//...

//构造函数
MainWindow::MainWindow(QWidget *parent)
	: QWidget(parent)
	, ui(new Ui::MainWindow)
{
	ui->setupUi(this);

#ifdef Q_OS_WINDOWS
	SetWindowDisplayAffinity((HWND)winId(), WDA_EXCLUDEFROMCAPTURE);
#endif

//...

}

//...

//更新背景
void MainWindow::onTimerCaptureTimeout()
{
	QPixmap pix = QApplication::primaryScreen()->grabWindow(0, x(), y(), width(), height());
	ui->lblBlur->setPixmap(pix);
}
```
运行效果:
![图 - 运行效果](https://i2.100024.xyz/2022/11/18/nlc6tf.webp)

## 收获与感悟
- 多翻文档，总能有意外收获
- 实践是检验真理的唯一标准
- 抵制 CSDN，从我做起
## 相关信息
[代码仓库](https://github.com/Hatsushigure/MyQtAcrylicWidget)