# pnpm-yiwen

[![npm](https://img.shields.io/npm/v/yiwen-test-add?label=yiwen-test-add)](https://www.npmjs.com/package/yiwen-test-add)
[![node](https://img.shields.io/badge/node-%3E%3D18-339933)](https://nodejs.org/)
[![pnpm](https://img.shields.io/badge/pnpm-8-F69220)](https://pnpm.io/)
[![lerna](https://img.shields.io/badge/lerna-8.1.2-C084FC)](https://lerna.js.org/)

用 **pnpm workspace** 和 **Lerna** 搭起来的最小 monorepo 练习仓库。里面有一个已发布到 npm 的函数包 `yiwen-test-add`，以及一个在本地引用它的私有示例包 `yiwen-demo`。

2024-03-13，仓库用 Lerna 的固定版本模式把子包版本记到 `0.0.12`。npm 上的 [`yiwen-test-add@0.0.12`](https://www.npmjs.com/package/yiwen-test-add) 与本仓库当前代码一致。

## 包一览

| 目录 | 包名 | 发布情况 | 做什么 |
| --- | --- | --- | --- |
| `packages/yiwen-add` | `yiwen-test-add` | npm `0.0.12` | 导出函数，返回 `a / b` |
| `packages/demo` | `yiwen-demo`（`private: true`） | 注册表中不可用 | 本地引用上面的包并打印结果 |

### `yiwen-test-add`

目录名是 `packages/yiwen-add`，npm 包名是 `yiwen-test-add`。入口依赖 `lodash` 的 `divide`：

```js
const _ = require('lodash');

module.exports = function (a, b) {
    console.log('yiwen-add');
    return _.divide(a, b);
}
```

调用 `fn(1, 2)` 时，标准输出先打印一行 `yiwen-add`，返回值是 `0.5`。

npm 上的 `0.0.12` 压缩包包含 `index.js`、`package.json` 和空的 `LICENSE.md`。

### `yiwen-demo`

`packages/demo/package.json` 用 `"yiwen-test-add": "workspace:*"` 依赖工作区里的包。`pnpm install` 之后，`packages/demo/node_modules/yiwen-test-add` 会链接到 `packages/yiwen-add`。

`packages/demo/index.js` 的内容是：

```js
const yiwen=require('@yiwen/test-add')

console.log(yiwen(1,2)) // 0.5
```

这里的模块名是 `@yiwen/test-add`，工作区里实际安装的包名是 `yiwen-test-add`。安装完成后执行 `node packages/demo/index.js` 会抛出 `Cannot find module '@yiwen/test-add'`。注释里的 `0.5` 与 `1 / 2` 的结果一致，但这条示例入口目前跑不通。

## 技术栈

| 依赖 | 版本 | 说明 |
| --- | --- | --- |
| Node.js | `>= 18` | 来自 `lerna@8.1.2` 的 `engines`。下文命令在 Node.js `v22.23.4` 上执行过 |
| pnpm | 8（验证时为 `8.15.9`） | `pnpm-lock.yaml` 的 `lockfileVersion` 为 `6.0` |
| Lerna | `8.1.2` | 根目录依赖。`lerna.json` 指定 `npmClient: pnpm`，版本模式为固定版本 `0.0.12` |
| lodash | `4.17.21` | `yiwen-test-add` 的运行时依赖 |

根包名是 `pnpm-yiwen`，版本号是 `1.0.0`。它不在 `packages/` 下，版本不跟随 Lerna。根 `package.json` 的 `main` 指向 `index.js`，根目录没有这个文件。

## 目录结构

```text
.
├── lerna.json              # Lerna：固定版本 0.0.12，客户端为 pnpm
├── package.json            # 根包 pnpm-yiwen，依赖 lerna
├── pnpm-workspace.yaml     # workspace 包含 packages/*
├── pnpm-lock.yaml          # pnpm 8 锁文件
├── LICENSE.md              # 空文件
├── lerna-debug.log         # 一次失败的 lerna version 调试日志
└── packages/
    ├── yiwen-add/          # 包名 yiwen-test-add
    │   ├── index.js
    │   └── package.json
    └── demo/               # 私有包 yiwen-demo
        ├── index.js
        └── package.json
```

`.gitignore` 只忽略 `node_modules/`。仓库里没有截图。

## 环境要求

- Node.js 18 或更高版本
- 能读取当前锁文件的 pnpm 8

本环境自带的 pnpm `10.33.4` 执行 `pnpm install --frozen-lockfile` 会报 `ERR_PNPM_LOCKFILE_BREAKING_CHANGE`。下面改用 Corepack 准备的 pnpm `8.15.9`。

## 安装

在仓库根目录执行：

```bash
corepack enable
corepack prepare pnpm@8.15.9 --activate
pnpm install --frozen-lockfile
```

已在 Node.js `v22.23.4` 上验证：安装完成（539 个包），并链上 `yiwen-test-add`。

## 运行

在仓库根目录直接调用本地包：

```bash
node -e "console.log(require('./packages/yiwen-add')(1, 2))"
```

已验证的输出：

```text
yiwen-add
0.5
```

列出 Lerna 管理的包：

```bash
pnpm exec lerna ls
```

已验证：CLI 为 `lerna notice cli v8.1.2`，随后列出 `yiwen-demo (PRIVATE)` 和 `yiwen-test-add`。

运行示例入口（当前会失败）：

```bash
node packages/demo/index.js
```

已验证：进程以状态码 1 退出，错误为 `Cannot find module '@yiwen/test-add'`。

在其它目录使用 npm 上已发布的包：

```bash
npm install yiwen-test-add@0.0.12
node -e "console.log(require('yiwen-test-add')(1, 2))"
```

已验证：下载 `yiwen-test-add-0.0.12.tgz` 并安装其依赖后，执行 `(1, 2)` 同样打印 `yiwen-add`，返回值为 `0.5`。

## 测试

仓库没有测试用例。根包和两个子包的 `test` 脚本都是占位命令，执行后打印 `Error: no test specified` 并以状态码 1 退出。`pnpm test` 已按这个结果验证。

## 许可证与其它说明

- 根包和 `yiwen-test-add` 的 `package.json` 声明许可证为 `MIT`，`yiwen-demo` 声明为 `ISC`。
- 仓库中的 `LICENSE.md` 是空文件，发布到 npm 的压缩包里附带的也是这份空文件。
- `lerna-debug.log` 记录了一次 `lerna version` 失败：更新远端 `https://github.com/Hao-yiwen/pnpm-yiwen.git` 时连接中断。它是调试残留，不影响本地调用上面的函数。

## 参考

- 仓库：<https://github.com/Hao-yiwen/pnpm-yiwen>
- npm：<https://www.npmjs.com/package/yiwen-test-add>
- [pnpm workspace](https://pnpm.io/workspaces)
- [Lerna](https://lerna.js.org/)
