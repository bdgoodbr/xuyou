# 工作流技能：GitHub 更新注意事项

**技能ID**: `github-update`  
**技能名称**: GitHub 更新注意事项  
**版本**: v2.1  
**状态**: ✅ 可用

## 📋 技能描述

记录在 GitHub 仓库中更新文件时的关键注意事项和最佳实践，避免常见的路径错误和提交失败问题。

## 🚨 核心规则：GitHub 原型链接自动映射（必须遵守）

> **⚠️ 关键规则**：当用户说"修改github原型"或提供原型链接时，**必须**自动提交到正确的路径。

### URL 到文件路径映射表

| GitHub Pages URL | 仓库文件路径 | Git 提交路径 | 说明 |
|-----------------|-------------|------------|------|
| `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` | `/Users/xuyou/TYVISV0.1.html` | `TYVISV0.1.html` | HTM 铁运可视化原型 |
| `https://bdgoodbr.github.io/xuyou/lhz0105.html` | `/Users/xuyou/lhz0105.html` | `lhz0105.html` | 系统外落货纸原型 |
| `https://bdgoodbr.github.io/xuyou/index.html` | `/Users/xuyou/index.html` | `index.html` | 入口页面 |

### 触发条件

当用户说以下任一内容时，**必须**按照此规则处理：

1. **"修改github原型"** / **"修改 GitHub 原型"** / **"更新原型"** / **"改原型"**
2. **提供 GitHub Pages 链接**，例如：
   - `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html`
   - `https://bdgoodbr.github.io/xuyou/lhz0105.html`
   - `https://bdgoodbr.github.io/xuyou/index.html`

### 标准处理流程（必须执行）

#### 步骤1：从链接中提取文件名
```bash
# 从 https://bdgoodbr.github.io/xuyou/TYVISV0.1.html 提取 → TYVISV0.1.html
# 从 https://bdgoodbr.github.io/xuyou/lhz0105.html 提取 → lhz0105.html
```

#### 步骤2：确认仓库根目录
```bash
cd /Users/xuyou  # 仓库根目录（必须）
git rev-parse --show-toplevel  # 验证：应该输出 /Users/xuyou
```

#### 步骤3：同步文件到仓库根目录（如果需要）
```bash
# 如果修改的是工作目录的文件，必须同步到仓库根目录
# 例如：工作目录的文件 → 仓库根目录的文件
cp /Users/xuyou/Desktop/HTM/TYVISV0.1.html /Users/xuyou/TYVISV0.1.html
```

#### 步骤4：使用正确的相对路径提交（关键！）
```bash
# ✅ 正确：从仓库根目录，使用相对路径
cd /Users/xuyou
git add TYVISV0.1.html

# ❌ 错误：不要使用绝对路径或子目录路径
# git add /Users/xuyou/Desktop/HTM/TYVISV0.1.html
# git add Desktop/HTM/TYVISV0.1.html
```

#### 步骤5：验证路径正确性
```bash
# 查看暂存区的文件路径
git diff --cached --name-only
# 应该看到：TYVISV0.1.html（不是 Desktop/HTM/... 或其他路径）

# 如果路径错误，撤销并重新添加
# git reset TYVISV0.1.html
# 然后重新执行步骤3-4
```

#### 步骤6：提交并推送
```bash
git commit -m "更新原型: TYVISV0.1.html"
git push origin main
```

#### 步骤7：验证提交成功
```bash
# 查看最新提交包含的文件路径
git show HEAD --name-only
# 应该看到：TYVISV0.1.html

# 验证文件内容是否已更新
git show HEAD:TYVISV0.1.html | grep "你的修改关键词"
```

### 快速执行示例

当用户提供链接 `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` 时：

```bash
# 1. 进入仓库根目录
cd /Users/xuyou

# 2. 同步文件（如果修改的是工作目录的文件）
cp /Users/xuyou/Desktop/HTM/TYVISV0.1.html /Users/xuyou/TYVISV0.1.html

# 3. 添加文件（使用相对路径）
git add TYVISV0.1.html

# 4. 验证路径
git diff --cached --name-only | grep TYVISV0.1.html
# 应该输出：TYVISV0.1.html

# 5. 提交并推送
git commit -m "更新原型: TYVISV0.1.html"
git push origin main
```

