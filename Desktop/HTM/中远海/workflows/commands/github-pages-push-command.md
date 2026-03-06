# Command：GitHub Pages 自动推送

## 📋 Command 说明

这是一个用于自动更新 GitHub Pages 原型的 command。用户只需提供任意 GitHub Pages URL，系统会自动识别文件名、定位仓库、同步文件并推送到 GitHub。

## 🎯 使用方式

### 标准格式

```
更新GitHub原型：<GitHub Pages URL>
```

### 支持的URL格式示例

```
更新GitHub原型：https://bdgoodbr.github.io/xuyou/TYVISV0.1.html
更新GitHub原型：https://bdgoodbr.github.io/xuyou/lhz0105.html
更新GitHub原型：https://bdgoodbr.github.io/xuyou/index.html
更新GitHub原型：https://bdgoodbr.github.io/xuyou/
更新GitHub原型：https://bdgoodbr.github.io/xuyou
```

**说明**：
- 支持完整文件路径：`/xuyou/TYVISV0.1.html`
- 支持根目录：`/xuyou/` 或 `/xuyou`（默认处理 `index.html`）
- 自动从URL中提取文件名和仓库信息

## 🔄 执行流程

当用户提供任意 GitHub Pages URL 时，系统将自动执行以下步骤：

1. **解析URL**：
   - 从URL中提取仓库名（如：`xuyou`）
   - 从URL中提取文件名（如：`TYVISV0.1.html`）
   - 如果URL是根目录（`/xuyou/` 或 `/xuyou`），默认处理 `index.html`
   - 根据仓库名映射到本地仓库根目录（如：`xuyou` → `/Users/xuyou`）

2. **定位仓库**：
   - 确认正确的Git仓库根目录
   - 验证远程仓库地址匹配

3. **先下载并备份线上当前版本（强制）**：
   - **规则**：永远以用户提供的GitHub Pages URL对应的文件作为“最新基线”，先下载并备份后再改，避免本地备份不是最新导致误改。
   - 建议命令：`curl -L "<GitHub Pages URL>" -o "<仓库根目录>/Desktop/HTM/中远海/备份/<文件名>.backup_<时间戳>"`
   - **下载失败则停止流程**（不要直接用本地旧备份继续）。

4. **确定本次要推送的源文件**：
   - 优先使用“用户本次实际修改后的文件”（通常位于`<仓库根目录>/Desktop/HTM/中远海/原型/`某目录）
   - 若本地不确定是否最新/文件不存在：先将步骤3下载的线上版本复制为本地原型基线，再在其上修改

5. **同步文件**：将更新后的文件复制到仓库根目录

6. **验证差异**：查看文件变更内容

7. **提交文件**：使用相对路径添加到Git

8. **验证路径**：确认提交路径正确

9. **推送更新**：推送到GitHub远程仓库

10. **验证结果**：确认推送成功

## 📝 提示词模板（给AI）

