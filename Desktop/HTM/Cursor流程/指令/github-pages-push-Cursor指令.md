# Cursor Command: GitHub Pages 推送

## 📋 指令说明

这是 GitHub Pages 自动推送 技能的 Cursor Command 指令定义，必须与 `12-github-pages-push.md` 保持同步。

**同步更新规则**：当 `12-github-pages-push.md` 更新时，本指令也必须同步更新。

---

## 💻 Cursor Command 定义

```
--- Cursor Command: GitHub Pages 推送.md ---
使用工作流：GitHub Pages 自动推送

请根据用户提供的 GitHub Pages URL，自动完成文件同步、提交和推送。

**输入**：GitHub Pages URL（任意格式）
- 完整文件路径：`https://<用户名>.github.io/<仓库名>/<文件名>`
- 根目录：`https://<用户名>.github.io/<仓库名>/` 或 `https://<用户名>.github.io/<仓库名>`

**要求**：

1. **URL 解析**：
   - 从URL中提取仓库名（如：`xuyou`）
   - 从URL中提取文件名（如：`TYVISV0.1.html`）
   - 如果URL是根目录（以 `/` 结尾或没有文件路径），默认处理 `index.html`
   - 根据仓库名映射到本地仓库根目录（如：`xuyou` → `/Users/xuyou`）

2. **仓库确认**：
   - 进入仓库根目录：`cd <仓库根目录>`
   - 验证仓库：`git rev-parse --show-toplevel`
   - 验证远程：`git remote -v`（确认远程地址包含提取的仓库名）

3. **先下载并备份线上最新版本（强制）**：
   - 说明：**永远以GitHub Pages链接上的文件为“最新基线”**，避免本地备份/原型文件不是最新导致误改。
   - 备份目录：`<仓库根目录>/Desktop/HTM/中远海/备份/`
   - 命令（示例）：
     - `mkdir -p "<备份目录>"`
     - `timestamp=$(date +%Y%m%d_%H%M%S)`
     - `curl -L "<GitHub Pages URL>" -o "<备份目录>/<文件名>.backup_${timestamp}"`
     - 验证下载成功（文件存在且非空），失败则**停止流程**。

4. **确定本次要推送的源文件（按优先级）**：
   - 优先1：用户本次实际修改的文件（通常在`<仓库根目录>/Desktop/HTM/中远海/原型/`下某个模块目录内）
   - 优先2：刚刚从GitHub Pages下载并备份的“线上最新版本”（当本地文件不存在/不确定是否最新时，先复制为本地原型基线再改）

5. **同步文件**：
   - 复制到仓库根目录：`cp <找到的源文件> <仓库根目录>/<文件名>`

6. **验证差异**：
   - `cd <仓库根目录>`
   - `git diff <文件名> | head -50`

7. **添加文件**（关键：使用相对路径）：
   - `cd <仓库根目录>`
   - `git add <文件名>`
   - `git diff --cached --name-only`  # 验证路径应该是 `<文件名>`，不是 `Desktop/HTM/...`

8. **提交并推送**：
   - `git commit -m "更新原型: <文件名>"`
   - `git push origin main`

9. **验证推送**：
   - `git log --oneline -1`
   - `git show HEAD --name-only`
   - `git ls-remote --heads origin main`

**关键注意事项**：
- 必须动态识别仓库根目录，不能硬编码
- 必须使用相对路径 `git add <文件名>`
- 提交前必须验证路径正确
- 支持任意 GitHub Pages URL 格式

**相关文档**：
- 技能文档：`中远海/workflows/skills/12-github-pages-push.md`
- Command 文档：`中远海/workflows/commands/github-pages-push-command.md`
- GitHub 更新注意事项：`中远海/workflows/skills/10-github-update.md`

--- End Command ---
```

---

## 🔄 同步更新机制

### 📌 更新优先级

1. **工作流技能定义**：`中远海/workflows/skills/12-github-pages-push.md`
   - **这是所有规范的权威来源**
   - 所有功能更新必须首先更新此文件

2. **Cursor Command指令**：`Cursor流程/指令/github-pages-push-Cursor指令.md`（本文件）
   - 当技能定义文件更新时，必须同步更新

3. **Command 文档**：`中远海/workflows/commands/github-pages-push-command.md`
   - 当技能定义文件更新时，必须同步更新

### 🔄 同步更新流程

**当 `12-github-pages-push.md` 更新时，必须同步更新本文件**：

1. **检查技能文档更新内容**
   - 查看 `12-github-pages-push.md` 的更新内容
   - 识别哪些内容需要在本文件中同步

2. **更新本文件**
   - 同步命令内容中的要求
   - 更新版本号
   - 更新对应技能版本

3. **检查一致性**
   - 使用下方检查清单逐项检查
   - 确保本文件与技能定义文件保持一致

### 📋 同步更新检查清单

**当 `12-github-pages-push.md` 更新时，必须检查本文件的以下内容**：

#### 功能类同步

- [ ] **URL 解析规则**：仓库名和文件名的提取逻辑是否已同步
- [ ] **仓库根目录映射**：仓库名到本地路径的映射规则是否已更新
- [ ] **文件查找策略**：查找源文件的优先级和命令是否已同步
- [ ] **执行步骤**：所有步骤的详细说明是否已更新
- [ ] **错误处理**：常见错误及解决方案是否已包含

#### 命令类同步

- [ ] **命令内容**：Cursor Command定义中的要求是否已更新
- [ ] **使用示例**：示例是否与最新规范一致
- [ ] **注意事项**：关键注意事项是否已同步

#### 文档类同步

- [ ] **版本号**：版本号是否已更新
- [ ] **对应技能版本**：是否已更新为最新的技能版本
- [ ] **最后更新日期**：是否已更新
- [ ] **相关文档链接**：是否已更新

---

**最后更新**：2026-02-27  
**对应技能版本**：v1.0

## 📝 版本历史

- v1.0：初始版本，与 `12-github-pages-push.md` v1.0 同步
