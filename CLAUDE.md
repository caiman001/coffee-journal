# BeanDance — Agent 工作指南

单文件咖啡冲煮手帐 PWA。线上版（V100）+ 全面改版小样（proto-receipt.html）。
产品与设计决策以 [docs/REDESIGN.md](docs/REDESIGN.md) 为准，先读它再动手。

## 硬约束（违反会造成数据丢失或架构破坏）

1. **`DB_NAME = 'BeanDance_V46_Stable'` 绝对不能改** —— IndexedDB 按此名存储用户数据，改名即清空所有用户记录
2. **单文件架构，零构建**：主应用是一个 index.html，不引入打包工具/框架/npm 依赖；用户习惯在 GitHub 网页直接改代码
3. **全部用相对路径**：部署在 GitHub Pages 子路径 `/beandance/` 下，绝对路径会 404
4. **发版时 bump `sw.js` 的 CACHE_NAME**（如 beandance-v100 → v101），否则老用户收不到更新
5. 与用户沟通用中文

## 文件地图

| 文件 | 说明 |
|---|---|
| `index.html` | 线上正式版（V100：暗黑玻璃风 + 烘焙度色彩主题） |
| `proto-receipt.html` | **改版小样 v10**（灵动岛打印机 + 便签画布），改版蓝图 |
| `sw.js` / `manifest.json` | PWA 外壳 |
| `docs/REDESIGN.md` | 改版设计文档：交互模型、视觉定稿、决策日志、backlog |
| `docs/*.png` | README 截图 |

## 常用工作流

- **本地预览**：`.claude/launch.json` 已配置 `python3 -m http.server 4173`；demo 数据加 `?demo`（不碰真实 IndexedDB）
- **截图**：headless Chrome（注意其最小视口宽 500px，`--window-size=390` 会被忽略）；demo 模式支持 `?demo&card=N&flip` 定位
- **部署**：push main → GitHub Pages 自动构建（约 1 分钟），线上 https://caiman001.github.io/beandance/
- **小样分享**：发布为 Artifact（同一文件路径重发即同链接），历史链接 https://claude.ai/code/artifact/4f340ca1-79a4-45a6-8512-d9e8ece7fa55
- 迭代小样时：改完必须本地预览验证（开合、打印、拖拽、编辑四条链路），用真实点击验证，合成 PointerEvent 在预览环境有假象

## 数据模型（IndexedDB store: coffees）

```js
// id=-1 元数据{title,author}，id=0 搜索卡背景图，id>0 豆子：
{ id, name, powder, liquid, time, grind,          // 字符串
  roast: 'light'|'medium'|'dark', roastDate: 'YYYY-MM-DD'|'',
  imageData: Blob|null, created: timestamp,
  brews: [{ date, powder, liquid, time, grind }] }
// 改版将新增：x, y, tilt（画布位置）；origin（产地）
// 备份格式 v2：{ app, version:2, exportedAt, meta, searchBg, beans:[{...bean, image: base64}] }
// 兼容 v1（裸数组、无图）导入
```
