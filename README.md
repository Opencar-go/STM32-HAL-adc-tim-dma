# test
stm32-HAL adc-tim-dma
# STM32——三重ADC交替采样—极限采样率7.2M

© 本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接。  
原文链接：[https://blog.csdn.net/2301_79131841/article/details/141086606](https://blog.csdn.net/2301_79131841/article/details/141086606)  [oai_citation:0‡CSDN博客](https://blog.csdn.net/2301_79131841/article/details/141086606?spm=1001.2014.3001.5501)

## 三重ADC交替采样

### 目录
- 三重ADC交替采样  
- 前言：  
- 正文：  
  - 时钟配置  
  - 基础配置  
  - 串口配置  
  - TIM定时器配置  
  - ADC配置  
  - DMA配置  
  - 代码示例  
- 结语：

---

### 前言：
本教程使用 STM32F407VET6，基于 CubeMX 与 Keil5 软件开发。刚刚参加完 2024 年全国大学生电子设计竞赛，在备赛过程中学习了很多信号类相关知识。在信号类题目中，ADC 是一个复杂且实用的要点，而与其紧密相关的几个参数就是 —— **采样率、采样精度**。STM32F407 单个 ADC 最大采样率大约为 2.4 M，你在有些时候可能会发现仍不能满足需求，这时 **三重ADC交替采样** 值得认真学习了。本次将给大家分享“三重ADC交替采样”的配置与使用。

关于三重 ADC 的原理，这里就不过多解释，主要给大家分享用法，便于大家快速使用。  
原理可参考这位博主的文章 —— 《stm32教程之三重ADC交错采样》。  [oai_citation:1‡CSDN博客](https://blog.csdn.net/2301_79131841/article/details/141086606?spm=1001.2014.3001.5501)

### 正文：

#### 时钟配置：
由于不同型号单片机开启单ADC 2.4 M 采样率的方法不完全相同，所以这里就以 2 M 采样率为示例，最终采样率约为 6 M。  
【插入图片】 ![Image](https://bbs.21ic.com/data/attachment/forum/202509/05/163811ohkc085wlad8m8lq.png)

#### 基础配置：
【插入图片】 ![Image](https://i-blog.csdnimg.cn/direct/f65f5da0a4704db2a08883324f0faf23.png)

#### 串口配置：
串口重定义的方法这里就不再赘述，网上教程很多。  
【插入图片】 ![Image](https://i-blog.csdnimg.cn/blog_migrate/812028c74f5a7f33ce6c7119dc37f7c9.png)

#### TIM定时器配置：
实际采样率为 2 M × 3 ≈ 6 M（通过更改时钟可达到 7.2 M）  
【插入图片】 ![Image](https://i-blog.csdnimg.cn/blog_migrate/6e354016b7d5c3e1821e003bfc63e616.png)

#### ADC配置：
【插入图片】 ![Image](https://i-blog.csdnimg.cn/direct/f65f5da0a4704db2a08883324f0faf23.png)

#### DMA配置：
关于 DMA 模式——循环与普通模式的区别，字节长度（word 与 half-word 的区别）可参考：  
【插入图片】 ![Image](https://i-blog.csdnimg.cn/blog_migrate/810cd0d7a29d529d523eedf5279df4e2.png)

#### 代码示例：
```c
#define ADC1_DMA_Size       1024
float    ADC1_ConvertedValue[ADC1_DMA_Size];
uint32_t ADC_Value          [ADC1_DMA_Size/2];
uint8_t  DMA_FLAG = 0;

// 启动代码
HAL_ADC_Start(&hadc2);
HAL_ADC_Start(&hadc3);
HAL_ADCEx_MultiModeStart_DMA(&hadc1,(uint32_t*)ADC_Value,512);
HAL_TIM_Base_Start(&htim8);

void HAL_ADC_ConvCpltCallback(ADC_HandleTypeDef *hadc)
{
    DMA_FLAG = 1;
}

while (1)
{
    if (DMA_FLAG == 1)
    {
        DMA_FLAG = 0;
        for(int i=0,m=0; m<512 && i<1024; )
        {
            ADC1_ConvertedValue[i] = (float)(ADC_Value[m] & 0x0000FFFF) * 3.3f / 4095.0f;
            i++;
            ADC1_ConvertedValue[i] = (float)((ADC_Value[m] & 0xFFFF0000)>>16) * 3.3f / 4095.0f;
            i++;
            m++;
        }
        for(int i=0; i<1024; i++)
        {
            printf("%f\r\n", ADC1_ConvertedValue[i]);
        }
    }
}