### 推荐：使用自动化脚本

**强烈推荐使用自动化脚本**，可以自动处理路径、备份、校验：

```bash
# 方法1：从 GitHub Pages 链接自动识别并更新（最推荐）
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/github-pages-update.sh https://bdgoodbr.github.io/xuyou/TYVISV0.1.html
# 或直接提供文件名
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/github-pages-update.sh TYVISV0.1.html

# 方法2：使用安全路径检查脚本
cd /Users/xuyou
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/git-add-safe.sh /Users/xuyou/Desktop/HTM/TYVISV0.1.html

# 方法3：使用完整的安全更新脚本
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/github-prototype-update.sh TYVISV0.1.html
```

### 常见 GitHub Pages 文件映射

| GitHub Pages 链接 | 仓库中的文件路径 | 说明 |
|------------------|----------------|------|
| `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` | `/Users/xuyou/TYVISV0.1.html` | 铁运可视化原型 |
| `https://bdgoodbr.github.io/xuyou/lhz0105.html` | `/Users/xuyou/lhz0105.html` | 落货纸原型 |
| `https://bdgoodbr.github.io/xuyou/index.html` | `/Users/xuyou/index.html` | 入口页面 |

### ⚠️ 关键注意事项

- **仓库根目录**：`/Users/xuyou`（不是 `/Users/xuyou/Desktop/HTM`）
- **GitHub Pages 文件位置**：仓库根目录下的文件（如 `TYVISV0.1.html`）
- **不要提交到错误路径**：避免提交到 `Desktop/HTM/...` 等子目录路径
- **提交前验证**：使用 `git diff --cached --name-only` 确认路径正确

## 🎯 标准流程：GitHub 原型文件更新（必须遵循）

> **重要**：所有在 GitHub 上修改原型文件的操作，都必须遵循以下标准流程，确保安全性和可追溯性。

### 流程概述

1. **从 GitHub 下载最新版本** → 确保基于最新代码修改
2. **备份到备份文件夹** → 防止意外丢失，便于回滚
3. **修改文件** → 在最新版本基础上进行修改
4. **提交并推送到 GitHub** → 更新到远程仓库
5. **（新增强制项）推送前备份“远端最新版本” + 关键内容校验** → 防止“本地旧版本覆盖远端较新页面”

### 方法一：使用自动化脚本（推荐）

我们提供了一个自动化脚本，可以一键完成整个流程：

```bash
# 使用脚本更新原型文件
cd /Users/xuyou/Desktop/HTM/中远海/workflows/scripts
./github-prototype-update.sh
# 或只更新其中几份：
# ./github-prototype-update.sh TYVISV0.1.html index.html
```

**脚本功能**：

- ✅ `git fetch` 获取远端最新版本（origin/main）
- ✅ **推送前强制备份“远端最新版本”**到 `/Users/xuyou/Desktop/HTM/中远海/备份`
- ✅ 备份文件命名：`文件名_YYYYMMDD_HHMMSS_remote.html`（明确这是远端快照）
- ✅ **关键内容校验**：缺少关键页面/入口链接会阻断 push（避免覆盖掉“可视化查询页”等）
- ✅ 自动提交并推送到 GitHub
- ✅ 显示 GitHub Pages 访问地址

### （强烈推荐）安装 pre-push 守卫：任何 git push 都先备份再校验

> 这一步是“彻底杜绝事故”的关键：以后即便你不走脚本、直接 `git push`，也会被 hook 自动拦截。

```bash
cd /Users/xuyou/Desktop/HTM/中远海/workflows/scripts
./install-prototype-prepush-hook.sh
```

**守卫会做什么**：

