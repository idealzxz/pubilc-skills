---
name: create-popup
description: 基于模板创建弹窗组件并自动注册到 modal store。当用户要求创建弹窗、新增弹窗组件、添加 Pop 弹窗时使用。
---

# 创建弹窗组件

基于项目弹窗模板，创建新的弹窗组件并自动注册到 `src/modal/modal.jsx` 的 `cfg` 配置中。

## 流程

### 1. 确定弹窗名称

弹窗组件名使用 **PascalCase**，以 `Pop` 结尾（如 `WinPop`、`NotWinPop`、`RulePop`）。

### 2. 创建组件文件

在 `src/components/{PopName}/` 下创建两个文件：

**{PopName}.jsx** — 基于以下模板，将所有 `__PopName__` 替换为实际名称：

```jsx
'use strict';

import React from 'react';
import { observer } from 'mobx-react';
import './__PopName__.less';
import modalStore from '@src/store/modal';

@observer
class __PopName__ extends React.Component {
  onClose = () => {
    modalStore.closePop('__PopName__');
  };

  onConfirm = () => {
    const { popData = {} } = this.props;
    if (popData.onConfirm) {
      popData.onConfirm();
    }
    modalStore.closePop('__PopName__');
  };

  render() {
    const { popData = {} } = this.props;
    const { confirmText = '知道了' } = popData;

    return (
      <div className="__PopName__ modal_center">
        <span className="bg" />
        <span className="confirmBtn" onClick={this.onConfirm}>
          {confirmText}
        </span>
        <span className="close" onClick={this.onClose} />
      </div>
    );
  }
}

export default __PopName__;
```

**{PopName}.less** — 基于以下模板：

```less
@import "../../res.less";

.__PopName__ {
  width: 750px;
  height: 1624px;
  left: 0px;
  top: 0px;
  position: absolute;
  .popupCenterShow();

  .bg {
    width: 620px;
    height: 500px;
    left: 65px;
    top: 400px;
    position: absolute;
    border-radius: 30px;
    // 替换弹窗背景图: .webpBg("__PopName__/bg.png");
    background: linear-gradient(180deg, #fef4c8 0%, #fff8dc 100%);
  }

  .confirmBtn {
    width: 360px;
    height: 90px;
    left: 195px;
    top: 940px;
    position: absolute;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 32px;
    color: #6a2a00;
    font-family: 'AlimamaFangYuanTiVF';
    font-variation-settings: 'wght' 700, 'BEVL' 80;
    border-radius: 45px;
    // 替换按钮背景图: .webpBg("__PopName__/btn.png");
    background: #f9d652;
  }

  .close {
    width: 63px;
    height: 63px;
    left: 344px;
    top: 1070px;
    position: absolute;
    .webpBg("Common/closeBtn.png");
  }
}
```

### 3. 创建素材文件夹

在 `src/assets/{PopName}/` 下创建同名文件夹，用于存放弹窗的背景图、按钮图等素材资源。

### 4. 注册到 modal 配置

编辑 `src/modal/modal.jsx`：

1. 在 import 区域末尾添加：
   ```jsx
   import __PopName__ from "@src/components/__PopName__/__PopName__";
   ```

2. 在 `cfg` 对象中添加：
   ```jsx
   __PopName__: __PopName__,
   ```

### 5. 调用方式

```jsx
import modalStore from '@src/store/modal';

modalStore.pushPop('__PopName__', {
  confirmText: '知道了',
  onConfirm: () => { /* 回调 */ },
});
```

## 注意事项

- 弹窗的蒙层和层级由框架 `src/modal/modal.jsx` 统一管理，组件内不需要自带蒙层
- 组件根元素必须添加 `modal_center` class 以实现居中
- 使用 `.popupCenterShow()` mixin 实现弹出缩放动画
- 关闭按钮统一使用 `Common/closeBtn.png`
- 背景图和按钮图由使用者在 less 中通过 `.webpBg()` 自行替换
