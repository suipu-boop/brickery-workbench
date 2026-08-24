# 内置积木（Builtin Bricks）：小积木默认进组装区，不占待选区

- 状态：待实施（2026-08-24，用户拍板）
- 影响仓库：brickery-workbench（工坊 Web UI 侧）
- 用户决策原文：「这些都是内置的不占空间的小功能。让他们直接出现在组装区吧，不要再出现在待选区就行了。」

## 背景与动机

积木本体是几 KB 的 JSON（元数据 + 注入提示），只有带引擎二进制的积木（如 high-config-doc 的 193MB editor_sdk）才需要按需下载。
把不占空间的小积木默认内置进底座，用户组装 agent 时零选择、开箱即用；待选区只留需要按需下载的扩展积木。

## 判定标准

- **内置（builtin）**：brick.json 未声明二进制（`binary_size` 为空或 0）→ 默认进组装区，不出现在待选区。
- **扩展（market）**：声明了二进制（`binary_size` > 0）→ 留在待选区，按需选装。
- 判定基于 `binary_size` 动态计算，不写死 id 列表：未来无二进制的新积木自动内置，带引擎的自动留市场。

## 改动清单

### 1. `brickery/web/live_vault.py` — 透传二进制声明

`fetch_bricks_online` 两个分支（完整详情 / 降级摘要）都补 `binary_size`：

- 完整详情分支：`"binary_size": int(raw_manifest.get("binary_size") or 0)`
- 降级分支（详情拉取失败）：`"binary_size": 0`（保守按无二进制，但 `_partial` 条目由后端统一降级为非内置，见下）

### 2. `brickery/web/server.py` — /api/bricks 增加 builtin 标记

`_api_bricks` 构造 item 时：

```python
"builtin": not (b.get("binary_size") or 0) and not b.get("_partial"),
```

- 无二进制 → builtin=True
- `_partial`（详情拉取失败、无法确认）→ 强制 builtin=False，避免误内置

### 3. `web/index.html` — 前端分流

- `loadBricks()` 加载后：对 `bricks` 中 `builtin === true` 的积木自动 `selected.add(b.name)`（进组装区）。
- `renderBrickList()`：过滤条件加 `if (b.builtin) return false;`（内置积木不再出现在待选区）。
- `renderCatTabs()` / `collapseAll()` / `expandAll()` / 计数：分类与数量统计基于「非内置」积木，避免空分类与计数错位。
- `downloadBatch()`：`currentFiltered` 已随 renderBrickList 过滤 builtin，批量下载不会包含内置积木（内置本就在底座，无需打包）。
- `renderAssembly()`：内置积木 chip 加「内置」角标；`removeSelect()` 对 builtin 积木直接忽略（内置不可移除，随底座常驻）。
- 依赖提示：内置积木若 `requires` 引用其他内置积木，selected 已自动包含，组装体检照常通过。

### 4. 内核 produce 链路 — 不改

`/api/produce` 与 `/api/assemble` 均以 `selected`（含内置积木）为准：内置积木随组装计划进入产物 `bricks/` 快照，自包含、不依赖积木库。`BRICK_TIERS` 出包模式机制保持不变（另一套写死分层，不冲突）。

## 体验效果

- 组装区：底座自带积木（如 PDF 提取、会议纪要、代码审查等）默认在场，标「内置」，不可移除。
- 待选区：只剩需要按需下载的扩展积木（如 high-config-doc 大引擎、feishu 等），用户组装只操心扩展部分。
- 产物：内置积木的快照照常打进 agent 包，开箱即用。

## 验证

1. `/api/bricks`：无二进制积木 `builtin: true`，high-config-doc `builtin: false`。
2. 工坊页面：刷新后组装区已含内置积木（标「内置」），待选区无内置积木，分类计数一致。
3. 组装校验 + 产出：内置积木进 `plan.order` 与产物 `bricks/` 快照。
4. 降级场景：模拟 brick.json 拉取失败（`_partial`），该积木不误内置。
