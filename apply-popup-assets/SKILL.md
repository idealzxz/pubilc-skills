---
name: apply-popup-assets
description: 将 src/assets/ 中的素材图片应用到对应弹窗组件的代码中。当用户说素材已放好、图片已放好、资源已准备好、帮我更新素材到代码时使用。
---

# 应用弹窗素材到代码

用户将素材图片放入 `src/assets/{PopName}/` 后，自动识别图片并更新到对应组件的 JSX 和 LESS 中。

## 流程

### 1. 扫描素材

用 Glob 扫描 `src/assets/{PopName}/` 下的所有图片文件。

### 2. 获取图片信息

对每张图片：
- 用 Read 工具查看图片内容，判断用途（标题、背景、按钮、提示文案等）
- 用 `sips -g pixelWidth -g pixelHeight` 获取像素尺寸，直接作为 less 中的宽高值

### 3. 更新组件代码

根据识别到的素材，更新 `src/components/{PopName}/` 下的 JSX 和 LESS：

**LESS 中使用素材**：通过 `.webpBg("{PopName}/{filename}")` 引用，例如：
```less
.title {
  width: 380px;   // 图片实际像素宽
  height: 86px;   // 图片实际像素高
  .webpBg("WinPop/title.png");
}
```

**JSX 中**：为每个素材添加对应的 `<span className="xxx" />` 元素，按钮类元素绑定点击事件。

### 4. 常见素材命名与用途对应

| 文件名 | 用途 | JSX 元素 |
|--------|------|----------|
| bg.png | 弹窗背景 | `<span className="bg" />` |
| title.png | 标题图片 | `<span className="title" />` |
| btn.png | 确认按钮 | `<span className="confirmBtn" onClick={this.onConfirm} />` |
| tips.png | 底部提示文案 | `<span className="tips" />` |
| 其他 | 根据图片内容判断 | 添加对应语义化 className |

### 5. 定位规则

- 所有元素使用 `position: absolute` 定位
- 容器宽度为 750px，元素水平居中时 `left = (750 - width) / 2`
- 元素纵向间距参考设计稿，从上到下依次排列