- 在 push 前自动 `git fetch origin main`
- 自动备份三份原型的远端最新版本到备份目录（带 `_remote` 时间戳）
- 校验三份原型关键内容（任意命中其一即可）：
  - `TYVISV0.1.html`：必须包含「铁运可视化查询 / 铁运可视化信息上报 / 铁运可视化（上报 + 查询）」之一（避免“查询页被覆盖掉”）
  - `lhz0105.html`：必须包含「window.APP_VERSION / 系统外落货纸 Living Spec / Living Spec」之一（避免空壳覆盖）
  - `index.html`：必须包含「铁运可视化（上报 + 查询）/ iron-visualization.html / TYVISV0.1.html」之一（避免入口页回退/丢入口）
  - 校验失败：**直接阻断 push**，防止误覆盖

### 方法二：手动执行标准流程

如果不想使用脚本，可以手动执行以下步骤：

#### 步骤1：切换到仓库根目录并拉取最新版本（三份原型都拉）

```bash
cd /Users/xuyou
git fetch origin main
git checkout origin/main -- TYVISV0.1.html lhz0105.html index.html
```

#### 步骤2：备份当前版本（三份原型都备份）

```bash
# 创建备份文件名（带时间戳）
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_DIR="/Users/xuyou/Desktop/HTM/中远海/备份"
mkdir -p "$BACKUP_DIR"

# 备份文件
cp /Users/xuyou/TYVISV0.1.html "$BACKUP_DIR/TYVISV0.1_${TIMESTAMP}.html"
cp /Users/xuyou/lhz0105.html "$BACKUP_DIR/lhz0105_${TIMESTAMP}.html"
cp /Users/xuyou/index.html "$BACKUP_DIR/index_${TIMESTAMP}.html"
echo "✓ 备份完成: $BACKUP_DIR/*_${TIMESTAMP}.html"
```

#### 步骤3：修改文件（在最新版本基础上改）

使用编辑器修改：
- `/Users/xuyou/TYVISV0.1.html`
- `/Users/xuyou/lhz0105.html`
- `/Users/xuyou/index.html`

#### 步骤4：提交并推送（三份文件一起提交）

```bash
cd /Users/xuyou
git add TYVISV0.1.html lhz0105.html index.html
git commit -m "更新原型文件：<修改说明>"
git push origin main
```

### 流程检查清单

在每次修改 GitHub 原型文件前，请确认：

- [ ] ✅ 已从 GitHub 拉取最新版本
- [ ] ✅ 已备份到 `/Users/xuyou/Desktop/HTM/中远海/备份` 文件夹
- [ ] ✅ 备份文件名包含时间戳（格式：`文件名_YYYYMMDD_HHMMSS.html`）
- [ ] ✅ 在最新版本基础上进行修改
- [ ] ✅ 提交信息清晰描述修改内容
- [ ] ✅ 已推送到 GitHub
- [ ] ✅ 已验证 GitHub Pages 更新（可能需要等待几分钟）

### 备份文件命名规范

备份文件必须遵循以下命名规范，便于管理和查找：

```text
<原文件名>_<时间戳>.html
```

**示例**：
- `TYVISV0.1_20260204_143022.html`
- `TYVISV0.2_20260204_150530.html`

**时间戳格式**：`YYYYMMDD_HHMMSS`（年月日_时分秒）

### 为什么需要这个流程？

1. **安全性**：备份可以防止意外覆盖或丢失代码
2. **可追溯性**：每次修改都有备份，可以随时回滚到任意版本
3. **一致性**：确保始终基于最新版本修改，避免冲突
4. **规范性**：统一的流程减少错误，提高效率

### 常见问题

**Q: 如果忘记备份怎么办？**  
A: 可以从 Git 历史中恢复：`git checkout <commit-hash> -- TYVISV0.1.html`，但建议养成备份习惯。

**Q: 备份文件太多怎么办？**  
A: 可以定期清理旧备份，但建议至少保留最近 10 个版本的备份。

**Q: 可以使用脚本更新其他文件吗？**  
A: 可以，脚本支持任何在 GitHub 仓库中的文件，只需修改文件名参数即可。

## ⚠️ 核心问题：git add 路径错误

### 问题现象

在更新 GitHub 仓库中的文件时，即使：
- ✅ 本地文件已正确修改
- ✅ 执行了 `git add` 和 `git commit`
- ✅ 执行了 `git push --force`

