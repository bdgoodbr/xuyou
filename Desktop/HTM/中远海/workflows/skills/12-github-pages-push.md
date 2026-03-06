# 工作流技能：GitHub Pages 自动推送

**技能ID**: `github-pages-push`  
**技能名称**: GitHub Pages 自动推送  
**版本**: v1.0  
**状态**: ✅ 可用

## 📋 技能描述

根据用户提供的 GitHub Pages URL，自动识别文件路径、同步文件到正确仓库、提交并推送到 GitHub。

## 🎯 使用方式

### 触发命令

用户只需提供 GitHub Pages URL，例如：
- `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html`
- `https://bdgoodbr.github.io/xuyou/lhz0105.html`

### 标准提示词模板

```
更新GitHub原型：<GitHub Pages URL>

例如：
更新GitHub原型：https://bdgoodbr.github.io/xuyou/TYVISV0.1.html
```

## 🔄 自动执行流程

### 步骤1：从URL自动提取文件名和仓库信息

**URL解析规则**：
- 格式：`https://<用户名>.github.io/<仓库名>/<文件路径>`
- 提取仓库名：从URL路径中提取 `<仓库名>` 部分
- 提取文件名：
  - 如果URL以 `/` 结尾或没有文件路径：默认为 `index.html`
  - 否则：提取URL最后一部分作为文件名
- 仓库根目录映射：
  - `xuyou` → `/Users/xuyou`
  - 其他仓库：根据实际情况映射（可通过 git remote 确认）

**示例**：
```bash
# 从 https://bdgoodbr.github.io/xuyou/TYVISV0.1.html 提取：
# - 仓库名：xuyou
# - 文件名：TYVISV0.1.html
# - 仓库根目录：/Users/xuyou

# 从 https://bdgoodbr.github.io/xuyou/lhz0105.html 提取：
# - 仓库名：xuyou
# - 文件名：lhz0105.html
# - 仓库根目录：/Users/xuyou

# 从 https://bdgoodbr.github.io/xuyou/ 提取：
# - 仓库名：xuyou
# - 文件名：index.html（默认）
# - 仓库根目录：/Users/xuyou
```

### 步骤2：确认仓库根目录

```bash
cd /Users/xuyou
git rev-parse --show-toplevel  # 验证：应该输出 /Users/xuyou
git remote -v  # 验证远程地址：git@github.com:bdgoodbr/xuyou.git
```

### 步骤2.5：备份并检查 GitHub 上的当前版本（关键！）

**目的**：
1. 从 GitHub Pages 下载当前版本并备份
2. 检查本地原型目录的文件是否基于最新 GitHub 版本
3. 如果本地文件不是基于最新版本，先同步 GitHub 版本到本地原型目录

