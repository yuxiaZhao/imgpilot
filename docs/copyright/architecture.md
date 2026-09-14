# 技术架构

## 一、整体架构

imgpilot 采用扁平模块化设计：

```
src/
├── types.ts       # 公共类型定义（ImageDataLike, Position, 各 Options 接口）
├── utils.ts       # 工具函数（像素读写、数学工具、二分搜索）
├── adapter.ts     # 浏览器适配层（加载/编码/文本渲染）
├── pipeline.ts    # 链式 Pipeline（含撤销/重做历史）
├── crop.ts        # 裁剪算法
├── resize.ts      # 缩放算法（最近邻/双线性）
├── rotate.ts      # 旋转/翻转算法
├── filter.ts      # 滤镜算法（含高斯模糊可分离卷积）
├── watermark.ts   # 水印算法（Alpha 混合）
├── compress.ts    # 压缩策略（二分搜索质量参数）
├── convert.ts     # 格式转换
├── metadata.ts    # 元信息提取（感知亮度、透明通道检测）
├── exif.ts        # EXIF 解析（JPEG APP1 段 TIFF 结构）
├── zip.ts         # ZIP 打包（本地文件头 + 中央目录）/ CRC-32
└── index.ts       # 统一入口
```

## 二、核心层设计原则

### 2.1 纯函数
所有处理函数签名形如 `(image: ImageDataLike, options) => ImageDataLike`，无副作用、无运行时依赖。

### 2.2 编码器注入
压缩与格式转换需要"将 ImageData 编码为二进制"，通过 `Encoder` 接口注入：

```ts
interface Encoder {
  encode(data: ImageDataLike, mimeType: ImageMimeType, quality?: number): Promise<Blob>;
}
```

### 2.3 文本渲染器注入
文本水印需要将文字渲染为像素，通过 `TextRenderer` 接口注入：

```ts
interface TextRenderer {
  renderText(text: string, options: { font: string; color: string; rotate?: number }): ImageDataLike;
}
```

浏览器用 Canvas 2D `fillText` 实现文本水印渲染。

## 三、适配层实现

### 3.1 浏览器适配（src/adapter.ts）

- `loadImage(source)`：支持 File/Blob/string/ArrayBuffer，统一转为 HTMLImageElement
- `createCanvas(w, h)`：优先 OffscreenCanvas，回退 HTMLCanvasElement
- `getImageData(source)`：通过 Canvas 2D `getImageData` 提取像素
- `toBlob(canvas, mime, quality)`：优先 `OffscreenCanvas.convertToBlob`，回退 `HTMLCanvasElement.toBlob`
- `toDataURL(canvas, mime, quality)`：通过 Canvas.toDataURL 输出
- `renderText(...)`：用 Canvas 2D `measureText` + `fillText` 渲染文本为 ImageData

## 四、入口与导出策略

### 4.1 package.json exports

```json
{
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

## 五、构建配置

### 5.1 tsup（库构建）

- 1 个入口：index
- 双格式：ESM + CJS
- DTS 通过 tsc 生成

### 5.2 Vite（demo 构建）

- demo/ 目录独立 Vite 项目
- alias `imgpilot` → `src/index.ts`，便于源码调试

### 5.3 VitePress（文档构建）

- docs/ 目录
- 含指南、API两大大板块

## 六、依赖清单

### 开发依赖
- typescript / tsup / vite / vitepress / eslint / prettier

所有图像处理算法均为原创实现，未引入 sharp/jimp/lwip 等图像库。

## 七、启动与部署

### 8.1 环境要求

- Node.js 18+（构建环境，非运行时要求）
- 包管理器：npm（或 pnpm/yarn）

### 8.2 安装依赖

```bash
npm install
```

### 8.3 开发与调试

```bash
npm run dev             # 开发模式（tsup watch）
npm run dev:demo        # 启动 demo 站点（Vite，默认 https://imgpilot.pages.dev/）
npm run dev:docs        # 启动文档站点（VitePress）
```

### 8.4 构建产物

```bash
npm run build           # 构建库（tsup，产出 dist/ 含 ESM/CJS/DTS）
npm run build:demo      # 构建演示站点静态文件
npm run build:docs      # 构建文档站点静态文件
```

### 8.5 目录说明

| 目录 | 说明 |
|---|---|
| `src/` | 源代码（核心算法 + 浏览器适配 + Pipeline） |
| `demo/` | Vite 演示站点（软著截图来源） |
| `docs/` | VitePress 文档（含软著申请材料） |
| `dist/` | 构建产物（不入库，由 tsup 生成） |