# 功能模块详解

## 1. 压缩模块（compress）

### 功能
将图片编码为指定格式并控制体积：
- 按指定质量（quality）压缩
- 按目标体积上限（maxSize）压缩，自动二分搜索最优质量

### 算法
- 仅指定 quality：直接用该 quality 调用编码器
- 指定 maxSize：在 [0.1, quality] 范围内二分搜索 12 次，每次调用编码器测量实际体积，找最大满足 ≤ maxSize 的 quality
- PNG 格式且最低质量仍超限时，自动降级到 WebP

### 接口
```ts
async function compress(
  image: ImageDataLike,
  options: { quality?, maxSize?, mimeType? },
): Promise<{ blob, quality, size, mimeType }>
```

## 2. 水印模块（watermark）

### 功能
在图片上叠加文本或图片水印：
- 文本水印：支持字体、颜色、旋转
- 图片水印：支持任意 ImageData
- 九宫格位置定位（top-left ... bottom-right）
- 平铺模式：覆盖整张图
- 透明度与 Alpha 混合

### 算法
Alpha 混合采用标准源_OVER 算法：

```
outA = sa + da * (1 - sa)
outRGB = (srcRGB * sa + dstRGB * da * (1 - sa)) / outA
```

### 接口
```ts
function applyWatermarkImage(image, wmImage, options): ImageDataLike
function applyTextWatermark(image, text, options, renderer?): ImageDataLike
```

## 3. 裁剪模块（crop）

### 功能
- 区域裁剪：指定 x/y/width/height
- 宽高比裁剪：指定 aspectRatio，自动计算区域并支持九宫格对齐

### 算法
宽高比模式下，比较源图宽高比与目标宽高比：
- 源更宽：按高度裁，宽度 = height * aspectRatio
- 源更窄：按宽度裁，高度 = width / aspectRatio
- 根据 align 决定裁剪区域偏移

## 4. 缩放模块（resize）

### 功能
- 最近邻采样（速度快，适合像素艺术）
- 双线性采样（质量好，默认）
- 四种适配模式：exact / contain / cover / fill

### 算法
双线性采样：对每个目标像素，逆变换到源坐标，取相邻 4 像素加权平均：

```
top = lerp(p00, p10, tx)
bot = lerp(p01, p11, tx)
out = lerp(top, bot, ty)
```

## 5. 旋转翻转模块（rotate / flip）

### 功能
- 任意角度旋转（画布自动扩展，空白透明）
- 90° 整数倍旋转（高效路径，无重采样损失）
- 水平/垂直翻转

### 算法
任意角度旋转采用逆变换：对每个目标像素，反向旋转到源坐标，双线性采样源像素。新画布尺寸 = ceil(|w*cos| + |h*sin|) × ceil(|w*sin| + |h*cos|)。

### 90° 整数倍高效路径
- `rotate90(image, deg)`：按 90/180/270 度整倍数旋转，调用 `rotate90Once` 循环处理，无重采样损失。
- `rotate90Once(image)`：顺时针旋转 90°，通过坐标重映射直接搬运像素，O(w*h) 复杂度。

## 6. 格式转换模块（convert）

### 功能
在 JPEG / PNG / WebP 之间转换，支持质量参数。

### 实现
依赖适配层注入的 Encoder，核心层只负责调用契约。

### 工具函数
- `normalizeMime(raw)`：MIME 类型标准化，`image/jpg` 统一为 `image/jpeg`，未知格式兜底为 `image/png`。

## 7. 滤镜模块（filter）

### 功能
支持 9 种滤镜参数，可叠加：
- grayscale 灰度
- sepia 棕褐
- brightness 亮度
- contrast 对比度
- blur 高斯模糊
- invert 反相
- opacity 不透明度
- hueRotate 色相旋转
- saturate 饱和度

### 算法
- 高斯模糊：可分离卷积（先水平后垂直），O(n*radius) 复杂度
- 色相旋转：基于 YIQ 颜色空间变换
- 其余：逐像素线性运算

## 8. 元信息模块（metadata）

### 功能
同步返回图片元信息：
- width / height / pixels / channels
- size（ImageData 字节数）
- averageBrightness（感知亮度平均）
- hasAlpha（是否含透明通道）

### 算法
遍历所有像素，按 `0.299*R + 0.587*G + 0.114*B` 计算感知亮度并累加，同时检测 alpha < 255。

## 9. EXIF 解析模块（exif）

### 功能
从 JPEG 原始字节中解析 EXIF 元信息：
- orientation（方向标记 1-8）
- make / model（相机厂商/型号）
- dateTime（拍摄时间）
- gps（GPS 经纬度）

### 实现
基于 JPEG APP1 段解析 TIFF 结构，提取 IFD 及 EXIF IFD 子条目。非 JPEG 格式或无 EXIF 段时返回空对象。

## 10. ZIP 打包模块（zip）

### 功能
浏览器端打包为 ZIP 文件：
- `createZip(entries)`：将多个文件打包为 ZIP Blob
- `crc32(data)`：计算 CRC-32 校验值

### 算法
实现标准的 ZIP 本地文件头 + 中央目录结构，支持 Store（无压缩）模式。CRC-32 采用查表法实现。

## 11. 工具函数（utils）

### 数据校验
- `assertImageData(img, label?)`：校验 ImageDataLike 对象有效性，验证 width/height 属性存在且像素数据长度匹配 `width * height * 4`，不合法时抛出 TypeError 或 RangeError。