```bash
cd /Users/xuyou

# 1. 从 GitHub Pages URL 下载当前版本
backup_dir="/Users/xuyou/Desktop/HTM/中远海/备份"
mkdir -p "$backup_dir"
timestamp=$(date +%Y%m%d_%H%M%S)
backup_file="$backup_dir/TYVISV0.1.backup_${timestamp}.html"
github_url="https://bdgoodbr.github.io/xuyou/TYVISV0.1.html"

# 2. 下载 GitHub Pages 上的当前版本
curl -L "$github_url" -o "$backup_file"

# 3. 验证下载成功（下载失败则停止流程，避免基于旧本地文件误改）
if [ ! -f "$backup_file" ] || [ ! -s "$backup_file" ]; then
    echo "❌ 错误：GitHub Pages 版本下载失败，请检查URL或网络后重试（不要继续使用本地备份）。"
    exit 1
fi
echo "✅ GitHub 版本已备份到：$backup_file"

# 4. 检查本地原型目录的文件是否存在
local_prototype_file="/Users/xuyou/Desktop/HTM/中远海/原型/铁运可视化信息上报/TYVISV0.1.html"

if [ ! -f "$local_prototype_file" ]; then
    echo "📝 本地原型文件不存在，同步 GitHub 最新版本到本地原型目录..."
    mkdir -p "$(dirname "$local_prototype_file")"
    cp "$backup_file" "$local_prototype_file"
    echo "✅ 已同步 GitHub 最新版本到：$local_prototype_file"
    echo "⚠️  请在此文件基础上进行修改，然后重新执行推送命令"
    exit 0
fi

# 5. 比较本地文件与 GitHub 版本是否一致（使用文件大小和关键内容检查）
local_size=$(stat -f%z "$local_prototype_file" 2>/dev/null || stat -c%s "$local_prototype_file" 2>/dev/null)
github_size=$(stat -f%z "$backup_file" 2>/dev/null || stat -c%s "$backup_file" 2>/dev/null)
size_diff=$((local_size - github_size))
size_diff_percent=$((size_diff * 100 / github_size))

# 6. 检查关键内容是否存在（避免本地文件是旧版本）
check_key_content() {
    local file="$1"
    # 检查是否包含关键标识（根据实际文件调整）
    grep -q "new Vue\|createApp" "$file" 2>/dev/null || return 1
    grep -q "铁运可视化\|TYVIS" "$file" 2>/dev/null || return 1
    return 0
}

# 7. 判断是否需要同步
need_sync=false
sync_reason=""

if [ ${size_diff_percent#-} -gt 20 ]; then
    # 文件大小差异超过20%，可能不是基于最新版本
    need_sync=true
    sync_reason="文件大小差异过大（本地：${local_size}字节，GitHub：${github_size}字节，差异：${size_diff_percent}%）"
elif ! check_key_content "$local_prototype_file"; then
    # 缺少关键内容，可能是旧版本
    need_sync=true
    sync_reason="本地文件缺少关键内容（可能不是最新版本）"
fi

# 8. 如果需要同步，先备份本地文件，然后同步 GitHub 版本
if [ "$need_sync" = true ]; then
    echo "⚠️  检测到本地文件可能不是基于最新 GitHub 版本"
    echo "   原因：$sync_reason"
    echo ""
    echo "📝 执行同步操作："
    echo "   1. 备份当前本地文件到：${local_prototype_file}.local_backup_${timestamp}"
    cp "$local_prototype_file" "${local_prototype_file}.local_backup_${timestamp}"
    echo "   2. 同步 GitHub 最新版本到本地原型目录"
    cp "$backup_file" "$local_prototype_file"
    echo "✅ 已同步 GitHub 最新版本到：$local_prototype_file"
    echo ""
    echo "⚠️  重要提示："
    echo "   - 你的本地修改已备份到：${local_prototype_file}.local_backup_${timestamp}"
    echo "   - 本地原型文件已更新为 GitHub 最新版本"
    echo "   - 请在此最新版本基础上重新进行修改，然后重新执行推送命令"
    exit 0
else
    echo "✅ 本地文件与 GitHub 版本基本一致，继续使用本地文件进行推送"
fi
```

**说明**：
- 备份文件保存在：`/Users/xuyou/Desktop/HTM/中远海/备份/`
- 文件名格式：`<原文件名>.backup_<时间戳>.html`
- **自动同步机制**：
  - 如果本地原型文件不存在 → 自动同步 GitHub 版本
  - 如果本地文件与 GitHub 版本差异过大（大小差异>20%或缺少关键内容）→ 先备份本地文件，再同步 GitHub 版本
  - 同步后提示用户在新版本基础上修改
- **安全保护**：同步前会备份本地文件（带 `.local_backup_` 前缀），不会丢失你的修改

### 步骤3：使用本地原型目录中的文件（已确保基于最新版本）

**说明**：经过步骤2.5的检查，本地原型目录中的文件已经确保是基于最新 GitHub 版本的。

```bash
# 直接使用本地原型目录中的文件（这是你修改后的版本）
local_prototype_file="/Users/xuyou/Desktop/HTM/中远海/原型/铁运可视化信息上报/TYVISV0.1.html"

# 验证文件存在
if [ ! -f "$local_prototype_file" ]; then
    echo "❌ 错误：本地原型文件不存在，请先执行步骤2.5同步 GitHub 版本"
    exit 1
fi

echo "✅ 使用本地原型文件：$local_prototype_file"
```

### 步骤4：同步文件到仓库根目录

**说明**：将本地修改后的文件复制到仓库根目录，准备提交推送。

```bash
# 将本地修改后的文件复制到仓库根目录
# 优先使用原型目录中的文件（这是你修改后的版本）
source_file="/Users/xuyou/Desktop/HTM/中远海/原型/铁运可视化信息上报/TYVISV0.1.html"
cp "$source_file" /Users/xuyou/TYVISV0.1.html

# 如果原型目录没有，使用备份目录中最新的文件
if [ ! -f "/Users/xuyou/TYVISV0.1.html" ]; then
    latest_backup=$(find /Users/xuyou/Desktop/HTM/中远海/备份 -name "*TYVISV0.1*" -type f -mtime -1 | sort -r | head -1)
    if [ -n "$latest_backup" ]; then
        cp "$latest_backup" /Users/xuyou/TYVISV0.1.html
    fi
fi
```

### 步骤5：验证文件差异

```bash
cd /Users/xuyou
git diff TYVISV0.1.html | head -50  # 查看变更内容
```

