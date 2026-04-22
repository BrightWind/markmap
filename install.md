# Markmap 使用方式

## 方式 2：JavaScript/TypeScript API

### 场景 A：浏览器端渲染（React/Vue 等框架）

```bash
pnpm add markmap-lib markmap-view markmap-toolbar
```

```ts
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view';

const transformer = new Transformer();

// 解析 Markdown → 节点树 + 所需资源
const { root, features } = transformer.transform(`
# Hello
## World
## Markmap
`);
const assets = transformer.getUsedAssets(features);

// 渲染到页面上一个 <svg id="mindmap"> 元素
const mm = Markmap.create(
  '#mindmap',
  {
    // 根节点展开，其他节点默认折叠
    initialExpandLevel: 1,
  },
  root,
);
```

### 场景 B：Node.js 服务端生成独立 HTML 文件

```bash
pnpm add markmap-lib markmap-render
```

```ts
import { Transformer } from 'markmap-lib';
import { fillTemplate } from 'markmap-render';
import fs from 'fs';

const transformer = new Transformer();
const { root, features } = transformer.transform(markdownText);
const assets = transformer.getUsedAssets(features);

const html = fillTemplate(root, assets, {
  jsonOptions: {
    // 根节点展开，其他节点默认折叠
    initialExpandLevel: 1,
  },
});
fs.writeFileSync('output.html', html);
```

生成的 `output.html` 是一个完整的单页思维导图，可直接在浏览器打开，无需额外依赖。

### CLI 场景下（`markmap-cli`）默认折叠

`markmap-cli` 现在默认等价于：

```bash
--initial-expand-level 1
```

即不改 Markdown 也会默认折叠子节点。

如需覆盖默认值，可在命令行指定：

```bash
pnpm --filter markmap-cli exec node ./bin/cli.js ./demo.md -o ./demo.markmap.html --initial-expand-level 2 --no-open
```

如果你是通过命令行把 Markdown 转成 HTML，建议在 Markdown 顶部加 frontmatter：

```md
---
markmap:
  initialExpandLevel: 1
---

# 你的主题
## 子节点 A
## 子节点 B
```

`initialExpandLevel: 1` 表示仅展开根节点，其他节点默认折叠。

---

## 方式 6：MCP Server

第三方包 [`markmap-mcp-server`](https://github.com/jinzcdev/markmap-mcp-server)，让 AI 工具（如 Claude Desktop、VS Code Copilot）能通过 MCP 协议调用 markmap 生成思维导图。

```bash
npm install -g @jinzcdev/markmap-mcp-server
```

在 MCP 客户端（如 Claude Desktop 的 `claude_desktop_config.json`）中配置：

```json
{
  "mcpServers": {
    "markmap": {
      "command": "npx",
      "args": ["-y", "@jinzcdev/markmap-mcp-server"]
    }
  }
}
```

---

## 使用本地修改的代码

`pnpm add markmap-lib` 安装的是 npm 上发布的版本。如果改了本项目代码，有以下方案：

### 方案一：先 build，再 `pnpm link`（推荐）

```powershell
# 在 markmap 项目根目录，构建修改的包
pnpm --filter markmap-lib build
pnpm --filter markmap-view build
pnpm --filter markmap-render build

# 在 markmap-lib 包目录，注册全局 link
cd packages/markmap-lib
pnpm link --global

# 在目标项目目录
cd D:\your-project
pnpm link --global markmap-lib
```

### 方案二：本地路径安装

在目标项目 `package.json` 里：

```json
{
  "dependencies": {
    "markmap-lib": "file:D:/workspace_1/SPKES/markmap/packages/markmap-lib",
    "markmap-view": "file:D:/workspace_1/SPKES/markmap/packages/markmap-view"
  }
}
```

然后 `pnpm install`。

### 方案三：加入 monorepo

在 `pnpm-workspace.yaml` 里加上新项目路径，包间通过 workspace 协议自动引用，无需 link。

> **注意：方案一和方案二改代码后必须先 `build`，目标项目消费的是 `dist/` 产物，不是源码。**
