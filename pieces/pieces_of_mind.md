# 思维碎片

非常喜欢这个翻译，Pieces Of Mind，记录所有好玩的想法。

* 目录
  
  * 如何fuzz无法导致crash的memory leak
  
  * 如何Fuzz RDP Client
  
  * 结构感知型Fuzzing

## 1. 如何fuzz无法导致crash的memory leak

Mandiant团队的两篇博客：

[Fuzzing Image Parsing in Windows, Part One: Color Profiles](https://www.mandiant.com/resources/fuzzing-image-parsing-in-windows-color-profiles)

[Fuzzing Image Parsing in Windows, Part Two: Uninitialized Memory](https://www.mandiant.com/resources/fuzzing-image-parsing-in-windows-uninitialized-memory)

通常情况下，只有在memory leak导致程序崩溃的情况下才会被fuzzer捕获到。但如果这个memory leak并没有导致程序崩溃，那普通的fuzzer是无法将其发现的。常见的fuzzer目前只能根据进程是否崩溃来判断是否出现了bug。

但现实世界中又确实存在这样的内存泄露。比如图像的编解码，很有可能出现图像泄露后程序并不会崩溃，而是泄露的内存很有可能会被当作像素数据显示出来。那如何fuzz出这样的情况呢？

博客中给出了一个精巧的思路：

先将图像文件解码成对象，然后获取得到对象的像素数据（一段内存）。对比同一个图像文件两次生成的像素数据是否相同。由于内存泄露出来的数据一般是随机的，如果同一个图像文件两次生成的像素内容不同，那很有可能就出现了内存泄露。

```
#define ROUNDS 20
unsigned char* DecodeImage(char *imagePath)
{
      unsigned char *pixels = NULL;     
      // use GDI or WIC to decode image and get the resulting pixels
      ...
      return pixels;
}
void Fuzz(char *imagePath)
{
      unsigned char *refPixels = DecodeImage(imagePath);     
      if(refPixels != NULL)
      {
            for(int i = 0; i < ROUNDS; i++)
            {
                  unsigned char *currPixels = DecodeImage(imagePath);
                  if(!ComparePixels(refPixels, currPixels))
                  {
                        // the reference pixels and current pixels don't match
                        // crash now to let the fuzzer know of this file
                        CrashProgram();
                  }
                  free(currPixels);
            }
            free(refPixels);
      }
}
```

不得不说，这个思路真是精巧。

## 2. 如何Fuzz RDP Client

BlackHat 2019年关于fuzz RDP客户端的议题：

[Fuzzing-And-Exploiting-Virtual-Channels-In-Microsoft-Remote-Desktop-Protocol-For-Fun-And-Profit](https://i.blackhat.com/eu-19/Wednesday/eu-19-Park-Fuzzing-And-Exploiting-Virtual-Channels-In-Microsoft-Remote-Desktop-Protocol-For-Fun-And-Profit-4.pdf)

议题中使用WinAFL的persistance mode对RDP客户端mstsc.exe中的Mstscax.dll中能够从服务端向客户端传输数据的Virtual channels进行了fuzz。作者考虑的是这一个逻辑涉及的内存操作很多且复杂。剪切板、音视频、打印机、智能卡、挂载磁盘等逻辑都是通过虚拟通道实现的。

经常学习blackhat的议题还是很有必要的。

## 3. 结构感知型Fuzzing策略

知道创宇404实验室大佬的议题：

[一枚字体Crash到Fuzzing的探索]([无干货不KCon｜KCon 2021 部分 PPT 发布](https://paper.seebug.org/1748/))

作者首先通过FormatFuzzer构造改进的010editor tiff模板来生成样本，得到一个crash，感觉有戏。然后对这个模板加入了FormatFuzzer针对模板的变异策略再次生成样本，反而没有得到新的crash。思考可能是因为变异改变了模板结构，导致被判定为不合法的字体文件。因此提出了基于AFL和TinyInst加入代码覆盖率的Fuzzer，又获得一个crash，感觉反馈是个改进的方向。

此时作者思考了一个最核心的问题：FormatFuzzer通过010editor模板生成新的样本，虽然是结构化的，但生成的结构中的数据过于随机化，细化度不够，也就是说虽然生成了目标样本，但与实际的样本内容差别过大，基于模板的样本生成的结构也过于单一。

于是，作者借鉴了AFLSmart的思路，AFLSmart通过调用Peach对输入的现有图片数据进行解析，生成chunk数据，然后对chunk数据进行了算法变异，生成新的有效畸形数据后再组合为新的样本。作者就提出了生成结构树的想法。

输入大量字体，提取结构树和数据表内容。然后对结构树和数据表分别进行变异，最后根据变异后的结构树和数据表重组成新的字体。新的字体也可以在整体上进行再次变异。

大量字体-->结构树和数据-->分别变异-->变异后的结构树、变异后的chunk数据、根据结构树生成的chunk数据-->生成新的字体-->二次变异-->字体样本。

这个过程作者称之为结构感知型Fuzzing策略。个人觉得这是样本生成这个细分领域的下一步的发展方向。如果再结合代码覆盖率和数据流敏感的遗传算法，会极大增强现有Fuzzer的效果。
