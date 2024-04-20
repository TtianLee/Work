
# So-VITS-SVC操作流程

[so-vits-svc项目](https://github.com/svc-develop-team/so-vits-svc)

其实跟着项目本身的readme做就行，提点需要注意的。

## 📝 最简单跑起来需要的代码

具体哪些参数可以调，去项目里看一目了然。

### 1. 重采样至 44100Hz 单声道

```shell
python resample.py
```

#### 注意

虽然本项目拥有重采样、转换单声道与响度匹配的脚本 resample.py，但是默认的响度匹配是匹配到 0db。这可能会造成音质的受损。而 python 的响度匹配包 pyloudnorm 无法对电平进行压限，这会导致爆音。所以建议可以考虑使用专业声音处理软件如`adobe audition`等软件做响度匹配处理。若已经使用其他软件做响度匹配，可以在运行上述命令时添加`--skip_loudnorm`跳过响度匹配步骤。如：

```shell
python resample.py --skip_loudnorm
```

### 2. 自动划分训练集、验证集，以及自动生成配置文件

```shell
python preprocess_flist_config.py --speech_encoder vec768l12
```

### 3. 生成 hubert 与 f0

```shell
python preprocess_hubert_f0.py --f0_predictor dio
```

### 4.主模型训练

```shell
python train.py -c configs/config.json -m 44k
```

###  5.推理

```shell
# 例
python inference_main.py -m "logs/44k/G_30400.pth" -c "configs/config.json" -n "yoursongsname.wav" -t 0 -s "yourrolename"
```

必填项部分：
+ `-m` | `--model_path`：模型路径
+ `-c` | `--config_path`：配置文件路径
+ `-n` | `--clean_names`：wav 文件名列表，放在 raw 文件夹下
+ `-t` | `--trans`：音高调整，支持正负（半音）
+ `-s` | `--spk_list`：合成目标说话人名称
+ `-cl` | `--clip`：音频强制切片，默认 0 为自动切片，单位为秒/s

把前面的依赖环境、编码器、底模什么的下好，这个流程下来肯定能跑通的。



## 📝注意事项

### 扩散模型

------

跑的step跟主模型比例大概1：7，跑的步数太多，电音更严重

### 数据集

------

自己录干声吧，感觉网上找不太到，也可能我不会找（
看别人说数据集至少100条，每条5s-10s这样，最好一共是1个小时。【我跑的最好的模型数据集就60+，300条的反而没那个好，真是让人百思不得其解，可能质量不太好吧。多试试。】

##### 数据处理工具

[一键跳转下载](https://pan.baidu.com/s/1YGBzf1ZUKjP0tBcz7zBBYQ?pwd=agwd)

主要下音频处理工具里的后3个：
+ foobar 一键转wav格式
+ Rix 细节处理歌曲的干声
+ Au 自不必多说
+ 第一个uvr也能下，但比较慢，建议这么下[UVR5](https://link.zhihu.com/?target=https%3A//github.com/Anjok07/ultimatevocalremovergui/releases/download/v5.5.0/UVR_v5.5.1_setup.exe)

切片工具：[Audio Slicer](https://link.zhihu.com/?target=https%3A//github.com/flutydeer/audio-slicer/releases)

<img src="https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/v2-f2374d867d68371604157933102a834b_720w.webp" style="zoom: 80%;" />

- Threshold（阈值）：以 dB（分贝）表示的 RMS 阈值。所有 RMS 值都低于此阈值的区域将被视为静音。如果音频有噪音，增加此值。
- Minimum Length（最小长度）：以默认值 5000（毫秒为单位）为例，简单来说就是切割后的每个音频片段都不少于 5 秒。
- Minimum Interval（最小间距）：以默认值 300（毫秒为单位）为例，简单来说就是少于 0.3 秒的静音不会被切割丢掉，超过 0.3 秒的静音部分才丢掉。如果音频仅包含短暂的中断，请将此值设置得更小。此值越小，此应用程序可能生成的切片音频剪辑就越多。请注意，此值必须小于 minimum length 且大于 hop size。
- Hop Size（跳跃步长）：每个 RMS 帧的长度（说白了就是精度），以毫秒为单位。增加此值将提高切片的精度，但会降低处理速度。默认值为 10。
- Maximum Silence Length（最大静音长度）：在切片音频周围保持的最大静音长度，以毫秒为单位。根据需要调整此值。请注意，设置此值并不意味着切片音频中的静音部分具有完全给定的长度。如上所述，该算法将搜索要切片的最佳位置。默认值为 1000。

### 推理歌曲处理

------

使用uvr5分离伴奏与人声
四个步骤，分离伴奏和（人声+和声+混响），分离和声与（人声+混响），分离人声和混响，分离人声和噪音
最终得到伴奏，和声，人声去混响。
用到的模型没有，就去小扳手里download center 里找了下载放目录里。
具体请看图：

**分离伴奏和（人声+和声+混响)**

<img src="https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/63b7e35a93df81326a4aa3482f087a3.png" style="zoom: 50%;" />

**分离和声与（人声+混响）**

<img src="https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/51773fe946c2530749c4d5c6786f5a6.png" style="zoom:50%;" />

**分离人声和混响**

<img src="https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/5c1e2328700d0842ca64753c3079333.png" style="zoom:50%;" />

**分离人声和噪音**

<img src="https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/f9d001cc0f36e58ca13f39e4adab7f3.png" style="zoom:50%;" />

**最终保留的文件**

![](https://cdn.jsdelivr.net/gh/TtianLee/Image@main/img/032223c23f579c5452850bce86071c9.png)

把第一个改个名放进项目raw文件就行。


### 突然想到的注意

看经验贴说跑个8000-12000步比较好，实践下来确实。看你存的model，G_XXX.pth，XXX=8000左右的模型就还行。

关于遇到越跑越电音，感觉可能是数据集不太行。

### 问题

关于webUI用不了，少了个包，安装又开始wheel咋咋咋——尚未开始解决
