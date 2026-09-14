# imgpilot 软件说明书

> 本文档为 imgpilot 软件著作权申请材料之一，按软著申请要求编写。

## 一、软件基本信息

| 项 | 内容 |
|---|---|
| 软件名称 | imgpilot 纯前端图片处理工具库 |
| 软件简称 | imgpilot |
| 版本号 | V0.1.0 |
| 开发完成日期 | 2026 年 |
| 软件分类 | 图像处理软件 / 工具库 |
| 运行环境 | 现代浏览器（Chrome 90+ / Firefox 88+ / Safari 14+） |
| 编程语言 | TypeScript |
| 源代码量 | 约 60+ 页（详见源代码文件清单） |

## 二、软件功能概述

imgpilot 是一款纯前端图片处理工具库，提供 10 大功能模块：**压缩、水印、裁剪、缩放、旋转翻转、格式转换、滤镜、元信息提取、EXIF 解析、ZIP 打包**。软件提供两种调用风格：

1. **链式 API**：`imgpilot(src).resize().watermark().toBlob()` 风格，贴近自然语言，便于组合多步处理
2. **函数式 API**：每个模块独立导出纯函数（`compress / watermark / crop / resize / rotate / filter / convert / metadata / parseExif / createZip`），便于按需引用与组合

软件采用扁平模块化设计：核心算法为纯函数，不依赖任何运行时；适配层基于浏览器 Canvas 2D API。该设计使浏览器用户安装后零原生依赖。

## 三、技术特点

1. **纯前端定位**：核心算法基于 `ImageData`（`{ width, height, data: Uint8ClampedArray }`），可在浏览器、Worker 等任意 JavaScript 环境运行
2. **零原生依赖（浏览器）**：核心算法为纯函数，浏览器入口零原生依赖
3. **算法自实现**：不引入 sharp/jimp 等成熟图像库，所有处理算法（双线性采样、高斯模糊、Alpha 混合等）均自主实现
4. **类型严格**：TypeScript 严格模式，提供完整类型定义（.d.ts）
5. **多格式输出**：支持 JPEG / PNG / WebP 三种输出格式
6. **二分搜索压缩**：压缩模块支持目标体积上限，通过二分搜索质量参数实现精准体积控制

## 四、功能模块清单

| 模块 | 编程 API | 关键特性 |
|---|---|---|
| 压缩 | `compress()` | 质量压缩 + 目标体积二分搜索 |
| 水印 | `watermark()` | 文本/图片、九宫格位置、平铺、旋转 |
| 裁剪 | `crop()` | 区域裁剪 + 宽高比裁剪 |
| 缩放 | `resize()` | 最近邻/双线性、contain/cover |
| 旋转翻转 | `rotate()` / `flip()` | 任意角度 + 90° 整数倍高效路径 |
| 格式转换 | `convert()` | JPEG/PNG/WebP 互转 |
| 滤镜 | `filter()` | 灰度/棕褐/亮度/对比度/模糊/反相/色相/饱和度 |
| 元信息 | `metadata()` | 宽高/体积/平均亮度/透明通道检测 |
| EXIF 解析 | `parseExif()` | 方向/厂商/型号/拍摄时间/GPS |
| ZIP 打包 | `createZip()` / `crc32()` | 浏览器端 ZIP 打包，CRC-32 校验 |

## 五、运行环境

### 浏览器环境

- Chrome 90+ / Edge 90+
- Firefox 88+
- Safari 14+
- 需支持 Canvas 2D API 与 ImageData

## 六、安装方式

```bash
# 浏览器项目
npm install imgpilot
```

## 七、知识产权声明

本软件全部源代码由作者独立编写，未使用任何第三方图像处理算法库（如 sharp、jimp、lwip）。构建工具（tsup/vite/vitest/vitepress）与类型定义（@types/node）均为开发依赖，不进入运行时产物。所有图像处理算法均为原创实现。

## 八、文档导航

- [功能模块详解](./modules.md)
- [操作手册](./manual.md)
- [技术架构](./architecture.md)