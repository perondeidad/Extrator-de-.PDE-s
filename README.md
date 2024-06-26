texto escrito por letleon
# !Unpde 解密 FC .PDE 文件

## 此仓库仅可用于学习目的，不可用于商业目的！

## 说明

#### FC 版本:1.0.1.9920

[百度网盘](https://pan.baidu.com/s/1MkWH6L6bsx3PJQWJuslwKw?pwd=fcfc "百度网盘")

[OneDrive](https://1drv.ms/f/s!AhZa6xQIJXNNjUoVv3AFIUxeb-Ck?e=6N12WI "OneDrive")

[123 云盘](https://www.123pan.com/s/b5Y0Vv-rN4J3.html)

## 测试程序

[Unpde.exe](TestRelease/Unpde.exe)

请不要用于商业目的！

#### 目前状态：

小部分文件无法解，测试后发现可解压任意版本 PDE 文！

.lua .mesh 等文件似乎是自定义格式!

想要完整获取内容，需要在 Program.cs中手动添加目录

目录的地址就需要你自行搜索了

### TODO

- [x] 除了FC的PDE文件，还可以解压AS的PDE文件
- [x] 可以解压目录下任意pde文件
- [x] 修改逻辑后，可解压 1.0.1.9920 和 1.0.6.7960 版本的 PDE 数据，理论上可解任意版本 PDE 数据
- [x] 优化最终解密逻辑，判断某些不用二次解密的文件
- [x] 优化 Offsetlog，可以保存为树状结构，更直观查找
- [x] fsb 音乐文件可以用 [FSB 提取器 16.10.21 (aezay.dk)](http://aezay.dk/aezay/fsbextractor/) 正确播放，不在使用 [foobar2000](https://www.foobar2000.org/download)
- [x] 找到了两个目录地址
- [x] 不能二次解密的文件会后.cache 后缀，没有.cache 后缀的是被二次解密的文件，有些虽然没有.cache 后缀，可能会有部分文件不正常。
- [ ] 最终解密逻辑，有些文件会越界，有些没问题，还得跟~
- [ ] lua 文件二次解密后似乎还是加密的！
- [x] 初步完成貌似最终解密逻辑!🎉
- [x] 跟踪了最终解密逻辑，发现相当复杂~有空再搞把！
- [x] 找到了貌似最终解密的逻辑！！！🎉
- [x] 解密后的 DDS 文件不能正确显示！怀疑 DDS 文件被修改了！
- [ ] 未知 170 个块不知是干啥的！
- [x] 错误 解密后的 FSB 音乐以及 SWF 文件有些不完整！
- [ ] 某些目录 比如 1580 处，写的是 7000 大小 实际只有 1000 大小，导致读取的数据量不正常！还得跟~
- [x] lua 似乎不正确或是加密的！
- [x] 一个猜测，dds tag lua 可能不是真正的文件，可能只是个指针？
- [ ] 发现很多偏移值并不是只读了一次，而且有些偏移值的每次读的大小都不同！
- [x] 未跟踪 解密时大小超过 1000H 时(也就是 KEY 大小)，如何继续循环？已解决，Key 循环就行了！
- [x] 读取大资源文件时，不知从哪里抓到的偏移值，怀疑时 170 表里的值！
- [x] 已知 解密 Fun4 是解 170 表
- [x] 已知 解密 Fun1 / 2 是解密文件/文件夹表
- [x] 已知 解密 Fun 3 是解密实际单个文件的函数
- [x] 简单模拟实现了 Fun 3 抓文件夹或文件的逻辑
- [x] 简单实现了利用 1000H 的目录数据，批量导出了 1188 个文件,但文件不完整!!!
- [x] 可以以目录的方式写入磁盘
- [x] 可以输出一个解密后的 HEX 作为调试时使用
- [x] 可以输出一个全部偏移值的 JSON 供调试时使用
- [x] 添加了文件验证类，在没有找到真正文件数据之前先顶着！
- [x] 目前可解 4000 个文件，但大多数我认为不是真正的文件
- [x] OffsetLog 改成嵌套结构

---

# Build

#### 准备:

系统 windows10/11

安装 visual studio 2022 (选择 .NET 桌面开发)

打开源码，之后右键 管理解决方案的 NuGet 程序包,下载 Newtonsoft.Json v13.0.3

#### 编译:

首先你需要先 build 程序，然后会生成 Unpde\bin\x86\Debug\net8.0 目录

将 v1.0.1.9920 版本的 finalcombat.pde 文件

放置到 Unpde\bin\x86\Debug\net8.0 目录中

在比 build 或 run 即可在 Unpde\bin\x86\Debug\net8.0 目录下创建一个新的目录 Unpde

这个目录就是解密的结果目录了！

# 解密流程?

先读 170 个块！
接着读 1000H
1000H 是根目录
解密 -> 读取文件夹或文件
读取文件 -> 获取偏移值 -> 读取 -> 解密 -> 完成
实际上是，读取文件 -> 获取偏移值 -> 读取 -> 解密 -> 在处理好几次之后放入内存供程序使用
读取目录 -> 获取偏移值 -> 读取 -> 解密 ->
如果内容里有文件就按照 读取文件 的逻辑读取解密
如果还是目录 就循环执行 读取目录 逻辑

---

# 线索

## Key 获取

在 00A60950 Xor 解密函数中

往下 到达这个循环，观察 EDI ，EDI 的位置就是 KEY 的数据，

大小是 1000H（ecx）

![key](images/key.png)

## 最终解密逻辑(验证后发现比我想的要复杂)

在 004CFA90 函数处
将初次解码的文件数据 再次处理

![D1.png](images/D1.png)

循环读取初次解码的数据然后处理

![D2.png](images/D2.png)

从内存复制处某个 DDS，DDS 可被正确识别并显示
像是个弹痕！

![D4.png](images/D4.png)

## 文件偏移值与大小获取方法 (26A4A000H)

```
文件名
  boss01wep01skeleton.skel.cache
文件偏移量
  B8690200
  即 000269B8 + 1 = 000269B9H
  即可得到偏移量，在经过计算即可得到在PDE中的实际地址

文件大小
  56000000
  即56H，此文件的大小
```

![offset1](images/offset1.jpg)

## 文件与文件夹的标识以及块大小

```
文件夹 02
文件 01
块大小 80H -> 128
```

![offset2](images/offset2.jpg)

![offset4](images/offset4.jpg)

![offset3](images/offset3.jpg)

---

# Heads

**Head 分析结果如下**

```
黄色 -> head: 010A0000 02000100
青色 -> 从0X18开始到结尾的大小: 6DD90500 -> 0X0005D96D
红色 -> 文件数量？
绿色 -> 解码后大小: E83D0822 -> 0X00083DE8
实际参与解码的数据:
	从0x21开始(紫色竖线)到结尾
最终文件:
	需要移除前8个字节
```

![head_info4](images/head_info4.png "head_info4")

## .lua

010A0000 02000100

## .anim

00050000 02000300

## .fsb

46534234

## .tga / .dds

01020000 02000100

## .dcl

000C0000 02000200

## .swf

4357530A

## .ttf

00010000 00120100 00040020 44534947

## .mesh

00010000 02000800

## .occ

010A0000 02000100

## .physx

00070000 02000500

## .pd9

02040000 02000300

## .vd9

01040000 02000300

## .skel

00060000 02000300

## .spr

000D0000 02000000

## .vfx

01020000 02000100

---

# 预览

## FSB 音乐文件播放器

#### [~~foobar2000~~](https://www.foobar2000.org/download)

~~需要用到这个老牌播放器
[vgmstream decoder](https://www.foobar2000.org/components/view/foo_input_vgmstream)~~

~~以及这个插件即可播放 FSB 音乐文件
(但现在解出的资源不完整！！！)~~

#### [FSB 提取器 16.10.21 (aezay.dk)](http://aezay.dk/aezay/fsbextractor/)

可正确识别 fsb 文件内的所有音频，以及可以提取音频

#### Radio 部分 可用在线预览

[百度网盘 -&gt; 解码测试 &gt;音乐解码测试 &gt;](https://pan.baidu.com/s/1MkWH6L6bsx3PJQWJuslwKw?pwd=fcfc)

---

## DDS 预览

#### [Texture Tools Exporter | NVIDIA Developer](https://developer.nvidia.com/texture-tools-exporter)
