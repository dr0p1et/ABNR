# 思维碎片

非常喜欢这个翻译，Piece Of Mind，记录所有的好玩想法。

* 目录
  
  * 如何fuzz无法导致crash的memory leak
  
  * 如何Fuzz RDP Client

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
