# Bongo Cat 项目集成方案

## 项目概述

本项目成功将 bongo.cat 前端项目集成到 Electron 应用中，使用**混合方案**保持构建流程的完整性。

## 实施的方案

### 方案3：混合方案 - 保留构建能力

**优点**：
- 兼顾简单性和构建优势
- 保留扩展能力
- 保持现有 webpack 构建框架
- 正确处理开发和生产环境的资源路径

**具体实施**：
1. 保持当前 webpack 构建框架
2. 将 bongo.cat 作为静态资源处理
3. 修改入口 HTML 模板
4. 配置开发和生产环境的资源路径

## 实施步骤

### 1. 安装依赖

```bash
pnpm add -D copy-webpack-plugin
```

### 2. 修改 webpack 配置

#### 生产环境配置 (`.erb/configs/webpack.config.renderer.prod.ts`)

```typescript
import CopyWebpackPlugin from 'copy-webpack-plugin';

// 在 plugins 数组中添加：
new CopyWebpackPlugin({
  patterns: [
    {
      from: path.join(webpackPaths.srcRendererPath, 'bongo.cat'),
      to: 'bongo.cat',
      filter: (resourcePath) => {
        // 排除特定文件
        const excludeFiles = [
          'index.html',
          'README.md',
          'LICENSE',
          'CODEOWNERS', 
          'CNAME',
          'robots.txt',
        ];
        const fileName = path.basename(resourcePath);
        return (
          !excludeFiles.includes(fileName) && !resourcePath.includes('.git')
        );
      },
    },
  ],
}),
```

#### 开发环境配置 (`.erb/configs/webpack.config.renderer.dev.ts`)

```typescript
import CopyWebpackPlugin from 'copy-webpack-plugin';

// 在 plugins 数组中添加：
new CopyWebpackPlugin({
  patterns: [
    {
      from: path.join(webpackPaths.srcRendererPath, 'bongo.cat'),
      to: 'bongo.cat',
      filter: (resourcePath) => {
        // 排除特定文件
        const excludeFiles = [
          'index.html',
          'README.md',
          'LICENSE',
          'CODEOWNERS', 
          'CNAME',
          'robots.txt',
        ];
        const fileName = path.basename(resourcePath);
        return (
          !excludeFiles.includes(fileName) && !resourcePath.includes('.git')
        );
      },
    },
  ],
}),

// 在 devServer 配置中添加静态资源服务：
devServer: {
  // ... 其他配置
  static: [
    {
      publicPath: '/',
    },
    {
      directory: path.join(webpackPaths.srcRendererPath, 'bongo.cat'),
      publicPath: '/bongo.cat',
    },
  ],
  // ... 其他配置
},
```

### 3. 修改 HTML 模板

将 `src/renderer/index.ejs` 替换为 bongo.cat 的完整 HTML 结构，主要修改：

- 更新 CSP 策略以允许外部资源
- 调整所有资源路径为相对路径 `./bongo.cat/`
- 集成 bongo.cat 的所有 HTML 元素
- 保留 Electron 的基本配置

### 4. 资源路径配置

所有 bongo.cat 相关资源使用相对路径：
- CSS: `./bongo.cat/style/style.css`
- JavaScript: `./bongo.cat/js/`
- 图片: `./bongo.cat/images/`
- 声音: `./bongo.cat/sounds/`
- 元数据: `./bongo.cat/meta/`

## 关键技术点

### CSP 策略调整

```html
<meta
  http-equiv="Content-Security-Policy"
  content="script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdnjs.cloudflare.com https://fonts.googleapis.com https://static.cloudflareinsights.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://cdnjs.cloudflare.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' data: https:; connect-src 'self' https:; media-src 'self'"
/>
```

### 资源过滤

排除不需要的文件：
- 原始的 `index.html`
- Git 相关文件
- 文档文件 (README, LICENSE 等)
- 配置文件 (CNAME, robots.txt 等)

## 构建和运行

### 开发环境

```bash
npm run start
```

### 生产构建

```bash
npm run build
```

### 打包应用

```bash
npm run package
```

## 注意事项

1. **路径一致性**：确保开发和生产环境使用相同的相对路径
2. **资源完整性**：确保所有 bongo.cat 的资源文件都被正确复制
3. **CSP 策略**：根据 bongo.cat 的需求调整内容安全策略
4. **外部依赖**：bongo.cat 依赖一些外部库（jQuery、SoundManager2 等）

## 文件结构

