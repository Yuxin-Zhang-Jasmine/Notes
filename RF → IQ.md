# 📘 RF → IQ 变换与复包络提取笔记

## 🧠 背景与目标

在超声、通信等系统中，我们经常需要将实数形式的射频信号（RF）转为复数形式的基带信号（IQ），以提取出包络和相位，进行后续处理。这个过程叫作：

> **下变频 + 低通滤波 → IQ 提取**

得到的复信号也称为：**复包络（Complex Envelope）**。

---

## 🧩 基本形式与目标

设实数 RF 信号为：

$$
s_\text{RF}(t) = a(t) \cdot \cos(2\pi f_c t + \phi(t))
$$

其中：

* $a(t)$：慢变的**包络**（Envelope）
* $\phi(t)$：慢变的**瞬时相位**
* $f_c$：载频（carrier frequency）

目标是提取出：

$$
s_\text{IQ}(t) = a(t) \cdot e^{j\phi(t)}
$$

这是原始信号的**复包络（Complex Envelope）**，含有**幅度信息**（包络）与**相位信息**，用于后续信号处理。

---

## 📌 名词解释：复包络、复本振、实本振

### 🔹 复包络（Complex Envelope）

复包络是信号从载频 $f_c$ 下变频至零频（基带）后，得到的**复数形式的调制信号**：

$$
s_\text{IQ}(t) = a(t)e^{j\phi(t)}
$$

它的模长 $|s_\text{IQ}(t)| = a(t)$，就是我们熟悉的“包络”；它的辐角 $\arg(s_\text{IQ}(t)) = \phi(t)$，表示信号相位。

尽管名字叫“包络”，其实这个复数信号包含了**包络 + 相位**的全部信息。

---

### 🔹 复本振（Complex Local Oscillator）

使用一个复数形式的本振（local oscillator）进行混频：

$$
\text{LO}(t) = e^{-j2\pi f_c t}
$$

这种方法称为**复本振法**，直接将 RF 乘上复指数即可完成 IQ 分离。

---

### 🔹 实本振（Real Local Oscillator）

更早期的系统无法直接处理复数信号，因此使用两个实数本振进行分别乘法，得到 I/Q 分量：

* $I(t) = s_\text{RF}(t) \cdot \cos(2\pi f_c t)$
* $Q(t) = s_\text{RF}(t) \cdot (-\sin(2\pi f_c t))$

再通过低通滤波器分别滤除高频项，即可还原：

$$
I(t) + jQ(t) \approx \frac{1}{2} \cdot a(t) \cdot e^{j\phi(t)}
$$

---

## ✅ 方法一：复本振法（直接复乘）

直接将 RF 信号乘以复指数：

$$
s_\text{IQ}(t) = s_\text{RF}(t) \cdot e^{-j2\pi f_c t}
$$

推导如下：

$$
\begin{align*}
s_\text{IQ}(t) &= a(t)\cos(2\pi f_c t + \phi(t)) \cdot e^{-j2\pi f_c t} \\
&= \frac{a(t)}{2} \left[ e^{j(2\pi f_c t + \phi(t))} + e^{-j(2\pi f_c t + \phi(t))} \right] \cdot e^{-j2\pi f_c t} \\
&= \frac{a(t)}{2} \left[ e^{j\phi(t)} + e^{-j(4\pi f_c t + \phi(t))} \right]
\end{align*}
$$

其中：

* 第一项 $\propto a(t)e^{j\phi(t)}$：**目标基带信号**
* 第二项频率在 $2f_c$ 附近：**镜像分量**

🔻 **关键**：下变频后，仍需**低通滤波**去除第二项（高频镜像），保留基带成分。

---

## ✅ 方法二：实本振法（两路乘法）

构造两路实本振：

$$
\begin{cases}
I(t) = s_\text{RF}(t) \cdot \cos(2\pi f_c t) \\
Q(t) = s_\text{RF}(t) \cdot (-\sin(2\pi f_c t))
\end{cases}
$$

**同相分量（I）：**

$$
\begin{aligned}
I(t) &= s_{\mathrm{RF}}(t) \cdot \cos(2\pi f_c t) \\
     &= a(t) \cos(2\pi f_c t + \phi(t)) \cdot \cos(2\pi f_c t)
\end{aligned}
$$

使用积化和公式：

$$
\cos A \cos B = \frac{1}{2} \left[\cos(A-B) + \cos(A+B)\right]
$$

代入得：

$$
I(t) = \frac{a(t)}{2} \left[ \cos(\phi(t)) + \cos(4\pi f_c t + \phi(t)) \right]
$$

**正交分量（Q）：**

$$
\begin{aligned}
Q(t) &= s_{\mathrm{RF}}(t) \cdot (-\sin(2\pi f_c t)) \\
     &= -a(t) \cos(2\pi f_c t + \phi(t)) \cdot \sin(2\pi f_c t)
\end{aligned}
$$

使用积化和公式：

$$
\cos A \sin B = \frac{1}{2} \left[ \sin(A+B) + \sin(B-A) \right]
$$

代入得：

$$
Q(t) = \frac{a(t)}{2} \left[ \sin(\phi(t)) - \sin(4\pi f_c t + \phi(t)) \right]
$$


经过低通滤波：

$$
\begin{cases}
I(t) \to \frac{a(t)}{2} \cos(\phi(t)) \\
Q(t) \to \frac{a(t)}{2} \sin(\phi(t))
\end{cases}
$$

于是组成复信号：

$$
I(t) + jQ(t) = \frac{a(t)}{2} e^{j\phi(t)}
$$

再乘以 2，就得到目标复包络。

===
## 🎯 总结方法流程图

```
      实数 RF 信号
            ↓
   × e^{-j2πf_c t}（复本振）
            ↓
     含镜像的复信号
            ↓
     LPF（低通滤波）
            ↓
     复包络（IQ 信号）
```
<img width="976" height="598" alt="屏幕截图 2025-08-20 152511" src="https://github.com/user-attachments/assets/e2f5892a-46a8-4e6e-86b8-312e37914b7d" />
<img width="1085" height="787" alt="image" src="https://github.com/user-attachments/assets/00c89831-042b-4cb5-9194-c643587742ac" />
<img width="1131" height="743" alt="image" src="https://github.com/user-attachments/assets/f9b0269c-cb7e-4ca7-818b-4c2114ad57f9" />





---

这个版本是否满足你的需求？如果你希望我再加一张流程图或者统一格式，随时可以说。