但 GitHub 上的文件仍然**没有更新**，显示的内容仍然是旧版本。

### 根本原因

**`git add` 时使用了错误的文件路径**，导致 Git 将文件视为新文件而不是更新现有文件。

#### 错误示例

```bash
# ❌ 错误1：使用带引号的绝对路径
git add "Desktop/HTM/中远海/归档文档/GitHub备份/github/xuyou/TYVISV0.1.html"

# ❌ 错误2：从错误的目录执行 git add
cd /Users/xuyou/Desktop/HTM
git add 中远海/归档文档/GitHub备份/github/xuyou/TYVISV0.1.html

# ❌ 错误3：路径中包含特殊字符未正确处理
git add Desktop/HTM/中远海/归档文档/GitHub备份/github/xuyou/TYVISV0.1.html
```

**问题分析**：
- Git 会将错误路径的文件视为**新文件**添加到仓库
- 即使提交成功，仓库中的**原始文件路径**并未被更新
- 强制推送也无法解决，因为推送的是错误路径的提交

#### 正确做法

```bash
# ✅ 方法1：在仓库根目录使用相对路径（推荐）
cd /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github
git add xuyou/TYVISV0.1.html
git commit -m "更新文件"
git push

# ✅ 方法2：使用正确的绝对路径（确保路径准确）
git add /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github/xuyou/TYVISV0.1.html
git commit -m "更新文件"
git push

# ✅ 方法3：先检查文件在仓库中的实际路径
cd /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github
git ls-files | grep TYVISV0.1.html  # 查看文件在仓库中的路径
git add xuyou/TYVISV0.1.html  # 使用正确的相对路径
```

## 🔍 验证步骤

### 1. 确认仓库根目录

```bash
# 找到 Git 仓库根目录
cd /path/to/your/repo
git rev-parse --show-toplevel
```

### 2. 检查文件在仓库中的路径

```bash
# 查看文件在 Git 仓库中的实际路径
git ls-files | grep filename

# 或查看特定文件
git ls-files xuyou/TYVISV0.1.html
```

### 3. 提交前验证

```bash
# 查看将要提交的文件路径
git status

# 查看暂存区的文件
git diff --cached --name-only

# 确认文件路径正确
git diff --cached --name-only | grep TYVISV0.1.html
```

### 4. 提交后验证

```bash
# 查看最新提交包含的文件
git show HEAD --name-only

# 查看文件内容是否已更新
git show HEAD:xuyou/TYVISV0.1.html | grep "集装箱号"
```

## 📝 标准操作流程

### 步骤1：定位仓库根目录

> ⚠️ 当前实际仓库根目录是 **`/Users/xuyou`**，不是某个子目录（例如 `.../GitHub备份/github`）。

```bash
cd /Users/xuyou
git rev-parse --show-toplevel  # 应该输出：/Users/xuyou
```

### 步骤2：确认文件路径

```bash
# 查看文件在仓库中的路径（用于确认）
git ls-files | grep TYVISV0.1.html

# 关键结论：
# 1）用于 GitHub Pages 的原型文件路径：TYVISV0.1.html（仓库根目录下）
# 2）xuyou/TYVISV0.1.html 更像是备份/历史文件，不能只改这一份
```

### 步骤3：修改文件

```bash
# 使用编辑器或工具修改文件
# 或使用 sed 批量替换
sed -i '' 's/箱号/集装箱号/g' xuyou/TYVISV0.1.html
```

### 步骤4：使用正确的路径添加文件

```bash
# 使用相对路径（从仓库根目录）
git add xuyou/TYVISV0.1.html

# 验证暂存区
git status
```

### 步骤5：提交更改

```bash
git commit -m "更新：修改 HTM 原型 TYVISV0.1.html"
```

### 步骤6：推送到 GitHub（原型）

```bash
git push
```

## 🛡️ 解决方案：防止 git add 路径错误

### 方案1：使用自动化脚本（推荐）

