+++
title = "MOX VOC 空气传感器并不像你想象的那样工作"
date = "2025-01-24T17:47:09+08:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = ""
authorTwitter = "" #do not include @
cover = "https://s2.loli.net/2025/01/24/1SNGpPEj3xfhgmk.webp"
tags = ["voc", "sensor"]
keywords = ["voc", "sensor"]
description = "消费级 MOX VOC 传感器仅能表示相对过去一段时间内，传感器目前所在地空气质量的相对值。绝对值没有实际意义。原因？请继续阅读..."
showFullContent = false
readingTime = false
hideComments = false
+++

## 为什么会这样？
- 消费级 MOX VOC 传感器由于设计原因，无法准确得出当前环境下，空气质量的绝对值：
    - MOX VOC 传感器对空气内不同气体成分的敏感度不同，输出的值与空气中实际含有什么成分、含有多少成分没有直接联系 [^1]；
    - 然而，VOC 并不都是有害的，也不都是无害的[^2]。无法准确测量每种气体的成分，也就难以仅通过 VOC 指标获得准确的室内空气质量。
- 为此，近年来新出的消费级 MOX VOC 传感器目前大都仅输出一个指标，以期待获得更加有意义的室内空气指标；
- 该指标是相对于过去一段时间内，当前室内空气质量的相对值，非绝对值。换言之：
    - 该值无法告诉你当前环境下是否依然存在**过去一段时间内存在的 VOCs**；
    - 如果室内空气质量在长于过去一段时间后依然糟糕，那么 MOX VCC 的补偿算法会将这些糟糕的空气质量指标映射到“正常”的基准值，并以此为基准得出后续的空气质量（有可能是所谓的“良好”！）[^3]。
- 因此，MOX VCC 传感器实际上只能告诉你：
    - 在过去一段时间内，VOC 值是否发生了相对变化。
        - 你可以把它想象成人的“鼻子”。人在一个时间待得很长之后，将无法分辨当前环境空气内还存留的气味。然而，一旦去到别的地方，人的鼻子就很容易分辨出与先前环境空气气味的不同。这也是 Sensiron 自己对于自家 SGP41 MOX 传感器 VOC Index 如何工作的比喻 [^4]。
- 而不能告诉你：
    - 目前室内空气（以室内当前有害气体的含量为基准）质量如何。
- 而且比较麻烦的是，目前没有准确的设计资料，告诉我们 VOC 指标是否为线性变化。该值呈现怎样的统计规律暂不可知。

## 为什么要研究这个？
起因是 pokon548 在使用 BME680 传感器的时候，发现一些很不符合常理的输出结果：
- 在传感器执行校准时，AQI 发生非常大的波动，从 10 ~ 200 左右不等。然而从国内气象站的数据来看，外部环境空气质量此时并未发生巨大的波动；
- 关机并开机，AQI 初始值一定是 50[^5]。该值需要等待很长一段时间（aka 校准）才能反映目前空气质量的相对状态；
- 即便室外空气质量未改善，室内未开启空气净化器等，AQI 也会在一定时间后变为“良好”。

这使得 pokon548 开始质疑 MOX VOC 传感器的准确性。于是 pokon548 开始在网上查找资料，并最终得出了上面的那些研究结论。

## 如何获得更精确的 AQI 绝对值？
目前 pokon548 并没有什么很好的想法。原因是 pokon548 的消费能力并不能达到企业级的水准，不能直接拿到专业准确的气体传感器（一套传感器下来大约上千上万起步！）。

因此，也许在消费级领域，只能购买 >2 个相同型号的 MOX VOC 传感器。一些放在室内，另一些放在室外，并通过其它可信渠道，获得最近气象站的室内空气指数，并比较室外 / 室内传感器输出的 AQI，执行算法映射，得出相对更精确的室内 AQI 绝对值。

你有好点子吗？请务必在下方评论区分享一下 :)

[^1]: https://sensirion.com/media/documents/4B4D0E67/6520038C/GAS_AN_SGP4x_BuildingStandards_D1_1.pdf
[^2]: https://iaqscience.lbl.gov/introduction-vocs
[^3]: 基于博世对旗下 BME680 传感器 AQI 算法的介绍：[https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme680-ds001.pdf](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme680-ds001.pdf)。实际上，目前市面上所有的 MOX 传感器均拥有共同的设计缺陷，因此这一点实际上也适用于其它输出 AQI 的消费级 MOX VOC 传感器
[^4]: https://sensirion.com/media/documents/02232963/6294E043/Info_Note_VOC_Index.pdf
[^5]: 这是博世 BME680 传感器的 AQI 初始基准值。对于 Sensirion，该值应该为 100