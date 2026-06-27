# Three.js 3x3 魔方单文件实现说明

## 项目简介

本项目目标是使用一个独立的 HTML 文件实现一个物理级高保真的 3x3 魔方交互演示。页面需基于 Three.js ES Modules 构建，支持自然手势拖拽、层旋转、视角旋转、打乱与重置功能。

该 README 用于说明需求来源、实现目标、技术约束和验收要点，便于开发者或 AI 编程助手按统一标准完成实现。

## 需求来源

- 来源链接：https://shuiyuan.sjtu.edu.cn/t/topic/485364/4
- 原始发布者：@尝试玩法真菌
- 内容类型：前端图形学与 Web 交互实现提示词

完整提示词已整理在同目录下的 `prompt-source.txt`。

## 技术要求

实现需满足以下约束：

- 单文件架构：HTML、CSS、JavaScript 全部合并在一个 HTML 文件中。
- 依赖引入：通过 `importmap` 从 unpkg 或 cdn.skypack 引入 Three.js、OrbitControls 与 Tween.js。
- 零素材依赖：不得加载外部图片或贴图，所有贴纸纹理需使用 HTML5 Canvas API 程序化生成。
- 模型组成：场景中包含 27 个独立 Cubies，并保留小方块之间的物理间隙。
- 光影效果：开启 ShadowMap，并配置 AmbientLight 与 DirectionalLight。
- 贴纸效果：使用 Canvas 绘制圆角矩形贴纸，模拟黑色塑料边框、颜色贴纸和高光质感。

## 核心实现要点

### 空间位置驱动的层筛选

旋转某一层时，不维护复杂的 3D 状态数组，而是遍历所有小方块，根据它们的世界坐标位置判断是否属于目标层。判断时应使用 Epsilon 阈值，避免浮点误差导致层识别失败。

### Pivot 挂载旋转机制

旋转层时创建临时 Pivot 对象：

1. 将目标层 Cubies 使用 `pivot.attach(object)` 挂载到 Pivot。
2. 对 Pivot 执行动画旋转。
3. 动画结束后使用 `scene.attach(object)` 将 Cubies 放回场景。
4. 清理 Pivot。

该方式依赖 Three.js 的世界矩阵计算，避免手动处理复杂四元数变换。

### 坐标清洗

每次旋转完成后，需要对所有 Cubies 的位置与旋转值做网格对齐，使用 `Math.round()` 或等价方式消除浮点累积误差，防止模型逐步偏移或散架。

### 自然手势识别

交互系统需区分：

- 左键拖拽：旋转魔方某一层。
- 右键拖拽：通过 OrbitControls 旋转视角。

左键拖拽时，应使用射线检测获取点击面法线，并根据法线确定潜在旋转轴。再将候选 3D 轴投影到 2D 屏幕空间，与鼠标拖拽向量做点积比较，选择匹配度最高的轴作为旋转方向。

实现需支持 1:1 实时跟手，并在松开鼠标后用 Tween.js 吸附到最近的 90 度倍数。

## 功能验收清单

- 页面为单个 HTML 文件。
- 不加载任何外部图片或贴图。
- 魔方由 27 个独立 Cubies 组成。
- Cubies 之间有可见但细微的物理间隙。
- 贴纸有圆角、黑边和高光质感。
- 场景有立体光照和阴影。
- 左键拖拽可旋转单层。
- 右键拖拽可旋转视角。
- 手势方向在正面、背面、顶面等视角下符合视觉直觉。
- 松手后能磁吸到最近 90 度。
- 提供 Scramble 和 Reset 按钮。
- 代码注释清楚解释手势投影算法与 Pivot 挂载逻辑。

## 建议文件结构

```text
rubiks-cube.html
README.md
prompt-source.txt
```

其中 `rubiks-cube.html` 为最终实现文件，`README.md` 与 `prompt-source.txt` 为说明文档。
