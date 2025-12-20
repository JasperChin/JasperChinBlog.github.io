# Hugo博客从初始化到GitHub Pages部署完整指南
## 一、准备工作
### 1. 安装Hugo（macOS系统，适配你的本地路径）
```bash
# 方式1：通过Homebrew安装（推荐，自动适配arm64架构）
brew install hugo

# 验证安装成功（查看版本，需≥0.150）
hugo version
# 预期输出：hugo v0.152.2+extended+withdeploy darwin/arm64 ...

# 方式2：手动下载（若brew安装失败）
# 访问 https://github.com/gohugoio/hugo/releases 下载对应macOS版本
# 解压后将hugo可执行文件放到 /usr/local/bin 目录，再执行 hugo version 验证
```

### 2. 配置GitHub SSH（避免HTTPS认证失败）
```bash
# 检查本地是否已有SSH密钥
ls -la ~/.ssh

# 若无密钥，生成新SSH密钥（替换为你的GitHub邮箱）
ssh-keygen -t ed25519 -C "JasperChinChiaoHsin@gmail.com"

# 将SSH密钥添加到ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 复制SSH公钥（粘贴到GitHub账户的SSH设置中）
pbcopy < ~/.ssh/id_ed25519.pub
```
> 补充：GitHub添加SSH公钥路径：Settings → SSH and GPG keys → New SSH key → 粘贴公钥并保存。

## 二、初始化Hugo项目（适配你的本地路径）
```bash
# 进入你的代码根目录（固定路径）
cd /Users/jasper/学习/code/go

# 初始化Hugo项目（仓库名与GitHub一致）
hugo new site JasperChinBlog.github.io

# 进入项目根目录（后续所有操作均在此目录执行）
cd /Users/jasper/学习/code/go/JasperChinBlog.github.io
```

## 三、安装主题（推荐Hugo官方Ananke，稳定无坑）
```bash
# 克隆Ananke主题到themes目录（官方维护，零配置启动）
git clone https://github.com/theNewDynamic/gohugo-theme-ananke.git themes/ananke

# 删除主题自带.git目录（避免子模块冲突，当作普通文件提交）
rm -rf themes/ananke/.git
```

## 四、核心配置（hugo.toml）
替换 `/Users/jasper/学习/code/go/JasperChinBlog.github.io/hugo.toml` 文件内容：
```toml
# 项目级GitHub Pages地址（格式：https://用户名.github.io/仓库名/）
baseURL = "https://JasperChin.github.io/JasperChinBlog.github.io/"
languageCode = "zh-CN"  # 中文站点配置
title = "我的博客"       # 自定义博客标题
theme = "ananke"        # 主题名（与themes目录下文件夹名一致）
```

## 五、创建第一篇文章
```bash
# 在当前项目根目录执行，创建文章（自动生成到content/posts目录）
hugo new content/posts/我的第一篇预览文章.md

# 关键操作：编辑文章文件，将draft改为false（否则预览/部署不显示）
# 编辑路径：/Users/jasper/学习/code/go/JasperChinBlog.github.io/content/posts/我的第一篇预览文章.md
# 修改内容：draft = false
```

## 六、本地预览（验证效果）
```bash
# 在项目根目录启动Hugo预览服务
hugo server

# 访问本地地址（需与baseURL对应，避免路径错误）
# 浏览器打开：http://localhost:1313/JasperChinBlog.github.io/
```

## 七、部署到GitHub Pages
### 步骤1：关联GitHub仓库
```bash
# 在项目根目录初始化本地Git仓库
git init

# 关联GitHub仓库（替换为你的仓库SSH地址）
git remote add origin git@github.com:JasperChin/JasperChinBlog.github.io.git
```

### 步骤2：创建部署配置文件
```bash
# 在项目根目录创建GitHub Actions目录（自动触发部署）
mkdir -p .github/workflows

# 新建部署配置文件
touch .github/workflows/deploy.yml
```

将以下内容写入 `/Users/jasper/学习/code/go/JasperChinBlog.github.io/.github/workflows/deploy.yml`：
```yaml
name: 部署博客
on:
  push:
    branches: [ main ]  # 推送到main分支时自动触发部署
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: 拉取仓库代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 拉取完整提交历史，避免编译异常

      - name: 安装Hugo（与本地版本一致，避免编译差异）
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.152.2'
          extended: true

      - name: 编译静态网页（压缩HTML/CSS，提升加载速度）
        run: hugo --minify

      - name: 部署到GitHub Pages
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public  # 编译后的静态文件目录
```

### 步骤3：提交并推送代码
```bash
# 在项目根目录创建.gitignore，忽略Hugo构建产物和系统临时文件
echo -e "public/\nresources/\n.hugo_build.lock\n.DS_Store\nThumbs.db\n.vscode/\n.idea/" > .gitignore

# 暂存所有文件（配置+主题+文章+部署脚本）
git add .

# 提交代码
git commit -m "初始化Hugo博客+配置自动部署"

# 推送到GitHub（SSH自动认证，无需输入密码）
git push -u origin main
```

## 八、GitHub Pages网页端配置
1. 打开GitHub仓库 → 点击顶部`Settings` → 左侧导航栏找到`Pages`；
2. 在`Source`区域配置：
   - 选择`Deploy from a branch`；
   - `Branch`下拉框选择`gh-pages`；
   - 目录保持`/(root)`；
3. 点击`Save`保存配置；
4. 将仓库改为`Public`（免费GitHub账户仅支持公开仓库启用Pages功能）。

## 九、验证部署结果
部署完成后（GitHub Actions显示绿色对勾），等待1-2分钟（生效延迟），访问最终博客地址：
`https://JasperChin.github.io/JasperChinBlog.github.io/`

## 关键避坑总结
1. 路径统一：所有操作均在 `/Users/jasper/学习/code/go/JasperChinBlog.github.io` 目录执行，避免路径错乱；
2. 主题选择：优先使用Hugo官方或维护活跃的主题（如Ananke），避免小众主题的配置兼容问题；
3. baseURL格式：项目级仓库必须遵循`https://用户名.github.io/仓库名/`，否则本地预览和部署后路径错乱；
4. 认证方式：GitHub已禁用密码认证，必须配置SSH或Personal Access Token（PAT）；
5. 文章显示：`draft = false`是文章在预览/部署中显示的必要条件，默认`true`（草稿状态）不显示；
6. 仓库权限：免费GitHub账户需将仓库设为Public，私有仓库启用Pages需付费升级；
7. 主题冲突：下载主题后必须删除自带的`.git`目录，避免Git子模块嵌套问题。

---

### 使用说明
1. 复制上述完整内容，保存为`Hugo博客部署指南.md`文件；
2. 后续新增文章：在 `/Users/jasper/学习/code/go/JasperChinBlog.github.io` 目录执行`hugo new content/posts/新文章.md`，编辑后将`draft`改为`false`，提交推送即可自动部署；
3. 本地调试：修改配置/主题后，在项目根目录执行`hugo server`预览效果，确认无误再推送。