# 操作手册

## 一、编程 API 操作

### 1.1 浏览器链式调用

```ts
import { imgpilot, Position } from 'imgpilot';

// 1. 加载图片源
const pipeline = await imgpilot(fileInput.files[0]);

// 2. 链式处理
const result = await pipeline
  .resize({ width: 800, fit: 'contain', algorithm: 'bilinear' })
  .filter({ brightness: 0.1, saturate: 1.2 })
  .watermark({ text: '© 2026', position: Position.BottomRight, opacity: 0.8 })
  .compress({ maxSize: 100 * 1024, mimeType: 'image/jpeg' });

// 3. 使用结果
const blob = result.blob;
const url = URL.createObjectURL(blob);
imgPreview.src = url;
sizeLabel.textContent = `${(result.size / 1024).toFixed(2)} KB，质量 ${result.quality.toFixed(2)}`;
```

### 1.2 函数式调用

```ts
import { crop, filter, metadata, Position } from 'imgpilot';
import { loadImage, getImageData, toDataURL } from 'imgpilot';

const img = await loadImage(file);
let id = getImageData(img);

// 查看元信息
const info = metadata(id);
console.log(info.width, info.height, info.averageBrightness);

// 裁剪为 16:9
id = crop(id, { aspectRatio: 16 / 9, align: Position.Center });

// 应用灰度滤镜
id = filter(id, { grayscale: 1 });

// 输出
const url = toDataURL({ data: id.data, width: id.width, height: id.height }, 'image/png');
```

## 二、Demo 站点操作

1. 启动 demo：`npm run dev:demo`
2. 浏览器打开 `http://localhost:3000`
  也可直接访问在线站点：[imgpilot.pages.dev](https://imgpilot.pages.dev/)
3. 点击或拖拽图片到左侧"选择图片"区域（单次最多 10 张）
4. 在右侧切换 Tab 选择处理类型，调整参数
5. 勾选"启用此步骤"复选框激活该处理步骤
6. 点击"执行处理"按钮
7. 结果区显示处理后的图片与元信息（含步骤链、尺寸、文件大小、平均亮度等）
8. 点击"下载结果"保存当前图片
9. 多图场景支持"下载全部"打包为 ZIP 下载
10. 点击预览区图片可打开全屏图像查看器灯箱，支持鼠标滚轮缩放（20%-800%）、拖拽平移、键盘 ← → 导航切换图片，多图场景显示"1/N"计数器
11. 裁剪步骤勾选后，点击"全屏框选"按钮可打开全屏裁剪灯箱，拖拽框选裁剪区域，支持清除选区
12. 结果预览区支持"单图预览"/"对比预览"模式切换；对比模式下通过滑块拖拽对比原图与处理后图片的效果差异