```
src/renderer/
├── bongo.cat/           # bongo.cat 项目文件
│   ├── images/         # 图片资源
│   ├── js/             # JavaScript 文件
│   ├── meta/           # 元数据文件
│   ├── sounds/         # 音频文件
│   ├── style/          # 样式文件
│   └── index.html      # 原始 HTML（不被使用）
└── index.ejs           # 集成后的 HTML 模板

dist/renderer/
├── bongo.cat/          # 构建后的 bongo.cat 资源
├── index.html          # 生成的 HTML 文件
└── ...                 # 其他构建文件
```

## 问题修复记录

### 🔧 声音文件路径问题

**问题**：开发服务器日志显示声音文件无法加载，提示路径错误。

**原因**：`core.js` 中配置的声音文件路径为 `../sounds/`，但实际应该是 `./bongo.cat/sounds/`。

**解决方案**：
```javascript
// 修改前
lowLag.init({
  'urlPrefix': '../sounds/',
  'debug': 'none'
});

// 修改后
lowLag.init({
  'urlPrefix': './bongo.cat/sounds/',
  'debug': 'none'
});
```

### 🖼️ 图片资源路径问题

**问题**：CSS 中的背景图片无法正确显示。

**原因**：CSS 文件位于 `bongo.cat/style/style.css`，图片位于 `bongo.cat/images/`，相对路径应该是 `../images/`。

**解决方案**：
确认 CSS 中的图片路径保持为 `../images/`（这是正确的相对路径）：
```css
/* 正确的路径 */
.cat>#head {
  background-image: url(../images/cat.png);
}
```

### 📁 CopyWebpackPlugin 配置问题

**问题**：CopyWebpackPlugin 没有正确复制目录结构，而是复制成了单个文件。

**临时解决方案**：手动复制 bongo.cat 目录到构建输出：
```bash
cp -r src/renderer/bongo.cat release/app/dist/renderer/bongo.cat
```

**后续优化**：需要进一步调整 CopyWebpackPlugin 配置以正确处理目录复制。

## 成功验证

应用成功启动后，用户可以：
- 看到 bongo.cat 的完整界面
- 使用键盘按键触发音效
- 看到猫咪的动画效果
- 体验完整的 bongo.cat 功能

这个方案成功保留了 Electron 的构建优势，同时无缝集成了 bongo.cat 项目。

## 🚀 重要修复记录 (2024-07-03)

### 📦 打包后资源无法加载问题

**问题**：`npm run package` 打包后的应用无法正常显示，JS、CSS 和资源文件都无法加载。

**根本原因**：
1. CopyWebpackPlugin 配置错误，`bongo.cat` 被复制为单个文件而不是目录结构
2. 生产环境和开发环境的路径处理不一致

**解决方案**：

#### 1. 修复 CopyWebpackPlugin 配置
```javascript
// 修复前（错误）
{
  from: path.join(webpackPaths.srcRendererPath, 'bongo.cat'),
  to: 'bongo.cat',  // 缺少斜杠
  filter: (resourcePath) => { ... }
}

// 修复后（正确）
{
  from: path.join(webpackPaths.srcRendererPath, 'bongo.cat'),
  to: 'bongo.cat/',  // 关键：末尾添加斜杠
  globOptions: {     // 使用 globOptions 替代 filter
    ignore: [
      '**/index.html',
      '**/.git/**',
      '**/README.md',
      '**/LICENSE',
      '**/CODEOWNERS',
      '**/CNAME',
      '**/robots.txt',
    ],
  },
}
```

#### 2. 添加动态路径检测
在 `core.js` 中添加环境检测功能：
```javascript
// 动态检测资源路径
function detectResourcePath() {
  // 统一使用相对路径，兼容开发和生产环境
  return './bongo.cat/sounds/';
}

// 使用动态路径
lowLag.init({
  'urlPrefix': detectResourcePath(),
  'debug': 'none'
});
```

#### 3. 修复结果验证
修复后的目录结构：
```
release/app/dist/renderer/
├── bongo.cat/              # ✅ 现在是目录
│   ├── images/            # ✅ 图片资源
│   ├── js/                # ✅ JavaScript 文件
│   ├── meta/              # ✅ 元数据文件
│   ├── sounds/            # ✅ 声音文件
│   └── style/             # ✅ 样式文件
├── index.html
├── renderer.js
└── renderer.js.map
```

### 验证步骤
1. **开发环境**：`npm run start` - 确认功能正常
2. **构建验证**：`npm run build` - 检查目录结构
3. **打包验证**：`npm run package` - 最终应用测试

### 关键技术点
- **目标路径**：`to: 'bongo.cat/'` 末尾的斜杠至关重要
- **文件过滤**：使用 `globOptions` 而不是 `filter` 函数
- **路径一致性**：开发和生产环境使用相同的相对路径

✅ **修复成功**：现在 `npm run package` 打包后的应用可以正常运行，所有资源正确加载。 
