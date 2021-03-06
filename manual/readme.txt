LCR原理

原理如图所示，利用互阻（跨阻）放大电路传输特性进行变换

[1.png]


推导过程如下：

[2.png]


 

 

所以 需要测量4个量

[3.png]


 

 

详细内容请参考 附件中的 安捷伦阻抗测量手册 PDF文件。

 

上面说明说明了自平衡电桥测阻抗的基本原理 即获得 电压 电流的 实部和虚部即可。那这又是怎么实现的呢？
应该怎样实现采集呢？

方法一：在正弦波电压Vx的0°和90°的这个瞬间用采集速度够高的ADC分别采集Vx和Vr的值，按上面的公式计算即可，可惜没那么快速度的ADC。

 

 方法二：根据采样定理，用正弦波频率10倍以上的采样频率的ADC实时对Vx和Vr采样 ，然后用数学算法计算两个正弦的幅值和相位差即可，玩DSP的童鞋可能比较喜欢。但是这里ADC对速度的要求没有方法一那么高，但是一旦测量频率升高那么ADC的速度要求也随之提高，对处理器的性能要求也升高。

 

方法三：类似方法一将0°和90°信号分开测量，但所不同的是

①通过开关将Vx 在0°到180°这段时间的Vx或者Vr的信号取平均送入ADC，180°到360°这段时间将接地信号送入ADC，这样可以或者实部；

 ②通过开关将Vx 在90°到270°这段时间的Vx或者Vr的信号取平均送入ADC，270°到90°这段时间将接地信号送入ADC，这样可以或者虚部；

 

为什么可以这样？？？？？

 

 以纯电阻为例，先对①的条件下Vx和Vr取平均（怎么取？积分呗 ）可以分别得到 a的平均和c的平均，然后②的条件下Vr取平均值，这里结果是0，为什么？因为从坐标轴上看90°到270°这一段的信号是关于180°中心对称，抵消了，而其余时间是接地的0（实际上也可以用积分推导）。 是不是符合纯电阻特性？ 如果不是纯电阻 ②的条件平均值就不是0了 这个与相位差有关。

 

但是绝对 0°，90°，180°之类的很难做到，上面只需要三次采集的 不躲不做四次采集就是Vx的虚部

 

 

这样的好处是可以使用低速的ADC  但也因不停地切换带来噪声，所以不得不进行取平均

 

上面发的俄版电路图就是根据方法三做的

正弦信号使用MCU（暂定使用STM32F103C8，定时器功能不错）产生方波，然后通过带通滤波器即可。原理：将方波的阶跃函数用傅里叶级数展开就可以看到基频和多次谐波，将多次谐波滤掉即可。暂定几个常用频率 100Hz，120Hz，1KHz，10KHz，至于100K则因需要高速运放，暂时不考虑。

 

被测阻抗（以下简称Zx）的电流部分由互阻放大器进行I-V转换，为了适应不同Zx，有必要做增益控制。另外最大转化电流与运放最大输出电流有关，如果运放输出功率变大，则芯片会发热，反馈电阻也会发热，这样会影响测量精度。

 

在电压输出部分有必要做一个衰减来减小输出电压，当然也不能无限制的小。衰减的办法由两个：一，因为使用的是带通滤波，可以借助不匹配的带通网络来衰减，这个衰减的很厉害（与带通的设计参数有关），虽然可以测量小阻抗时给自平衡电桥带来便利，但是会给后续采集电路带来麻烦；二，电阻网络，这个方法的衰减增益比较难控制，也需要考虑。如果不做增益，就需要改变输出阻抗，这对测量小于输出阻抗1/100的Zx也比较麻烦。

 

Zx电压测量部分，也因各种输出电压的关系，需要做一个可控增益，可以使用运放搭成仪表放大，但是电阻的匹配精度不容易做好，这里选择程控仪表放大。

 

数据采集的ADC 在积分型和ΔΣ之间犹豫，但是仔细看了ADS1118的数据手册，发现其频率响应里对 100Hz 1Kz 几乎都在衰减低谷，这会严重降低信噪比，故最终还是采用积分型ADC 