**使用现有的安全更新脚本：**
```bash
# 自动处理路径、备份、校验
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/github-prototype-update.sh TYVISV0.1.html
```

**使用新的安全路径检查脚本：**
```bash
# 自动检查并修正路径
cd /Users/xuyou
/Users/xuyou/Desktop/HTM/中远海/workflows/scripts/git-add-safe.sh /Users/xuyou/Desktop/HTM/TYVISV0.1.html
```

### 方案2：手动验证路径（标准流程）

```bash
# 1. 确认仓库根目录
cd /Users/xuyou
git rev-parse --show-toplevel  # 应该输出：/Users/xuyou

# 2. 查看文件在仓库中的实际路径
git ls-files | grep TYVISV0.1.html

# 3. 确认 GitHub Pages 使用的文件路径（仓库根目录）
# 输出应该包含：TYVISV0.1.html（不是 Desktop/HTM/...）

# 4. 同步文件到仓库根目录（如果需要）
cp /Users/xuyou/Desktop/HTM/TYVISV0.1.html /Users/xuyou/TYVISV0.1.html

# 5. 使用相对路径添加（从仓库根目录）
git add TYVISV0.1.html

# 6. 验证暂存区路径
git diff --cached --name-only | grep TYVISV0.1.html
# 应该输出：TYVISV0.1.html（不是其他路径）

# 7. 提交并推送
git commit -m "更新: TYVISV0.1.html"
git push origin main
```

### 方案3：清理重复文件（可选）

如果仓库中有多个同名文件，可以考虑清理：

```bash
# 查看所有 TYVISV0.1.html 文件
git ls-files | grep TYVISV0.1.html

# 删除不需要的文件（谨慎操作！）
# git rm Desktop/HTM/中远海/归档文档/GitHub备份/github/xuyou/TYVISV0.1.html
# git commit -m "清理：删除重复的 TYVISV0.1.html"
```

### 验证提交是否正确

```bash
# 查看最新提交包含的文件路径
git show HEAD --name-only

# 确认文件路径是 TYVISV0.1.html（不是其他路径）
git show HEAD --name-only | grep TYVISV0.1.html
# 应该输出：TYVISV0.1.html

# 查看文件内容是否已更新
git show HEAD:TYVISV0.1.html | grep "我的下载"
```

# 如果需要强制推送（谨慎使用）
git push --force
```

### 步骤7：验证更新（原型）

```bash
# 1）从 GitHub 仓库拉取原型文件（用于内容校验）
curl -s "https://raw.githubusercontent.com/bdgoodbr/xuyou/main/TYVISV0.1.html" | head

# 2）访问 GitHub Pages 上的原型页面（浏览器打开）
#    正确页面地址：
#    https://bdgoodbr.github.io/xuyou/TYVISV0.1.html
#
# 3）必要时使用无痕窗口或强制刷新（Cmd+Shift+R）绕过浏览器缓存
```

## 🎯 针对“修改 HTM 原型 TYVISV0.1.html”的特别流程

> 只要用户说：“**要修改原型** / **改 HTM 页面**”，默认指的就是
> `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` 这一页，
> 必须按照下面这一套来操作。

1. **总是从正确仓库根目录开始**

```bash
cd /Users/xuyou
git rev-parse --show-toplevel  # 应该是 /Users/xuyou
```

2. **只改对的那份文件**

- 用编辑器修改：`/Users/xuyou/TYVISV0.1.html`
- 如需备份，可以同步更新 `xuyou/TYVISV0.1.html`，但：
  - **GitHub Pages 实际渲染的是仓库根目录的 `TYVISV0.1.html`**
  - 不能只改 `xuyou/TYVISV0.1.html` 否则网页不会变

3. **提交和推送命令（固定模板，可复用）**

```bash
cd /Users/xuyou

# 查看差异
git diff TYVISV0.1.html

# 添加并提交
git add TYVISV0.1.html
git commit -m "更新 HTM 原型 TYVISV0.1.html：<简要说明修改点>"
git push
```

4. **固定验证网址**

- **GitHub 仓库文件（用于看源码）**  
  `https://github.com/bdgoodbr/xuyou/blob/main/TYVISV0.1.html`
