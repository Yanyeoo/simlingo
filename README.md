# SimLingo 项目阶段性总结

## 一、项目背景与目标

在自动驾驶端到端范式不断演进的背景下，VLA（Vision-Language-Action）模型逐渐成为连接感知、理解与决策的关键技术路线。SimLingo（CarLLava）作为 CVPR 2025 Spotlight 工作，提出了一种 **Vision-Only + Language-Action Alignment** 的闭环自动驾驶框架，在无需高精地图与显式规划模块的前提下，实现了可解释、高层决策驱动的自动驾驶行为。

本项目的核心目标是：

* **完整复现 SimLingo 的闭环推理流程（Closed-loop Inference）**
* 在 CARLA 0.9.15 仿真环境中实现 **可视化运行**
* 为后续研究 **高效 VLA / 推理加速 / 模型裁剪** 打下工程与系统基础

> 项目定位：以“跑通 + 理解 + 可扩展”为第一阶段目标，而非从零训练模型。

---

## 二、实验环境与硬件条件

### 1. 硬件资源

* GPU：NVIDIA RTX 4090 × 1
* 显存：24GB
* 磁盘空间：≈100GB
* 运行方式：远程服务器（SSH + VSCode）

### 2. 软件与系统

* 操作系统：CentOS（服务器端）
* Python 环境：conda / venv（simlingo 专用环境）
* CUDA / Driver：支持 RTX 4090
* 仿真平台：CARLA 0.9.15
* 可视化：NoMachine + 虚拟/物理显示器 + Vulkan

---

## 三、项目实施过程

### 3.1 SimLingo 代码与模型准备

* 目标仓库：`RenzKa/simlingo`
* 模型体量较大，依赖 **HuggingFace + Git LFS**

#### 遇到的问题

* Git LFS 在服务器环境下多次下载失败
* 网络不稳定导致模型权重损坏

#### 解决方案

* 使用 HuggingFace 官方脚本 `hfd.sh`
* 结合 `aria2c` 进行多线程断点续传
* 成功下载并校验模型权重（Inference-only）

📌 **结论**：在国内/教育网环境下，`hfd.sh + aria2c` 是比 Git LFS 更稳妥的方案。

---

### 3.2 CARLA 仿真环境搭建

#### 关键目标

* 使用 **GPU 渲染（Vulkan）** 而非 CPU OpenGL
* 支持 **可视化窗口 + SimLingo RPC 连接**

#### 实际挑战

1. 默认无头（headless）模式 → OpenGL CPU 渲染
2. Xvfb 虚拟屏幕 ≠ 真正 GPU 渲染
3. Vulkan 驱动 / DISPLAY / NoMachine 之间关系复杂

#### 关键技术点

* 使用 NoMachine 创建虚拟桌面（如 `:1001`）
* CARLA 使用 Vulkan 后端绑定 GPU
* 显式指定 DISPLAY，避免 Xvfb
* VirtualGL 路径错误导致 `vglrun` 失败

#### 最终方案

* CARLA 在 NoMachine 虚拟屏幕上 **成功使用 GPU Vulkan 渲染**
* SimLingo 可通过 `localhost:2000` 正常连接
* 实现 **可视化 + 闭环推理同时运行**

📌 **里程碑**：CARLA GPU Vulkan 启动成功，标志着系统级最大难点被突破。

---

### 3.3 SimLingo 闭环推理运行

* Bench2Drive 任务集加载成功
* XML route 能正常解析
* 推理流程：

  * CARLA → 视觉输入
  * Vision Encoder
  * Language-Action Alignment
  * 高层动作输出（变道 / 加减速等）

#### 已验证能力

* 模型权重加载完整
* 推理阶段可稳定运行（Inference Only）
* 与 CARLA 仿真形成闭环

---

## 四、问题汇总与解决记录

| 问题                             | 原因                  | 解决方式                   |
| ------------------------------ | ------------------- | ---------------------- |
| Git LFS 下载失败                   | 网络 / 权限             | hfd.sh + aria2c        |
| `ModuleNotFoundError: srunner` | PYTHONPATH 未设置      | 修正 CARLA Python API 路径 |
| CARLA 闪退                       | Vulkan / DISPLAY 错误 | 固定 DISPLAY + GPU 渲染    |
| OpenGL CPU 渲染                  | 无显示或 Xvfb           | NoMachine 虚拟屏幕         |
| vglrun 找不到                     | 路径不一致               | 使用实际解压路径               |

---

## 五、阶段性成果

1. ✅ 成功复现 **SimLingo 推理级别系统**
2. ✅ 打通 **CARLA – VLA 模型 – 决策输出 – 可视化** 全链路
3. ✅ 积累大量 **工程级 Debug 经验**（GPU / Vulkan / Display）
4. ✅ 对 SimLingo 的系统结构与设计理念形成清晰理解

这一步相当于：

> **从“读论文”正式走到“系统级复现”**

---

## 六、不足与当前限制

* 未进行模型训练与微调（仅 Inference）
* 对 Language-Action 对齐细节仍停留在代码层理解
* Bench2Drive 指标尚未系统评测
* 推理效率与显存占用仍有优化空间

---

## 七、下一步计划（与个人研究方向对齐）

结合本人后续 VLA / 高效模型研究方向，下一阶段计划：

1. **推理效率分析**

   * latency / FPS / 显存占用
   * 与非 VLA baseline 对比

2. **高效化方向探索**

   * LoRA / QLoRA 在 VLA 中的可行性
   * Vision Encoder 冻结 + 语言对齐层微调

3. **系统级改造**

   * 模块化 SimLingo 推理结构
   * 支持替换 Vision / LLM Backbone

4. **论文与汇报**

   * 作为后续 VLA baseline 对照实验
   * 纳入阶段性科研汇报 / 开题材料

---

## 八、总结性评价

SimLingo 项目的复现过程并非“代码一跑就完”，而是一次 **完整的系统工程训练**。从模型下载、环境配置，到 GPU 渲染、闭环联调，每一个问题都逼迫我真正理解：

> **大模型自动驾驶，不只是模型问题，更是系统问题。**

该阶段工作为后续自主设计高效 VLA 模型、进行真实科研创新奠定了坚实基础。