```
用户提供了GitHub Pages URL：<URL>

请按照以下步骤执行：

1. 从URL自动提取信息：
   - URL: <URL>
   - 解析规则：
     * 格式：https://<用户名>.github.io/<仓库名>/<文件路径>
     * 提取仓库名：从URL中提取 `<仓库名>` 部分
     * 提取文件名：
       - 如果URL以 `/` 结尾或没有文件路径：默认为 `index.html`
       - 否则：提取URL最后一部分作为文件名（如：`TYVISV0.1.html`）
   - 仓库根目录映射：
     * `xuyou` → `/Users/xuyou`
     * 其他仓库：根据实际情况映射（可通过 git remote 确认）

2. 确认仓库：
   - 进入仓库根目录：cd <仓库根目录>
   - 验证仓库：git rev-parse --show-toplevel
   - 验证远程：git remote -v（确认远程地址包含提取的仓库名）

2.5. **备份并检查 GitHub 上的当前版本（关键！）**：
   - 目的：
     * 从 GitHub Pages 下载当前版本并备份
     * 检查本地原型目录的文件是否基于最新 GitHub 版本
     * 如果本地文件不是基于最新版本，先同步 GitHub 版本到本地原型目录
   - 备份目录：<仓库根目录>/Desktop/HTM/中远海/备份/
   - 备份文件名格式：<文件名>.backup_<时间戳>.html
   - 下载命令：curl -L "<GitHub Pages URL>" -o "<备份文件路径>"
   - 验证：检查文件是否存在且非空，如果下载失败则停止执行
   - **版本一致性检查**：
     * 如果本地原型文件不存在 → 自动同步 GitHub 版本到本地原型目录
     * 如果本地文件与 GitHub 版本差异过大（大小差异>20%或缺少关键内容）→ 先备份本地文件，再同步 GitHub 版本
     * 同步后提示用户在新版本基础上修改，然后重新执行推送
   - **安全保护**：同步前会备份本地文件（带 `.local_backup_` 前缀），不会丢失你的修改

3. 使用本地原型目录中的文件（已确保基于最新版本）：
   - 文件路径：<仓库根目录>/Desktop/HTM/中远海/原型/<对应目录>/<文件名>
   - 说明：经过步骤2.5的检查，本地原型目录中的文件已经确保是基于最新 GitHub 版本的
   - 如果文件不存在或版本不一致，步骤2.5会自动同步，此时会提示用户修改后重新执行

4. 同步文件到仓库根目录：
   - 将本地修改后的文件复制到仓库根目录：cp <找到的源文件> <仓库根目录>/<文件名>
   - 说明：这个文件是你要推送到 GitHub 的修改后版本

5. 验证差异：
   - cd <仓库根目录>
   - git diff <文件名> | head -50

5.5. **代码结构验证（关键！避免Vue结构错误）**：
   - 如果是 HTML/Vue 文件，执行以下检查：
     * 检查重复的 Vue Options API 对象：
       - `grep -c "^\s*computed:" <文件名>`（应该只出现1次）
       - `grep -c "^\s*methods:" <文件名>`（应该只出现1次）
       - `grep -c "^\s*data()" <文件名>`（应该只出现1次）
     * 如果发现重复定义，**必须停止推送**，提示用户修复后再推送
     * 检查 Vue 实例结构完整性：
       - `grep -n "new Vue\|createApp\|Vue.createApp" <文件名>`（确认Vue实例定义存在）
   - 如果检查失败，输出错误信息并停止流程，不执行后续步骤

6. 添加文件（关键：使用相对路径）：
   - cd <仓库根目录>
   - git add <文件名>
   - git diff --cached --name-only  # 验证路径应该是 <文件名>，不是 Desktop/HTM/...

7. 提交并推送：
   - git commit -m "更新原型: <文件名>"
   - git push origin main

8. 验证推送：
   - git log --oneline -1
   - git show HEAD --name-only
   - git ls-remote --heads origin main

⚠️ 关键注意事项：
- 必须动态识别仓库根目录，不能硬编码
- 必须使用相对路径 git add <文件名>
- 提交前必须验证路径正确
- 支持任意GitHub Pages URL格式
- **代码结构验证**：对于 HTML/Vue 文件，必须检查 Vue Options API 结构（computed、methods、data 等不能重复定义），发现问题必须停止推送
```

## 🎯 完整提示词示例

### 示例1：完整文件路径

```
更新GitHub原型：https://bdgoodbr.github.io/xuyou/TYVISV0.1.html

请按照以下步骤执行：

1. 从URL自动提取信息：
   - URL: https://bdgoodbr.github.io/xuyou/TYVISV0.1.html
   - 仓库名: xuyou（从URL路径提取）
   - 文件名: TYVISV0.1.html（从URL路径提取）
   - 仓库根目录: /Users/xuyou（根据仓库名映射）

2. 确认仓库：
   - 进入仓库根目录：cd /Users/xuyou
   - 验证仓库：git rev-parse --show-toplevel
   - 验证远程：git remote -v

2.5. 备份 GitHub 上的当前版本：
   - 备份目录：/Users/xuyou/Desktop/HTM/中远海/备份/
   - 备份文件名：TYVISV0.1.backup_<时间戳>.html
   - 下载命令：curl -L "https://bdgoodbr.github.io/xuyou/TYVISV0.1.html" -o "<备份文件路径>"
   - 验证下载是否成功

3. 查找本地修改后的文件：
   - 优先1：/Users/xuyou/Desktop/HTM/中远海/原型/铁运可视化信息上报/TYVISV0.1.html（这是你修改后的版本）
   - 优先2：/Users/xuyou/Desktop/HTM/中远海/备份/ 目录下最新的备份文件
   - 说明：优先使用原型目录中的文件，因为这是你修改后的版本

4. 同步文件到仓库根目录：
   - 将本地修改后的文件复制到：cp <找到的源文件> /Users/xuyou/TYVISV0.1.html
   - 说明：这个文件是你要推送到 GitHub 的修改后版本

5. 验证差异：
   - cd /Users/xuyou
   - git diff TYVISV0.1.html | head -50

6. 添加文件（关键：使用相对路径）：
   - cd /Users/xuyou
   - git add TYVISV0.1.html
   - git diff --cached --name-only  # 验证路径应该是 TYVISV0.1.html

7. 提交并推送：
   - git commit -m "更新原型: TYVISV0.1.html"
   - git push origin main

8. 验证推送：
   - git log --oneline -1
   - git show HEAD --name-only

⚠️ 关键注意事项：
- 必须在 /Users/xuyou 仓库根目录操作
- 必须使用相对路径 git add TYVISV0.1.html
- 提交前必须验证路径正确
```