- **GitHub Pages 原型页面（给业务/用户看的页面）**  
  `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html`

> 以后只要有“修改原型”的需求，直接沿用这套路径和网址，不要再去动
> `.../GitHub备份/github` 下面的镜像路径。

## 🎯 最佳实践

### 1. 始终在仓库根目录操作

```bash
# ✅ 推荐：先 cd 到仓库根目录
cd /Users/xuyou
git add relative/path/to/file

# ❌ 避免：从子目录或备份目录中操作 Git
cd /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github  # 这里不要再当成真正仓库根目录
```

### 2. 使用相对路径

```bash
# ✅ 推荐：在 /Users/xuyou 下使用相对路径
git add TYVISV0.1.html

# ❌ 避免：跨目录长绝对路径（容易指到备份目录）
git add /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github/TYVISV0.1.html
```

### 3. 提交前检查

```bash
# 提交前务必检查
git status
git diff --cached --name-only
```

### 4. 验证提交内容

```bash
# 查看提交包含的文件和路径
git show HEAD --stat
git show HEAD --name-only
```

### 5. 使用脚本自动化

```bash
#!/bin/bash
# update_github_file.sh

REPO_ROOT="/Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github"
FILE_PATH="xuyou/TYVISV0.1.html"

cd "$REPO_ROOT" || exit 1

# 修改文件
# ... 你的修改操作 ...

# 使用相对路径添加
git add "$FILE_PATH"

# 检查状态
git status

# 提交
git commit -m "更新文件"

# 推送
git push
```

## 🚨 常见错误及解决方案

### 错误1：文件路径包含引号

**错误**：
```bash
git add "xuyou/TYVISV0.1.html"  # 引号会被包含在路径中
```

**解决**：
```bash
git add xuyou/TYVISV0.1.html  # 去掉引号
```

### 错误2：从错误目录执行（把备份目录当成仓库根目录）

**错误**：
```bash
cd /Users/xuyou/Desktop/HTM/中远海/归档文档/GitHub备份/github
git add TYVISV0.1.html  # 实际上这不是仓库真正根目录下的文件
```

**解决**：
```bash
cd /Users/xuyou
git add TYVISV0.1.html  # 在真正仓库根目录下操作
```

### 错误3：路径不匹配

**错误**：
```bash
# 仓库中的文件路径是 xuyou/TYVISV0.1.html
# 但使用了错误的路径
git add TYVISV0.1.html  # 缺少 xuyou/ 前缀
```

**解决**：
```bash
# 先查看正确的路径
git ls-files | grep TYVISV0.1.html
# 然后使用正确的路径
git add xuyou/TYVISV0.1.html
```

## 🔗 快速参考：URL 到文件路径映射

### 自动识别规则

当用户提供以下格式的输入时，自动识别并映射：

| 用户输入示例 | 识别到的文件名 | 仓库路径 |
|------------|--------------|---------|
| `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` | `TYVISV0.1.html` | `/Users/xuyou/TYVISV0.1.html` |
| `https://bdgoodbr.github.io/xuyou/lhz0105.html` | `lhz0105.html` | `/Users/xuyou/lhz0105.html` |
| `https://bdgoodbr.github.io/xuyou/index.html` | `index.html` | `/Users/xuyou/index.html` |
| "修改github原型 TYVISV0.1" | `TYVISV0.1.html` | `/Users/xuyou/TYVISV0.1.html` |
| "改HTM页面" | `TYVISV0.1.html` | `/Users/xuyou/TYVISV0.1.html` |
| "改原型" | 需要用户确认具体文件 | - |

### 标准提交命令模板

```bash
# 1. 进入仓库根目录
cd /Users/xuyou

# 2. 同步文件（如果修改的是工作目录文件）
cp /Users/xuyou/Desktop/HTM/TYVISV0.1.html /Users/xuyou/TYVISV0.1.html

# 3. 使用相对路径添加（关键！）
git add TYVISV0.1.html

# 4. 验证路径
git diff --cached --name-only
# 应该输出：TYVISV0.1.html

# 5. 提交并推送
git commit -m "更新: TYVISV0.1.html"
git push origin main
```