### 步骤5.5：代码结构验证（关键！避免Vue结构错误）

**目的**：在推送前检查代码结构，避免因重复定义 Vue Options API 对象导致页面无法正常显示。

**检查规则**（仅对 HTML/Vue 文件执行）：

```bash
cd /Users/xuyou

# 1. 检查 Vue Options API 对象是否重复定义
computed_count=$(grep -c "^\s*computed:" TYVISV0.1.html)
methods_count=$(grep -c "^\s*methods:" TYVISV0.1.html)
data_count=$(grep -c "^\s*data()" TYVISV0.1.html)

# 2. 验证结果
if [ "$computed_count" -gt 1 ]; then
    echo "❌ 错误：发现 $computed_count 个 computed 对象定义（应该只有1个）"
    echo "   这会导致 Vue 实例初始化失败，所有页面无法显示"
    echo "   请修复后再推送"
    exit 1
fi

if [ "$methods_count" -gt 1 ]; then
    echo "❌ 错误：发现 $methods_count 个 methods 对象定义（应该只有1个）"
    exit 1
fi

if [ "$data_count" -gt 1 ]; then
    echo "❌ 错误：发现 $data_count 个 data() 函数定义（应该只有1个）"
    exit 1
fi

# 3. 检查 Vue 实例是否存在
if ! grep -q "new Vue\|createApp\|Vue.createApp" TYVISV0.1.html; then
    echo "⚠️  警告：未找到 Vue 实例定义，可能不是 Vue 文件，跳过结构检查"
fi

echo "✅ 代码结构验证通过"
```

**验证失败处理**：
- 如果发现重复定义，**必须停止推送流程**
- 提示用户修复问题后再执行推送
- 不执行后续的 `git add`、`git commit`、`git push` 步骤

### 步骤6：使用相对路径添加文件（关键！）

```bash
cd /Users/xuyou
git add TYVISV0.1.html  # ✅ 使用相对路径

# 验证暂存区路径
git diff --cached --name-only
# 应该输出：TYVISV0.1.html（不是 Desktop/HTM/...）
```

### 步骤7：提交并推送

```bash
cd /Users/xuyou
git commit -m "更新原型: TYVISV0.1.html - <简要说明修改点>"
git push origin main
```

### 步骤8：验证推送成功

```bash
# 查看最新提交
git log --oneline -1
git show HEAD --name-only  # 确认文件路径正确

# 验证远程同步
git ls-remote --heads origin main
```

## 📝 URL 到文件路径映射规则

### 自动映射规则

系统会根据URL自动提取信息，无需手动配置：

| GitHub Pages URL | 仓库名 | 文件名 | 仓库根目录 | 说明 |
|-----------------|--------|--------|-----------|------|
| `https://bdgoodbr.github.io/xuyou/TYVISV0.1.html` | `xuyou` | `TYVISV0.1.html` | `/Users/xuyou` | 铁运可视化原型 |
| `https://bdgoodbr.github.io/xuyou/lhz0105.html` | `xuyou` | `lhz0105.html` | `/Users/xuyou` | 系统外落货纸原型 |
| `https://bdgoodbr.github.io/xuyou/index.html` | `xuyou` | `index.html` | `/Users/xuyou` | 入口页面 |
| `https://bdgoodbr.github.io/xuyou/` | `xuyou` | `index.html` | `/Users/xuyou` | 根目录（默认index.html） |
| `https://bdgoodbr.github.io/xuyou` | `xuyou` | `index.html` | `/Users/xuyou` | 根目录（默认index.html） |

### 仓库根目录映射表

| 仓库名 | 本地仓库根目录 | 说明 |
|--------|---------------|------|
| `xuyou` | `/Users/xuyou` | 默认HTM原型仓库 |
| 其他 | 根据 `git remote` 自动识别 | 支持扩展其他仓库 |

**注意**：如果遇到未知仓库名，系统会：
1. 尝试通过 `git remote` 查找匹配的本地仓库
2. 如果找不到，提示用户确认仓库路径

## ⚠️ 关键注意事项

1. **必须在正确的仓库根目录操作**：`/Users/xuyou`（不是 `/Users/xuyou/Desktop/HTM`）
2. **使用相对路径提交**：`git add TYVISV0.1.html`（不是绝对路径）
3. **提交前验证路径**：使用 `git diff --cached --name-only` 确认路径正确
4. **确保基于最新版本修改**：
   - 步骤2.5会自动检查本地原型文件是否基于最新 GitHub 版本
   - 如果本地文件不存在或版本不一致，会自动同步 GitHub 最新版本到本地原型目录
   - 同步前会备份你的本地修改（不会丢失）
   - 只有确保本地文件基于最新版本后，才会继续推送流程
