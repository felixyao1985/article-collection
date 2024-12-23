# 基于梅尔频谱的音频信号分类识别(Pytorch)_cnn音乐分类代码-CSDN博客
[基于梅尔频谱的音频信号分类识别 (Pytorch)\_cnn 音乐分类代码 - CSDN 博客](https://blog.csdn.net/guyuealian/article/details/120601437?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522bb51be7beb7281c4886e7e741b6d59e9%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=bb51be7beb7281c4886e7e741b6d59e9&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-120601437-null-null.142^v100^pc_search_result_base9&utm_term=%E5%B0%86%E5%A3%B0%E9%9F%B3%E8%BD%AC%E4%B8%BA%E5%A3%B0%E7%BA%B9%E5%9B%BE%E7%89%87&spm=1018.2226.3001.4187) 

## 链接

 [https://blog.csdn.net/guyuealian/article/details/120601437?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522bb51be7beb7281c4886e7e741b6d59e9%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=bb51be7beb7281c4886e7e741b6d59e9&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-120601437-null-null.142^v100^pc_search_result_base9&utm_term=%E5%B0%86%E5%A3%B0%E9%9F%B3%E8%BD%AC%E4%B8%BA%E5%A3%B0%E7%BA%B9%E5%9B%BE%E7%89%87&spm=1018.2226.3001.4187](https://blog.csdn.net/guyuealian/article/details/120601437?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522bb51be7beb7281c4886e7e741b6d59e9%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fall.%2522%257D&request_id=bb51be7beb7281c4886e7e741b6d59e9&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-4-120601437-null-null.142^v100^pc_search_result_base9&utm_term=%E5%B0%86%E5%A3%B0%E9%9F%B3%E8%BD%AC%E4%B8%BA%E5%A3%B0%E7%BA%B9%E5%9B%BE%E7%89%87&spm=1018.2226.3001.4187) 

## 备注:

基于梅尔频谱的音频信号分类识别 (Pytorch)

## 基于梅尔[频谱](https://so.csdn.net/so/search?q=%E9%A2%91%E8%B0%B1&spm=1001.2101.3001.7020)的音频信号分类识别 (Pytorch)

**目录**

[基于梅尔频谱的音频信号分类识别 (Pytorch)](#%E5%9F%BA%E4%BA%8E%E6%A2%85%E5%B0%94%E9%A2%91%E8%B0%B1%E7%9A%84%E9%9F%B3%E9%A2%91%E4%BF%A1%E5%8F%B7%E5%88%86%E7%B1%BB%E8%AF%86%E5%88%AB%28Pytorch%29)

[1. 项目结构](#1.%20%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84)

[2. 环境配置](#2.%20%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE)

[3. 音频识别基础知识](#4.%E9%9F%B3%E9%A2%91%E8%AF%86%E5%88%AB%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)

[(1) STFT 和声谱图 (spectrogram)    ](#%281%29%20STFT%E5%92%8C%E5%A3%B0%E8%B0%B1%E5%9B%BE%28spectrogram%29%20%C2%A0%20%C2%A0)

[(2) 梅尔频谱](#%282%29%C2%A0%E6%A2%85%E5%B0%94%E9%A2%91%E8%B0%B1)

[(3) 梅尔频率倒谱 MFCC](#%283%29%C2%A0%E6%A2%85%E5%B0%94%E9%A2%91%E7%8E%87%E5%80%92%E8%B0%B1)

[(4) MFCC 特征的过程](#%284%29%20MFCC%E7%89%B9%E5%BE%81%E7%9A%84%E8%BF%87%E7%A8%8B)

[4. 数据处理](#3.%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%EF%BC%881%EF%BC%89%E6%95%B0%E6%8D%AE%E9%9B%86Urbansound8K%C2%A0)

[（1）数据集 Urbansound8K ](#%EF%BC%881%EF%BC%89%E6%95%B0%E6%8D%AE%E9%9B%86Urbansound8K%C2%A0)

[（2）自定义数据集](#%EF%BC%882%EF%BC%89%E8%87%AA%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E9%9B%86)

[（3）音频特征提取: ](#%EF%BC%883%EF%BC%89%E9%9F%B3%E9%A2%91%E7%89%B9%E5%BE%81%E6%8F%90%E5%8F%96%3A%C2%A0)

[5. 训练 Pipeline](#4.%E8%AE%AD%E7%BB%83Pipeline)

[6. 预测 demo.py](#5.%E9%A2%84%E6%B5%8Bdemo.py)

[7. 源码下载](#7.%E6%BA%90%E7%A0%81%E4%B8%8B%E8%BD%BD)

* * *

本项目将使用 Pytorch，实现一个简单的的音频信号分类器，可应用于机械信号分类识别，鸟叫声信号识别等应用场景。 

项目使用 librosa 进行音频信号处理，backbone 使用 mobilenet_v2，在 Urbansound8K 数据上，最终收敛的准确率在训练集 99%，测试集 96%，如果想进一步提高识别准确率可以使用更重的 backbone 和更多的数据增强方法。

【完整的项目代码】：[基于梅尔频谱的音频信号分类识别 (Pytorch)](https://mp.weixin.qq.com/s/du1F3aeWYWl_2yKvumP1zg "基于梅尔频谱的音频信号分类识别 (Pytorch)")

【尊重原创，转载请注明出处】：[https://panjinquan.blog.csdn.net/article/details/120601437](https://panjinquan.blog.csdn.net/article/details/120601437 "https&#x3A;//panjinquan.blog.csdn.net/article/details/120601437")

* * *

**目录**

[1. 项目结构](#1.%20%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84)

[2. 环境配置](#2.%20%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE)

[3. 音频识别基础知识](#4.%E9%9F%B3%E9%A2%91%E8%AF%86%E5%88%AB%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)

[(1) STFT 和声谱图 (spectrogram)    ](#%281%29%20STFT%E5%92%8C%E5%A3%B0%E8%B0%B1%E5%9B%BE%28spectrogram%29%20%C2%A0%20%C2%A0)

[(2) 梅尔频谱](#%282%29%C2%A0%E6%A2%85%E5%B0%94%E9%A2%91%E8%B0%B1)

[(3) 梅尔频率倒谱 MFCC](#%283%29%C2%A0%E6%A2%85%E5%B0%94%E9%A2%91%E7%8E%87%E5%80%92%E8%B0%B1)

[(4) MFCC 特征的过程](#%284%29%20MFCC%E7%89%B9%E5%BE%81%E7%9A%84%E8%BF%87%E7%A8%8B)

[4. 数据处理](#3.%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%EF%BC%881%EF%BC%89%E6%95%B0%E6%8D%AE%E9%9B%86Urbansound8K%C2%A0)

[（1）数据集 Urbansound8K ](#%EF%BC%881%EF%BC%89%E6%95%B0%E6%8D%AE%E9%9B%86Urbansound8K%C2%A0)

[（2）自定义数据集](#%EF%BC%882%EF%BC%89%E8%87%AA%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E9%9B%86)

[（3）音频特征提取: ](#%EF%BC%883%EF%BC%89%E9%9F%B3%E9%A2%91%E7%89%B9%E5%BE%81%E6%8F%90%E5%8F%96%3A%C2%A0)

[5. 训练 Pipeline](#4.%E8%AE%AD%E7%BB%83Pipeline)

[6. 预测 demo.py](#5.%E9%A2%84%E6%B5%8Bdemo.py)

* * *

### 1. 项目结构

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/cf615e4f-80d9-4265-b69d-075a4184074d.png?raw=true)

* * *

### 2. 环境配置

```
pytorch==1.7.1,其他依赖库，请使用pip命令安装libsora和pyaudio,pydub等库

```

```null
librosa==0.8.1pyaudio==0.2.11pydub==0.23.1
```

 项目安装教程请参考（初学者入门，麻烦先看完下面教程，配置好开发环境）：

> -   [项目开发使用教程和常见问题和解决方法](https://blog.csdn.net/guyuealian/article/details/129163343 "项目开发使用教程和常见问题和解决方法")
> -   **视频教程：** [3 如何用 Anaconda 创建 pycharm 环境](https://www.bilibili.com/video/BV18Y411c7Qt/?spm_id_from=333.999.0.0 "3 如何用 Anaconda 创建 pycharm 环境")
> -   **视频教程：** [4 如何在 pycharm 中使用 Anaconda 创建的 python 环境](https://www.bilibili.com/video/BV1SY4y1z7eS/?spm_id_from=333.999.0.0 "4 如何在 pycharm 中使用 Anaconda 创建的 python 环境")
> -   **项目不要出现含有中文字符的目录文件或路径，否则会出现很多异常**！！！！！！！！

* * *

### 3. 音频识别基础知识

#### **(1) STFT 和声谱图 (spectrogram)**

声音信号是一维信号，直观上只能看到时域信息，不能看到频域信息。通过傅里叶变换 (FT) 可以变换到频域，但是丢失了时域信息，无法看到时频关系。为了解决这个问题，产生了很多方法，短时傅里叶变换，小波等都是很常用的时频分析方法。  

短时傅里叶变换 ([STFT](https://so.csdn.net/so/search?q=STFT&spm=1001.2101.3001.7020))，就是对短时的信号做傅里叶变换。原理如下：对一段长语音信号，分帧、加窗，再对每一帧做傅里叶变换，之后把每一帧的结果沿另一维度堆叠，得到一张图（类似于二维信号），这张图就是声谱图。

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/8c99d461-7905-4009-b0e2-0fd88d4d05a8.png?raw=true)

#### **(2) 梅尔频谱**

人耳能听到的频率范围是 20-20000HZ，但是人耳对 HZ 单位不是线性敏感，而是对低 HZ 敏感，对高 HZ 不敏感，将 HZ 频率转化为梅尔频率，则人耳对频率的感知度就变为线性。    

例如如果我们适应了 1000Hz 的音调，如果把音调频率提高到 2000Hz，我们的耳朵只能觉察到频率提高了一点点，根本察觉不到频率提高了一倍 。

将普通频率转化到 Mel 频率的公式是

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/d284c7da-eb11-4f2a-946b-4cd46aeb19cf.png?raw=true)

> 在 Mel 频域内，人对音调的感知度为线性关系。举例来说，如果两段语音的 Mel 频率相差两倍，则人耳听起来两者的音调也相差两倍。

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/06896f60-a779-4bca-8c5d-54c11e35b25e.png?raw=true)

上图是 HZ 到 Mel 的映射关系图，由于二者为 log 关系，在频率较低时，Mel 随 HZ 变化较快；当频率较高时，曲线斜率小，变化缓慢。

#### **(3) 梅尔频率倒谱**MFCC

: 梅尔频率倒谱系数，简称 MFCC，Mel-frequency cepstral coefficients)

-   梅尔频谱就是一般的频谱图加上梅尔滤波函数，这一步是为了模拟人耳听觉对实际频率的敏感程度
-   梅尔倒谱就是再对梅尔频谱进行一次频谱分析，具体就是对梅尔频谱取对数，然后做 DCT 变换，目的是抽取频谱图的轮廓信息，这个比较能代表语音的特征。
-   如果取低频的 13 位，就是最经典的语音特征 mfcc 了

#### **(4) MFCC 特征的过程**

-   先对语音进行预加重、分帧和加窗；
-   对每一个短时分析窗，通过 FFT 得到对应的频谱；
-   将上面的频谱通过 Mel 滤波器组得到 Mel 频谱；
-   在 Mel 频谱上面进行倒谱分析（取对数，做逆变换，实际逆变换一般是通过 DCT 离散余弦变换来实现，
-   取 DCT 后的第 2 个到第 13 个系数作为 MFCC 系数），获得 Mel 频率倒谱系数 MFCC，这个 MFCC 就是这帧语音的特征了

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/f33cf900-5abe-484e-8447-685d96e8f999.png?raw=true)

* * *

### 4. 数据处理

#### 

（1）数据集 Urbansound8K 

 Urbansound8K 是目前应用较为广泛的用于自动城市环境声分类研究的公共数据集，  
包含 10 个分类：**空调声、汽车鸣笛声、儿童玩耍声、狗叫声、钻孔声、引擎空转声、枪声、手提钻、警笛声和街道音乐声, 类别定义如下：** 

**data/UrbanSound8K/class_name.txt**

> ```
> air_conditioner
> car_horn
> children_playing
> dog_bark
> drilling
> engine_idling
> gun_shot
> jackhammer
> siren
> street_music
>
> ```

**特别特别说明：** UrbanSound8K 一共有 10 个文件夹，相同文件夹并非是相同类别，请不要误以为同一个文件夹是同一个类别的样本数据！！！官方提供 metadata/UrbanSound8.csv 文件，标注了每个样本的类别，如果你不想自己解析，请直接复用我的 train.txt 和 test.txt。

**数据集下载**：[https://zenodo.org/record/1203745/files/UrbanSound8K.tar.gz](https://zenodo.org/record/1203745/files/UrbanSound8K.tar.gz "https&#x3A;//zenodo.org/record/1203745/files/UrbanSound8K.tar.gz")

#### （2）自定义数据集

可以自己录制音频信号，制作自己的数据集，参考\[audio/dataloader/record_audio.py]  
每个文件夹存放一个类别的音频数据，每条音频数据长度在 3 秒左右, 建议每类的音频数据均衡  
生产 train 和 test 数据列表：参考\[audio/dataloader/create_data.py],train.txt 和 test.txt 格式如下：

> \[path/to/audio.wav,labels_name]

如: 

```null
ID1/audio.wav,label_names1ID2/audio.wav,label_names2...IDn/audio.wav,label_namesn
```

同时，你需要定义 class_name，指定类别的序列，训练代码会根据你的 class_name 进行训练：

> ```
> label_names1
> label_names2
> ...
> label_namesn
>
> ```

#### （3）音频特征提取: 

音频信号是一维的语音信号，不能直接用于模型训练，需要使用 librosa 将音频转为**梅尔频谱（Mel Spectrogram）**。

librosa 提供 python 接口，在音频、乐音信号的分析中经常用到

```null
wav, sr = librosa.load(data_path, sr=16000)# 使用librosa获得音频的梅尔频谱spec_image = librosa.feature.melspectrogram(y=wav, sr=sr, hop_length=256)# 计算音频信号的MFCCspec_image = librosa.feature.mfcc(y=wav, sr=sr)
```

librosa.feature.mfcc 实现 MFCC 源码如下：

```null
# -- Mel spectrogram and MFCCs -- #def mfcc(y=None, sr=22050, S=None, n_mfcc=20, dct_type=2, norm="ortho", lifter=0, **kwargs):    """Mel-frequency cepstral coefficients (MFCCs)    """    if S is None:        # 先调用melspectrogram，计算梅尔频谱，然后取对数:power_to_db        S = power_to_db(melspectrogram(y=y, sr=sr, **kwargs))    # 最后进行DCT变换    M = scipy.fftpack.dct(S, axis=0, type=dct_type, norm=norm)[:n_mfcc]    if lifter > 0:        M *= (1 + (lifter / 2) * np.sin(np.pi * np.arange(1, 1 + n_mfcc, dtype=M.dtype) / lifter)[:, np.newaxis])        return M    elif lifter == 0:        return M    else:        raise ParameterError("MFCC lifter={} must be a non-negative number".format(lifter))
```

> 可以看到，librosa 库中，梅尔倒谱就是再对梅尔频谱取对数，然后做 DCT 变换

关于 librosa 的使用方法，请参考：

1.   [音频特征提取——librosa 工具包使用 ](https://www.cnblogs.com/xingshansi/p/6816308.html "音频特征提取——librosa 工具包使用 ")
2.   [梅尔频谱 (mel spectrogram) 原理与使用 ](https://zhuanlan.zhihu.com/p/351956040 "梅尔频谱 (mel spectrogram) 原理与使用 ")

### 5. 训练 Pipeline

（1）构建训练和测试数据

```null
    def build_dataset(self, cfg):        """构建训练数据和测试数据"""        input_shape = eval(cfg.input_shape)        # 获取数据        train_dataset = AudioDataset(cfg.train_data, data_dir=cfg.data_dir, mode='train', spec_len=input_shape[3])        train_loader = DataLoader(dataset=train_dataset, batch_size=cfg.batch_size, shuffle=True,                                  num_workers=cfg.num_workers)         test_dataset = AudioDataset(cfg.test_data, data_dir=cfg.data_dir, mode='test', spec_len=input_shape[3])        test_loader = DataLoader(dataset=test_dataset, batch_size=cfg.batch_size, shuffle=False,                                 num_workers=cfg.num_workers)        print("train nums:{}".format(len(train_dataset)))        print("test  nums:{}".format(len(test_dataset)))        return train_loader, test_loader
```

由于 librosa.load 加载音频数据特别慢，建议使用 cache 先进行缓存，方便加速

```null
def load_audio(audio_file, cache=False):    """    加载并预处理音频    :param audio_file:    :param cache: librosa.load加载音频数据特别慢，建议使用进行缓存进行加速    :return:    """    # 读取音频数据    cache_path = audio_file + ".pk"    # t = librosa.get_duration(filename=audio_file)    if cache and os.path.exists(cache_path):        tmp = open(cache_path, 'rb')        wav, sr = pickle.load(tmp)    else:        wav, sr = librosa.load(audio_file, sr=16000)        if cache:            f = open(cache_path, 'wb')            pickle.dump([wav, sr], f)            f.close()    # Compute a mel-scaled spectrogram: 梅尔频谱图    spec_image = librosa.feature.melspectrogram(y=wav, sr=sr, hop_length=256)    return spec_image
```

（2）构建 backbone 模型

backbone 是一个基于 CNN+FC 的网络结构，与图像 CNN 分类模型不同的是，图像 CNN 分类模型的输入维度 (batch,3,H,W) 输入数据 depth=3，而音频信号的**梅尔频谱图**是深度为 depth=1，可以认为是灰度图，输入维度 (batch,1,H,W)，因此实际使用中，只需要将传统的 CNN 图像分类的 backbone 的第一层卷积层的 in_channels=1 即可。需要注意的是，由于维度不一致，导致不能使用 imagenet 的 pretrained 模型。

当然可以将**梅尔频谱图 (灰度图)**是转为 3 通道 RGB 图，这样就跟普通的 RGB 图像没有什么区别了，也可以 imagenet 的 pretrained 模型，如

> ```
> \# 将**梅尔频谱图(灰度图)**是转为为3通道RGB图
> spec\_image = cv2.cvtColor(spec\_image, cv2.COLOR_GRAY2RGB)
>
> ```

```null
    def build_model(self, cfg):        if cfg.net_type == "mbv2":            model = mobilenet_v2.mobilenet_v2(num_classes=cfg.num_classes)        elif cfg.net_type == "resnet34":            model = resnet.resnet34(num_classes=args.num_classes)        elif cfg.net_type == "resnet18":            model = resnet.resnet18(num_classes=args.num_classes)        else:            raise Exception("Error:{}".format(cfg.net_type))        model.to(self.device)        return model
```

（3）训练参数配置

相关的命令行参数，可参考：

```null
 def get_parser():    data_dir = "/home/dataset/UrbanSound8K/audio"    train_data = 'data/UrbanSound8K/train.txt'    test_data = 'data/UrbanSound8K/test.txt'    class_name = 'data/UrbanSound8K/class_name.txt'    parser = argparse.ArgumentParser(description=__doc__)    parser.add_argument('--batch_size', type=int, default=32, help='训练的批量大小')    parser.add_argument('--num_workers', type=int, default=8, help='读取数据的线程数量')    parser.add_argument('--num_epoch', type=int, default=100, help='训练的轮数')    parser.add_argument('--class_name', type=str, default=class_name, help='类别文件')    parser.add_argument('--learning_rate', type=float, default=1e-3, help='初始学习率的大小')    parser.add_argument('--input_shape', type=str, default='(None, 1, 128, 128)', help='数据输入的形状')    parser.add_argument('--gpu_id', type=int, default=0, help='GPU ID')    parser.add_argument('--net_type', type=str, default="mbv2", help='backbone')    parser.add_argument('--data_dir', type=str, default=data_dir, help='数据路径')    parser.add_argument('--train_data', type=str, default=train_data, help='训练数据的数据列表路径')    parser.add_argument('--test_data', type=str, default=test_data, help='测试数据的数据列表路径')    parser.add_argument('--work_dir', type=str, default='work_space/', help='模型保存的路径')    return parser
```

配置好数据路径，其他参数默认设置，即可以开始训练了：

```null
# data_dir是你的数据路径python train.py --data_dir="dataset/UrbanSound8K/audio1"
```

![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/ccf11c7c-4863-40db-aae8-c2b53aaaf5c8.png?raw=true)

训练完成，使用 mobilenet_v2, 最终训练集准确率 99% 左右，测试集 96% 左右，看起来有点过拟合了。如果想进一步提高识别准确率可以使用更重的 backbone，如 resnet34, 采用更多的数据增强方法，提高模型的泛发性。

训练过程可视化工具是使用 Tensorboard，使用方法：

```null
# 安装：pip install tensorboard  tensorboardX # 基本方法tensorboard --logdir=path/to/log/# 例如tensorboard --logdir=work_space/
```

\| ![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/17775632-3078-495c-8c41-019f40b6d5af.png?raw=true)
 \| ![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-12-23%2009-43-49/792c8d40-d2e2-4fc0-b943-e7e23aa79f63.png?raw=true)
 \|

完整的训练代码 train.py：

```null
import argparseimport osimport numpy as npimport torchimport tensorboardX as tensorboardfrom datetime import datetimefrom easydict import EasyDictfrom tqdm import tqdmfrom torch.utils.data import DataLoaderfrom torch.optim.lr_scheduler import StepLR, MultiStepLRfrom audio.dataloader.audio_dataset import AudioDatasetfrom audio.utils.utility import print_argumentsfrom audio.utils import file_utilsfrom audio.models import mobilenet_v2, resnet  class Train(object):    """Training  Pipeline"""     def __init__(self, cfg):        cfg = EasyDict(cfg.__dict__)        self.device = "cuda:{}".format(cfg.gpu_id) if torch.cuda.is_available() else "cpu"        self.num_epoch = cfg.num_epoch        self.net_type = cfg.net_type        self.work_dir = os.path.join(cfg.work_dir, self.net_type)        self.model_dir = os.path.join(self.work_dir, "model")        self.log_dir = os.path.join(self.work_dir, "log")        file_utils.create_dir(self.model_dir)        file_utils.create_dir(self.log_dir)         self.tensorboard = tensorboard.SummaryWriter(self.log_dir)        self.train_loader, self.test_loader = self.build_dataset(cfg)        # 获取模型        self.model = self.build_model(cfg)        # 获取优化方法        self.optimizer = torch.optim.Adam(params=self.model.parameters(),                                          lr=cfg.learning_rate,                                          weight_decay=5e-4)        # 获取学习率衰减函数        self.scheduler = MultiStepLR(self.optimizer, milestones=[50, 80], gamma=0.1)        # 获取损失函数        self.losses = torch.nn.CrossEntropyLoss()     def build_dataset(self, cfg):        """构建训练数据和测试数据"""        input_shape = eval(cfg.input_shape)        # 加载训练数据        train_dataset = AudioDataset(cfg.train_data,                                     class_name=cfg.class_name,                                     data_dir=cfg.data_dir,                                     mode='train',                                     spec_len=input_shape[3])        train_loader = DataLoader(dataset=train_dataset, batch_size=cfg.batch_size, shuffle=True,                                  num_workers=cfg.num_workers)        cfg.class_name = train_dataset.class_name        cfg.class_dict = train_dataset.class_dict        cfg.num_classes = len(cfg.class_name)        # 加载测试数据        test_dataset = AudioDataset(cfg.test_data,                                    class_name=cfg.class_name,                                    data_dir=cfg.data_dir,                                    mode='test',                                    spec_len=input_shape[3])        test_loader = DataLoader(dataset=test_dataset, batch_size=cfg.batch_size, shuffle=False,                                 num_workers=cfg.num_workers)        print("train nums:{}".format(len(train_dataset)))        print("test  nums:{}".format(len(test_dataset)))        return train_loader, test_loader     def build_model(self, cfg):        """构建模型"""        if cfg.net_type == "mbv2":            model = mobilenet_v2.mobilenet_v2(num_classes=cfg.num_classes)        elif cfg.net_type == "resnet34":            model = resnet.resnet34(num_classes=args.num_classes)        elif cfg.net_type == "resnet18":            model = resnet.resnet18(num_classes=args.num_classes)        else:            raise Exception("Error:{}".format(cfg.net_type))        model.to(self.device)        return model     def epoch_test(self, epoch):        """模型测试"""        loss_sum = []        accuracies = []        self.model.eval()        with torch.no_grad():            for step, (inputs, labels) in enumerate(tqdm(self.test_loader)):                inputs = inputs.to(self.device)                labels = labels.to(self.device).long()                output = self.model(inputs)                # 计算损失值                loss = self.losses(output, labels)                # 计算准确率                output = torch.nn.functional.softmax(output, dim=1)                output = output.data.cpu().numpy()                output = np.argmax(output, axis=1)                labels = labels.data.cpu().numpy()                acc = np.mean((output == labels).astype(int))                accuracies.append(acc)                loss_sum.append(loss)        acc = sum(accuracies) / len(accuracies)        loss = sum(loss_sum) / len(loss_sum)        print("Test epoch:{:3.3f},Acc:{:3.3f},loss:{:3.3f}".format(epoch, acc, loss))        print('=' * 70)        return acc, loss     def epoch_train(self, epoch):        """模型训练"""        loss_sum = []        accuracies = []        self.model.train()        for step, (inputs, labels) in enumerate(tqdm(self.train_loader)):            inputs = inputs.to(self.device)            labels = labels.to(self.device).long()            output = self.model(inputs)            # 计算损失值            loss = self.losses(output, labels)            self.optimizer.zero_grad()            loss.backward()            self.optimizer.step()             # 计算准确率            output = torch.nn.functional.softmax(output, dim=1)            output = output.data.cpu().numpy()            output = np.argmax(output, axis=1)            labels = labels.data.cpu().numpy()            acc = np.mean((output == labels).astype(int))            accuracies.append(acc)            loss_sum.append(loss)            if step % 50 == 0:                lr = self.optimizer.state_dict()['param_groups'][0]['lr']                print('[%s] Train epoch %d, batch: %d/%d, loss: %f, accuracy: %f，lr:%f' % (                    datetime.now(), epoch, step, len(self.train_loader), sum(loss_sum) / len(loss_sum),                    sum(accuracies) / len(accuracies), lr))        acc = sum(accuracies) / len(accuracies)        loss = sum(loss_sum) / len(loss_sum)        print("Train epoch:{:3.3f},Acc:{:3.3f},loss:{:3.3f}".format(epoch, acc, loss))        print('=' * 70)        return acc, loss     def run(self):        # 开始训练        for epoch in range(self.num_epoch):            train_acc, train_loss = self.epoch_train(epoch)            test_acc, test_loss = self.epoch_test(epoch)            self.tensorboard.add_scalar("train_acc", train_acc, epoch)            self.tensorboard.add_scalar("train_loss", train_loss, epoch)            self.tensorboard.add_scalar("test_acc", test_acc, epoch)            self.tensorboard.add_scalar("test_loss", test_loss, epoch)            self.scheduler.step()            self.save_model(epoch, test_acc)     def save_model(self, epoch, acc):        """保持模型"""        model_path = os.path.join(self.model_dir, 'model_{:0=3d}_{:.3f}.pth'.format(epoch, acc))        if not os.path.exists(os.path.dirname(model_path)):            os.makedirs(os.path.dirname(model_path))        torch.jit.save(torch.jit.script(self.model), model_path)  def get_parser():    data_dir = "/home/dataset/UrbanSound8K/audio"    train_data = 'data/UrbanSound8K/train.txt'    test_data = 'data/UrbanSound8K/test.txt'    class_name = 'data/UrbanSound8K/class_name.txt'    parser = argparse.ArgumentParser(description=__doc__)    parser.add_argument('--batch_size', type=int, default=32, help='训练的批量大小')    parser.add_argument('--num_workers', type=int, default=8, help='读取数据的线程数量')    parser.add_argument('--num_epoch', type=int, default=100, help='训练的轮数')    parser.add_argument('--class_name', type=str, default=class_name, help='类别文件')    parser.add_argument('--learning_rate', type=float, default=1e-3, help='初始学习率的大小')    parser.add_argument('--input_shape', type=str, default='(None, 1, 128, 128)', help='数据输入的形状')    parser.add_argument('--gpu_id', type=int, default=0, help='GPU ID')    parser.add_argument('--net_type', type=str, default="mbv2", help='backbone')    parser.add_argument('--data_dir', type=str, default=data_dir, help='数据路径')    parser.add_argument('--train_data', type=str, default=train_data, help='训练数据的数据列表路径')    parser.add_argument('--test_data', type=str, default=test_data, help='测试数据的数据列表路径')    parser.add_argument('--work_dir', type=str, default='work_space/', help='模型保存的路径')    return parser  if __name__ == '__main__':    parser = get_parser()    args = parser.parse_args()    print_arguments(args)    t = Train(args)    t.run()
```

### 6. 预测 demo.py

 【完整的项目代码】：[基于梅尔频谱的音频信号分类识别 (Pytorch)](https://mp.weixin.qq.com/s/du1F3aeWYWl_2yKvumP1zg "基于梅尔频谱的音频信号分类识别 (Pytorch)")

```null
import osimport cv2import argparseimport librosaimport torchimport numpy as npfrom audio.dataloader.audio_dataset import load_audio, normalizationfrom audio.dataloader.record_audio import record_audiofrom audio.utils import file_utils, image_utils  class Predictor(object):    def __init__(self, cfg):        # self.device = "cuda:{}".format(cfg.gpu_id) if torch.cuda.is_available() else "cpu"        self.device = "cpu"        self.class_name, self.class_dict = file_utils.parser_classes(cfg.class_name, split=None)        self.input_shape = eval(cfg.input_shape)        self.spec_len = self.input_shape[3]        self.model = self.build_model(cfg.model_file)     def build_model(self, model_file):        # 加载模型        model = torch.jit.load(model_file, map_location="cpu")        model.to(self.device)        model.eval()        return model     def inference(self, input_tensors):        with torch.no_grad():            input_tensors = input_tensors.to(self.device)            output = self.model(input_tensors)        return output     def pre_process(self, spec_image):        """音频数据预处理"""        if spec_image.shape[1] > self.spec_len:            input = spec_image[:, 0:self.spec_len]        else:            input = np.zeros(shape=(self.spec_len, self.spec_len), dtype=np.float32)            input[:, 0:spec_image.shape[1]] = spec_image        input = normalization(input)        input = input[np.newaxis, np.newaxis, :]        input_tensors = np.concatenate([input])        input_tensors = torch.tensor(input_tensors, dtype=torch.float32)        return input_tensors     def post_process(self, output):        """输出结果后处理"""        scores = torch.nn.functional.softmax(output, dim=1)        scores = scores.data.cpu().numpy()        # 显示图片并输出结果最大的label        label = np.argmax(scores, axis=1)        score = scores[:, label]        label = [self.class_name[l] for l in label]        return label, score     def detect(self, audio_file):        """        :param audio_file: 音频文件        :return: label:预测音频的label                 score: 预测音频的置信度        """        spec_image = load_audio(audio_file)        input_tensors = self.pre_process(spec_image)        # 执行预测        output = self.inference(input_tensors)        label, score = self.post_process(output)        return label, score     def detect_file_dir(self, file_dir):        """        :param file_dir: 音频文件目录        :return:        """        file_list = file_utils.get_files_lists(file_dir, postfix=["*.wav"])        for file in file_list:            print(file)            label, score = self.detect(file)            print("pred-label:{}, score:{}".format(label, score))            print("---" * 20)     def detect_record_audio(self, audio_dir):        """        :param audio_dir: 录制音频并进行识别        :return:        """        time = file_utils.get_time()        file = os.path.join(audio_dir, time + ".wav")        record_audio(file)        label, score = self.detect(file)        print(file)        print("pred-label:{}, score:{}".format(label, score))        print("---"*20)   def get_parser():    model_file = 'data/pretrained/model_075_0.965.pth'    file_dir = "data/audio"    class_name = 'data/UrbanSound8K/class_name.txt'    parser = argparse.ArgumentParser(description=__doc__)    parser.add_argument('--class_name', type=str, default=class_name, help='类别文件')    parser.add_argument('--input_shape', type=str, default='(None, 1, 128, 128)', help='数据输入的形状')    parser.add_argument('--net_type', type=str, default="mbv2", help='backbone')    parser.add_argument('--gpu_id', type=int, default=0, help='GPU ID')    parser.add_argument('--model_file', type=str, default=model_file, help='模型文件')    parser.add_argument('--file_dir', type=str, default=file_dir, help='音频文件的目录')    return parser  if __name__ == '__main__':    parser = get_parser()    args = parser.parse_args()    p = Predictor(args)    p.detect_file_dir(file_dir=args.file_dir)    # audio_dir = 'data/record_audio'    # p.detect_record_audio(audio_dir=audio_dir)
```

* * *

### 7. 源码下载

 【完整的项目代码】：[基于梅尔频谱的音频信号分类识别 (Pytorch)](https://mp.weixin.qq.com/s/du1F3aeWYWl_2yKvumP1zg "基于梅尔频谱的音频信号分类识别 (Pytorch)")

* * *