## 📚 相关文档

- [Git 官方文档](https://git-scm.com/doc)
- [GitHub 文档](https://docs.github.com)

## 🔗 关联技能

- **文件批量替换** (`file-replace`): 批量替换文件内容
- **Git 操作** (`git-operations`): 其他 Git 相关操作

## ⚙️ 配置参数

无特殊配置参数。

## 📝 注意事项

1. **路径一致性**：确保 `git add` 使用的路径与仓库中文件的路径完全一致
2. **仓库根目录**：始终从仓库根目录执行 `git add`，使用相对路径
3. **提交前验证**：使用 `git status` 和 `git diff --cached` 验证将要提交的文件
4. **推送后验证**：推送后等待几秒，从 GitHub 验证文件是否已更新
5. **强制推送谨慎**：只有在确认需要覆盖远程历史时才使用 `git push --force`

## 🚨 重要提醒：避免在错误目录操作

### 问题场景
当用户说"修改原型"或"更新 GitHub"时，**必须确认正确的文件路径**：

1. **GitHub Pages 使用的文件**：`/Users/xuyou/TYVISV0.1.html`（仓库根目录）
2. **工作目录中的文件**：`/Users/xuyou/Desktop/HTM/TYVISV0.1.html`（仅用于本地编辑）

### 预防措施

**在修改文件前，必须执行以下检查：**

```bash
# 1. 确认仓库根目录
cd /Users/xuyou
git rev-parse --show-toplevel  # 应该输出：/Users/xuyou

# 2. 确认 GitHub Pages 使用的文件路径
git ls-files | grep TYVISV0.1.html
# 应该看到：TYVISV0.1.html（仓库根目录下的文件）

# 3. 如果修改了工作目录的文件，必须同步到仓库根目录
cp /Users/xuyou/Desktop/HTM/TYVISV0.1.html /Users/xuyou/TYVISV0.1.html

# 4. 从正确的仓库根目录提交
cd /Users/xuyou
git add TYVISV0.1.html
git commit -m "更新说明"
git push
```

### 标准操作流程（必须遵循）

**每次修改原型文件时，按以下步骤操作：**

1. **修改文件**：在 `/Users/xuyou/Desktop/HTM/TYVISV0.1.html` 进行编辑
2. **同步文件**：将修改同步到 `/Users/xuyou/TYVISV0.1.html`
3. **验证差异**：`cd /Users/xuyou && git diff TYVISV0.1.html`
4. **提交推送**：从 `/Users/xuyou` 目录执行 `git add`、`git commit`、`git push`
5. **验证更新**：访问 `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` 确认更新

## 📅 更新记录

- **v2.1** (2026-02-05): 
  - 添加核心规则：GitHub 原型链接自动映射
  - 当用户说"修改github原型"或提供原型链接时，自动识别并提交到正确路径
  - 创建 URL 到文件路径映射表和快速参考
  - 新增工具脚本：`github-url-to-path.sh`（自动识别 URL 并映射到文件路径）
  - 新增工具脚本：`git-add-safe.sh`（安全路径检查脚本）

- **v2.0** (2026-02-04): 
  - 添加标准流程：GitHub 原型文件更新流程（必须遵循）
  - 提供自动化脚本 `github-prototype-update.sh`
  - 明确备份要求和命名规范
  - 强调每次修改前必须下载最新版本并备份
- **v1.2** (2026-02-04): 添加预防措施和标准操作流程，明确区分工作目录文件和 GitHub Pages 文件，避免在错误目录操作
- **v1.1** (2026-02-02): 补充"修改 HTM 原型 TYVISV0.1.html"的专用流程，明确实际仓库根目录为 `/Users/xuyou`，以及 GitHub/GitHub Pages 的正确网址和文件路径
- **v1.0** (2026-01-30): 初始版本，记录 git add 路径错误的根本原因和解决方案

---

**创建时间**：2026-01-30  
**最后更新**：2026-02-04  
**状态**：✅ 可用
