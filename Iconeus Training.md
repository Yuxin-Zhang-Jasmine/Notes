---

# 超声功能成像（fUS）与ULM技术培训会记录

## highlight BackScatter Imaging

* 可清晰显示更细小的血管结构。


## 📍会议基本信息

* **主办单位**：Leeds Tech 生物
* **主讲人**：Prof. Huang
* **时间安排**：

  * **5月8日**：

    1. 产品介绍 ([Iconeus](https://iconeus.com/))
    2. 数据采集（使用 IcoScan）及初步处理（IcoStudio）
  * **5月9日**：

    1. 数据处理（IcoLAB / MATLAB）
    2. 成像演示：Tomography 和 ULM

---

## 🧪 Iconeus 介绍

### Doppler 原理简介

* **Pulse Doppler**：

  * 有时间窗，单个探头，包含深度信息；
  * 难以捕捉高速血流；
  * 受成像区域限制。
* **Continuous Doppler**
* **fUS 功能**：计算 Power Doppler 谱线下方的面积，与 **CBV（Cerebral Blood Volume）** 强度相关。

### 核心成像技术

* **Multi-Plane Wave (PW)**：高 SNR，捕捉高速血流（单个PW最大帧率10kHz）。
* **SVD（奇异值分解）**：相比高通滤波器更好，可保留低速血流信号。

### fUS 优势特性

* 相较其他模态（光学、PET、fMRI）：见PPT

  * 灵敏度（SNR?）比 fMRI（金标准）更高。
* 可用于清醒或轻度镇静动物，避免麻醉带来的神经反应抑制。
* **Head-fix** 相较 active 更稳定，有助于大脑功能区的 registration 与 test。
* 可与 EEG、光学、电生理等设备高度兼容。
* 探头种类丰富：

  * 不同尺寸、分辨率、视场、穿透深度；
  * 支持 3D 探头（multi-array ||||），采集速度比 2D 快 4 倍。
* 同时适用于脑功能和脑血流研究。
* 用户友好，培训 1\~2 次可独立操作；无特殊实验室环境要求。

### 系统与软件信息

* **Iconeus**（售价约60万元人民币）由巴黎团队开发，目前仅专注技术开发，销售与反馈由代理商处理。
* **2022年**首次量产，**2024年8月**清华对外开放测试。
* 软件为非定期更新，系统中存在轻微 bug。
* **三大软件组件**：

  * **IcoScan**：数据采集；
  * **IcoStudio**：快速预览与质量检查（不支持多数据集处理）；
  * **IcoLab**：用于后处理与批量分析。
* 用户多采用本地高配置工作站处理数据（采集在 Iconeus Machine，处理在 PC-WorkStation）。
* 当前软件VS升级前软件：计算速度大幅提升，浅层图像质量下降，深层区域有所提升（变明亮），猜测与 Attenuation Compensation 有关。

### 数据说明

* 原始数据默认帧率 500 frames/sec，占用内存巨大。
* 示例：3 分钟的 ULM 采样数据约 15.7 GB（时间分辨率 400ms）。
* 原始数据（用于计算 ULM 的微泡追踪）可保存，但不开放用户访问。
* 原始数据（用于 fUS）**不**可保存。存的数据是fUS 图像像素数据（power doppler），脑区分析通过区域内像素平均获得。

---

## 🧪 实验流程

### 实验准备

1. 处理小鼠头部（剃毛、清洁）。
2. 使用特殊胶水保持头皮湿润。
3. 使用齿科树脂固定头架（头架用于固定探头）。
4. 打磨多余树脂以免影响扫描。
5. 本次实验常用探头：

   * **IcoPrime**（128 元素, 15 MHz, 分辨率 100μm）；
   * **IcoPrime-4D MultiArray**（256 元素, 中心频率与分辨率同上）。

### 📷 成像操作（IcoScan）

* 选择探头 → 调整探头位置（x, y, z, Rotation）。
* 推荐设置：

  * **Temporal resolution**：400 ms/PW （若选 200ms 或 100ms，原始数据不够长，心动噪音去不干净，SNR会偏低）；
    * 400ms中的200ms记了一段时间的原始数据（能够有两个完整的心动周期，，便于把鼠心跳噪音去掉）；
    * 剩下200ms用来处理原始数据，滤波之类。
  * **Averaging**：4 (compounding 4 PWs to give one (Power Doppler) Image)；
* 3D 成像模式：

  * **Fast Scan**：速度快，由 2D 切片组成, 但另一个方向分辨率不太OK；
  * **Tomography**：立体图像质量好，各角度分辨率都不错，但需更长扫描时间（线阵需 40 分钟，multi-array 约 10 分钟）。
* 2D ULM （用 2D 不用 3D， 因为数据量太大）

  * If uses the 4D probe, make sure tuning the probe-position parameters to let it be stable (让探头别动)
  * 微泡注射通常在小鼠**轻度**麻醉后进行。若采集时间长，3分钟补一次注射。
* 设置好相关参数后，在 Live View 界面下方点击播放按钮开始实时 CBV 成像与数据采集。



### 🧭 数据浏览与检查（IcoStudio）

* 可视化模式：Three View （三视图） / Collage View（不同时间点的单视图拼接） / 可转动的 3D 图。
* 注册与导航：

  * 在 Brain registration 可配准fMRI的 脑区图(show atlas) 和 血管reference(需导入)图, 支持 自/手动/在图上用箭头拖放配
  * 可手动画区，保存重复使用；
  * 在 Brain navigation 可手动画两点，导航探头位置。
* 可画脑区激活/抑制的红蓝图 （RED parts have CBV higher than the baseline while BLUE parts have CBV lower than the baseline）：
  * 在Relative CBV 选项卡
  * 设置时间窗、baseline、空间与时间平均方式：
  * time btw. (between) slices  = 0.2 s  : The time left for probe motion from 1 slice to the next 如果太短，相当于还没等probe停下就扫描，图像会花
  * 在Relative CBV 选项卡， 指定baseline 状态的起止时间，在baseline时间以外刺激动物
  * 设置 spatial(pixel) averaging + Temporal(number of frames) averaging
* 可看脑区功能连接

  * 在 Functional connectivity 选项卡
  * 一般用低频信号 勾选 low pass filtering 0.1 Hz, 
  * 勾选 Base line Correction
  * 勾选（optional）Remove global Signal （recommanded for awake animals）
  * 实际一般用 IcoLab 另外处理数据，因为IcoStudio显示的分析图可读性差

### 📊 数据分析（IcoLab / MATLAB / Excel / ImageG？）

* （Prof. Huang 他们可以提供相关MATLAB脚本(P文件形式), Demo ， toolbox）
* 3min ULM 数据在IcoLab处理约需1小时，可得：

  * 血流密度Density图；
  * 血流速度Velocity图；
  * **backscatter 振幅图（适合观测细血管）**。
* 功能连接矩阵（基于 Pearson 相关，CBV 数据）。
* 脑区随时间变化图（heatmap: 横轴时间，纵轴脑区名，颜色 PowerDoppler）。
* Pixel-wise relative CBV 变化动图。
* **4D 探头基于体素的群组分析流程**：

  1. 手动配准，得到Transform矩阵；
  2. 把图像转化到标准空间（让不同的动物的数据都处在标准空间）；
  3. 平均多只动物的图像。

---

## 🧪 其它说明

### 小鼠实验建议

* < 9 周：可直接扫描颅骨；
* 10 周 - 3 月龄：颅骨迅速增厚；
* > 3 月：建议打磨或替换颅骨为薄膜；
* 个体差异显著，影响图像质量；
* 情绪反应对实验影响波动大，物理反应较稳定。

### 应用场景

* 疾病检测
* 药物测试
* 转基因小鼠研究
* 认知、睡眠、疼痛、听觉等研究
* 脊髓、三叉神经节研究
* 脑机接口（非侵入式， traditional way is vasive, 植入sth.）
* 脏器ULM成像（肾脏、肝脏，body pressed and fixed, limit the scan erea in a board hole）
* 潜在临床应用：

  * 开颅术辅助；
  * 新生儿脑癫痫 功能连接；
  * 经颅脑血流ULM成像（在近耳区域照射）。
