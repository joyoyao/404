# 静态站点首页与 404 页

## 背景

工作区 `/Users/yaozaiyong/porter/static` 目前为空。Porter 通过 `STATIC_FILE_PATH`（默认 `/porter/static`）把该目录作为静态资源根目录提供，静态路由默认首页是 `index.html`，非 CSR 站点会预热并回退到 `404.html`。需要补齐这两个独立页面，风格与现有部署等待页 `porter/services/deploy_init_container/assets/init.html` 一致：居中、浅灰底、无构建依赖。

## 需求与处理逻辑

- `index.html`：站点根路径页面。说明这是帽子云静态站点，并提供进入方式说明。
- `404.html`：资源不存在时的页面。明确 404，并提供返回首页的链接（`/`）。
- 两个页面都是单文件 HTML，样式内联，不依赖外部字体、脚本或图片，避免静态根目录缺资源时页面破损。
- 语言为简体中文，`lang="zh-CN"`。

## 技术方案

不引入构建工具或公共 CSS。两页共用同一套视觉：背景 `#e3e5e8`，标题色 `#46484a`，说明色 `#787878`，主按钮 `#5c93ed`。字体使用系统字体栈（含苹方 / 微软雅黑），不请求 Google Fonts。

## 影响文件

| 文件 | 操作 |
| --- | --- |
| `/Users/yaozaiyong/porter/static/index.html` | 新建 |
| `/Users/yaozaiyong/porter/static/404.html` | 新建 |

不修改 Porter 路由、Helm 或网关配置。

## 实现要点

共用骨架：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>...</title>
  <style>
    body {
      margin: 0;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      background: #e3e5e8;
      font-family: "PingFang SC", "Microsoft YaHei", "Work Sans", sans-serif;
      color: #46484a;
    }
    main { width: min(640px, calc(100% - 48px)); text-align: center; }
    a.button {
      display: inline-block;
      margin-top: 28px;
      padding: 14px 28px;
      background: #5c93ed;
      color: #fff;
      text-decoration: none;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <main>...</main>
</body>
</html>
```

- `index.html`：标题「帽子云」，主文案说明静态站点已就绪，按钮链到当前目录说明（无其他页面，不伪造导航）。不放外链统计脚本。
- `404.html`：`meta robots` 为 `noindex, nofollow`；大号「404」；文案「页面不存在」；按钮「返回首页」指向 `/`。

## 边界

- 不假设 favicon、logo 或其它静态资源存在。
- 404 页不使用 JS 跳转，避免把错误页变成自动重定向。
- 页面在窄屏下不横向溢出（`width: min(...)`）。

## 预期结果

打开 `index.html` 看到首页；打开 `404.html` 看到 404，点击「返回首页」回到 `/`。两页均可离线打开。
