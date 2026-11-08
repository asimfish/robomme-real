# Franka 记忆真机实验方案

Astra 直接控制 vs. Astra + 记忆 Harness 在 Franka 上的真机实验方案，对照 RoboMME / RoboMME-Plus 的四类记忆任务。

- 在线页面：https://asimfish.github.io/robomme-real/
- 页面源文件：`index.html`（单文件，没有构建步骤）

## 更新方法

页面从 `docs/site` 分支发布（Settings → Pages → Branch: `docs/site`，目录 `/`）。

1. 直接编辑 `index.html`。
2. 在 `docs/site` 分支上提交并推送：

   ```bash
   git add index.html
   git commit -m "[site/docs]: update results"
   git push
   ```

3. GitHub Pages 会在 1–2 分钟内自动重新部署。

## 文件

| 文件 | 内容 |
|---|---|
| `index.html` | 实验方案页面（以此为准） |
| `real_robot_experiment_design.md` | 早期 Markdown 草稿（v0.2），已不再更新 |
| `.nojekyll` | 让 GitHub Pages 直接发布静态文件，不经过 Jekyll |

页面带有 `noindex` 标记，搜索引擎不会收录，但任何拿到链接的人都能打开。