5. **代码结构验证**：对于 HTML/Vue 文件，必须检查 Vue Options API 结构（computed、methods、data 等不能重复定义），发现问题必须停止推送

## 🚨 常见错误及解决方案

### 错误1：提交到错误的仓库

**症状**：文件提交到了 `/Users/xuyou/Desktop/HTM` 仓库，而不是 `/Users/xuyou` 仓库

**解决**：
```bash
# 确认当前仓库根目录
cd /Users/xuyou
git rev-parse --show-toplevel  # 应该是 /Users/xuyou
```

### 错误2：文件路径包含子目录前缀

**症状**：提交的文件路径是 `Desktop/HTM/xuyou/TYVISV0.1.html`

**解决**：
```bash
# 撤销错误的提交
git reset HEAD Desktop/HTM/xuyou/TYVISV0.1.html

# 在正确的仓库根目录重新提交
cd /Users/xuyou
git add TYVISV0.1.html
```

### 错误3：找不到源文件

**症状**：无法找到更新后的文件

**解决**：
```bash
# 扩大搜索范围
find /Users/xuyou/Desktop/HTM -name "*TYVISV0.1*" -type f -mtime -7

# 或询问用户文件位置
```

## 📚 相关文档

- [GitHub 更新注意事项](./10-github-update.md)
- [Git 操作最佳实践](./10-github-update.md#最佳实践)

## 🔗 快速参考

### 完整命令序列（示例）

```bash
# 1. 进入仓库根目录
cd /Users/xuyou

# 2. 备份并检查 GitHub 上的当前版本（关键！）
backup_dir="/Users/xuyou/Desktop/HTM/中远海/备份"
mkdir -p "$backup_dir"
timestamp=$(date +%Y%m%d_%H%M%S)
backup_file="$backup_dir/TYVISV0.1.backup_${timestamp}.html"
github_url="https://bdgoodbr.github.io/xuyou/TYVISV0.1.html"

# 下载 GitHub 版本
curl -L "$github_url" -o "$backup_file"
if [ ! -f "$backup_file" ] || [ ! -s "$backup_file" ]; then
    echo "❌ 错误：GitHub 版本下载失败"
    exit 1
fi
echo "✅ GitHub 版本已备份到：$backup_file"

# 检查并同步本地原型文件（确保基于最新版本）
local_prototype_file="/Users/xuyou/Desktop/HTM/中远海/原型/铁运可视化信息上报/TYVISV0.1.html"
if [ ! -f "$local_prototype_file" ]; then
    echo "📝 本地原型文件不存在，同步 GitHub 最新版本..."
    mkdir -p "$(dirname "$local_prototype_file")"
    cp "$backup_file" "$local_prototype_file"
    echo "✅ 已同步，请修改后重新执行"
    exit 0
fi

# 检查版本一致性（简化版：比较文件大小）
local_size=$(stat -f%z "$local_prototype_file" 2>/dev/null || stat -c%s "$local_prototype_file" 2>/dev/null)
github_size=$(stat -f%z "$backup_file" 2>/dev/null || stat -c%s "$backup_file" 2>/dev/null)
size_diff_percent=$(( (local_size - github_size) * 100 / github_size ))
if [ ${size_diff_percent#-} -gt 20 ]; then
    echo "⚠️  本地文件可能不是基于最新版本（差异：${size_diff_percent}%）"
    echo "📝 备份本地文件并同步 GitHub 版本..."
    cp "$local_prototype_file" "${local_prototype_file}.local_backup_${timestamp}"
    cp "$backup_file" "$local_prototype_file"
    echo "✅ 已同步，请修改后重新执行"
    exit 0
fi

# 3. 同步本地修改后的文件到仓库根目录
# 使用原型目录中的文件（已确保基于最新版本）
cp "$local_prototype_file" /Users/xuyou/TYVISV0.1.html

# 4. 验证差异
git diff TYVISV0.1.html | head -30

# 5. 代码结构验证（检查 Vue Options API）
computed_count=$(grep -c "^\s*computed:" TYVISV0.1.html)
if [ "$computed_count" -gt 1 ]; then
    echo "❌ 错误：发现重复的 computed 对象定义"
    exit 1
fi

# 6. 添加文件（使用相对路径）
git add TYVISV0.1.html

# 7. 验证路径
git diff --cached --name-only

# 8. 提交并推送
git commit -m "更新原型: TYVISV0.1.html"
git push origin main

# 9. 验证推送
git log --oneline -1
git show HEAD --name-only
```