### 示例2：根目录URL（默认index.html）

```
更新GitHub原型：https://bdgoodbr.github.io/xuyou/

请按照以下步骤执行：

1. 从URL自动提取信息：
   - URL: https://bdgoodbr.github.io/xuyou/
   - 仓库名: xuyou（从URL路径提取）
   - 文件名: index.html（URL以/结尾，默认为index.html）
   - 仓库根目录: /Users/xuyou（根据仓库名映射）

2. 确认仓库：
   - 进入仓库根目录：cd /Users/xuyou
   - 验证仓库：git rev-parse --show-toplevel
   - 验证远程：git remote -v

3. 查找源文件：
   - 优先查找：/Users/xuyou/Desktop/HTM/中远海/备份/ 目录下最新的 index.html 备份文件
   - 命令：find /Users/xuyou/Desktop/HTM/中远海/备份 -name "*index.html*" -type f -mtime -1 | sort -r | head -1

4. 同步文件：
   - 将找到的文件复制到：cp <找到的文件> /Users/xuyou/index.html

5. 验证差异：
   - cd /Users/xuyou
   - git diff index.html | head -50

6. 添加文件（关键：使用相对路径）：
   - cd /Users/xuyou
   - git add index.html
   - git diff --cached --name-only  # 验证路径应该是 index.html

7. 提交并推送：
   - git commit -m "更新原型: index.html"
   - git push origin main

8. 验证推送：
   - git log --oneline -1
   - git show HEAD --name-only

⚠️ 关键注意事项：
- 必须在 /Users/xuyou 仓库根目录操作
- 必须使用相对路径 git add index.html
- 提交前必须验证路径正确
```

### 示例3：不同文件名

```
更新GitHub原型：https://bdgoodbr.github.io/xuyou/lhz0105.html

请按照以下步骤执行：

1. 从URL自动提取信息：
   - URL: https://bdgoodbr.github.io/xuyou/lhz0105.html
   - 仓库名: xuyou（从URL路径提取）
   - 文件名: lhz0105.html（从URL路径提取）
   - 仓库根目录: /Users/xuyou（根据仓库名映射）

2. 确认仓库：
   - 进入仓库根目录：cd /Users/xuyou
   - 验证仓库：git rev-parse --show-toplevel
   - 验证远程：git remote -v

3. 查找源文件：
   - 优先查找：/Users/xuyou/Desktop/HTM/中远海/备份/ 目录下最新的 lhz0105 备份文件
   - 命令：find /Users/xuyou/Desktop/HTM/中远海/备份 -name "*lhz0105*" -type f -mtime -1 | sort -r | head -1

4. 同步文件：
   - 将找到的文件复制到：cp <找到的文件> /Users/xuyou/lhz0105.html

5. 验证差异：
   - cd /Users/xuyou
   - git diff lhz0105.html | head -50

6. 添加文件（关键：使用相对路径）：
   - cd /Users/xuyou
   - git add lhz0105.html
   - git diff --cached --name-only  # 验证路径应该是 lhz0105.html

7. 提交并推送：
   - git commit -m "更新原型: lhz0105.html"
   - git push origin main

8. 验证推送：
   - git log --oneline -1
   - git show HEAD --name-only

⚠️ 关键注意事项：
- 必须在 /Users/xuyou 仓库根目录操作
- 必须使用相对路径 git add lhz0105.html
- 提交前必须验证路径正确
```

## 📚 相关文档

- [GitHub Pages 自动推送技能](../skills/12-github-pages-push.md)
- [GitHub 更新注意事项](../skills/10-github-update.md)
