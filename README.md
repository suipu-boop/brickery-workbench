# Brickery Workbench · 积木工坊

**三项目之一（2026-08-22 拆分）**：本仓库 = **积木工坊**，面向用户的组装 + 分发端。

- **积木工坊** → 本仓库 brickery-workbench（市场浏览/组装/网页分发）
- **生成 agent** → 独立仓库 [brickery](https://github.com/suipu-boop/brickery)（内核/底座/产出）
- **积木加工厂** → 独立仓库 [brick-vault](https://github.com/suipu-boop/brick-vault)（积木库/契约/验收）
- 三项目关系与接口契约见 [brickery-meta/ARCHITECTURE.md](https://github.com/suipu-boop/brickery-meta)（会话启动先读）

## 定位

面向用户：浏览积木市场（直连 brick-vault）→ 选积木 → 组装 → 产出 agent 安装包 → 网页下载分发。

## 目录结构

```
app/                  # Swift 壳（与 brickery 共享，双仓库各存一份，改动需同步）
web/                  # 工作台前端（index.html）
brickery/web/         # 工坊后端（server.py / live_vault.py，构建时覆盖内核 web 子包）
site/                 # 网页下载站（GitHub Pages）
scripts/              # 打包脚本（build_workbench_app.sh）
specs/                # 工坊相关设计文档（workbench-live-market / project-org 等）
temp/                 # 构建中间产物（内嵌 python、内核缓存，不入库）
output/               # 打包产物（.app/.dmg，不入库）
```

## 构建

```bash
scripts/build_workbench_app.sh
```

构建时自动从 GitHub 拉取生成 agent 内核（brickery），合并本仓库 `brickery/web/` 覆盖，产出 `output/BrickeryWorkbench-<version>.dmg`。

## 版本与发布

- Release 资产：`BrickeryWorkbench-0.1.0.dmg`（替换式发布，网页下载链接不变）
- 网页站：https://suipu-boop.github.io/brickery-workbench （迁移后）
