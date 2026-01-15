# 环境要求

- 在线文档网站效果：https://chcnavdev.github.io
- 文档演示网站部署效果：https://chcnavdev.github.io/doc_site_template
- 部署本地环境要求：python、git
- 部署远程环境要求：github账号
- 维护人环境要求：仓库权限（**最大的优点就在此，与环境无关，维护人不需要关心任何环境，直接将仓库clone到本地，文档任何更新直接push即可发布**）

网站主要是基于[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)定制搭建，定制是为了适配华测官方网站颜色主题，网站基于Github的Git Pages服务部署，基于`GitHub Actions`实现自动同步和多语言适配切换。下面主要分为几个步骤介绍：

1. 跑通本地环境（一般基于python的pip安装个包就好了）
2. 实现多语言（单语言怎么实现，多语言就怎么实现，文档翻译一下就好了）
3. 如何实现远程自动部署同步（git pages建个仓库）
4. 如何实现主题色（基于官网css色彩配一下）

---

# 跑通本地环境
首先需要准备`python`环境，然后通过`pip`安装`Material for MkDocs`即可。
```bash
pip install mkdocs-material
```
> 建议单独建个虚拟的`python`环境。Mac Apple Silicon可以考虑用[Miniforge](https://github.com/conda-forge/miniforge)

依赖安装好后，就可以先本地新建一个网站，看看是否正常，能不能跑得通。
```shell
~ >> cd Desktop # 这里我在桌面路径下

~/Desktop >> mkdir testSite # 新建一个测试目录

~/Desktop >> cd testSite # 到测试目录下
 
~/Desktop/testSite >> mkdocs new ./zh # 基于当前目录创建本地网站zh

~/Desktop/testSite >> cd zh # 到本地网站目录下  

~/Desktop/testSite >> mkdocs serve # 将本地网站服务启动
```
> `>>` 符号后为命令

以下为启动服务后的输出：
```bash
INFO    -  Building documentation...
INFO    -  Cleaning site directory
INFO    -  Documentation built in 0.06 seconds
INFO    -  [19:11:25] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO    -  [19:11:25] Serving on http://127.0.0.1:8000/
```
其中`mkdocs.yml`为配置文件，多语言、导航、主题、站点url、站点标题等都在此配置。`docs`为`md`文档撰写的地方。

可以试着在`mkdocs.yml`中添加即将要配的主题，然后再看下效果：
```yml
site_name: My Docs
theme:
  name: material
```

修改完运行后，主题上会和第一版有很大的差别。到此都没问题的话，本地环境算是配好了，也就是`pip install mkdocs-material`安装个库而已。

---
# 实现多语言

实现多语言也就是再新建一个对应翻译语言的站点，然后修改一下配置文件即可。

## 多语言文档结构建立

目前`testSite`目录如下结构：
```bash
~/Desktop/testSite >> tree
.
└── zh
    ├── docs
    │   └── index.md
    └── mkdocs.yml

3 directories, 2 files
```

假设需要一个英文站点，所以可以再创建一个英文站点目录：
```bash
~/Desktop/testSite >> mkdocs new ./en
INFO    -  Creating project directory: ./en
INFO    -  Writing config file: ./en/mkdocs.yml
INFO    -  Writing initial docs: ./en/docs/index.md
```
创建完目录如下：
```bash
~/Desktop/testSite >> tree
.
├── en
│   ├── docs
│   │   └── index.md
│   └── mkdocs.yml
└── zh
    ├── docs
    │   └── index.md
    └── mkdocs.yml

5 directories, 4 files
```

然后把`index.md`改成对应的中英文内容，双语文档就算完成了，后面可以在配置文件配置跳转功能。

## 多语言跳转配置
多语言跳转需要分别为不同语言做下差异性配置，中英文站点配置分别如下：
```yml
# ------- 中文站点 -------
site_name: My Docs

# 主题配置
theme:
  name: material
  # 设置站点语言为中文
  language: zh

# 额外配置
extra:
  # 多语言切换配置
  alternate:
    - name: English
      link: ../en/
      lang: en
      

# ------- 英文站点 -------
site_name: My Docs

# 主题配置
theme:
  name: material
  # 设置站点语言为英文
  language: en

# 额外配置
extra:
  # 多语言切换配置
  alternate:
    - name: 中文
      link: ../zh/
      lang: zh
```

配置完分别将中英文两个站点都单独运行一下，能运行成功就行。
> 注意，这里同时运行两个服务会产生端口占用的问题，运行的时候最好指定一下端口，命令：`mkdocs serve --dev-addr <IP:PORT>`，不过这点不用担心，后面远程配置会解决这个问题。


---

# 文档发布

文档发布是基于Github的Git Pages功能做的，它提供免费的静态网站部署服务。Github同时还提供Git Actions服务，这样每次push文档变更的时候，就可以自动发布了。这样有一个很大的好处，**后续任何人接手维护文档，不需要配置任何环境，直接将仓库的md文档clone下来，任何变更修改后直接push就好了。**


## GitHub Actions 工作流
这一步主要是为了在代码推送到 main 分支或提交 PR 时，自动构建中英文两个 MkDocs 文档，自动识别浏览器语言，并部署到 GitHub Pages。

新建一个工作流需要在当前项目目录下新建`.github/workflows`目录，并在该目录下新建对应工作流的`.yml`文件，当前项目新建完目录如下：
```bash
~/Desktop/testSite >> tree -a
.
├── .github
│   └── workflows
│       └── deploy.yml
├── en
│   ├── docs
│   │   └── index.md
│   └── mkdocs.yml
└── zh
    ├── docs
    │   └── index.md
    └── mkdocs.yml
```

其中`deploy.yml`目录内容如下，可直接复制使用：
```yml
# 工作流名称（显示在 GitHub Actions 页面）
name: Deploy Multi-language MkDocs to GitHub Pages

# 触发条件
on:
  # 当代码 push 到 main 分支时触发（正式部署）
  push:
    branches: [ "main" ]

  # 当向 main 分支提交 Pull Request 时触发（用于构建校验）
  pull_request:
    branches: [ "main" ]

# GitHub Actions 运行时所需的最小权限
permissions:
  contents: read        # 读取仓库内容
  pages: write          # 写入 GitHub Pages
  id-token: write       # 用于 Pages 的安全身份认证（OIDC）

# 并发控制，防止多个 Pages 部署同时运行
concurrency:
  # 同一个仓库只允许一个 Pages 部署任务
  group: pages-${{ github.repository }}
  # 如果有新的部署任务进来，取消正在运行的旧任务
  cancel-in-progress: true

jobs:
  # 定义一个名为 build 的 job
  build:
    # 使用 GitHub 提供的最新 Ubuntu 运行环境
    runs-on: ubuntu-latest

    steps:
    # Step 1：拉取仓库代码
    - name: Checkout repository
      uses: actions/checkout@v4

    # Step 2：安装 Python 环境
    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        # 使用 Python 3.x 最新版本
        python-version: '3.x'

    # Step 3：安装 MkDocs 及 Material 主题
    - name: Install dependencies
      run: |
        pip install mkdocs-material

    # Step 4：构建中文文档
    - name: Build Chinese version
      run: |
        # 进入中文文档目录
        cd zh
        # 构建 MkDocs，并将输出放到 public/zh 目录
        mkdocs build --site-dir ../public/zh

    # Step 5：构建英文文档
    - name: Build English version
      run: |
        # 进入英文文档目录
        cd en
        # 构建 MkDocs，并将输出放到 public/en 目录
        mkdocs build --site-dir ../public/en

    # Step 6：生成首页 index.html（语言自动识别 + 手动选择）
    - name: Create index redirect
      run: |
        # 确保 public 目录存在
        mkdir -p public

        # 创建 public/index.html
        cat > public/index.html << 'EOF'
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="utf-8">
            <title>CHCNAV SDK Documentation</title>

            <!-- 兜底用的 meta 刷新跳转 -->
            <meta http-equiv="refresh" content="0; url=./zh/">

            <script>
                // 根据浏览器语言自动跳转
                const lang = navigator.language || navigator.userLanguage;
                if (lang.startsWith('zh')) {
                    window.location.href = './zh/';
                } else {
                    window.location.href = './en/';
                }
            </script>

            <!-- 页面样式 -->
            <style>
                body { 
                    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; 
                    text-align: center; 
                    padding: 50px; 
                    background: #f5f5f5;
                    margin: 0;
                }
                .container {
                    max-width: 600px;
                    margin: 0 auto;
                    background: white;
                    padding: 40px;
                    border-radius: 10px;
                    box-shadow: 0 2px 10px rgba(0,0,0,0.1);
                }
                .logo {
                    font-size: 2.5em;
                    font-weight: bold;
                    color: #1976d2;
                    margin-bottom: 20px;
                }
                .language-selector { margin: 30px 0; }
                .language-selector a { 
                    display: inline-block; 
                    margin: 10px; 
                    padding: 15px 30px; 
                    background: #1976d2; 
                    color: white; 
                    text-decoration: none; 
                    border-radius: 8px;
                    font-weight: 500;
                    transition: background 0.3s;
                }
                .language-selector a:hover {
                    background: #1565c0;
                }
                .subtitle {
                    color: #666;
                    margin-bottom: 30px;
                }
            </style>
        </head>

        <body>
            <div class="container">
                <div class="logo">CHCNAV SDK</div>
                <h1>Documentation Center</h1>
                <p class="subtitle">Please select your language / 请选择语言</p>
                <div class="language-selector">
                    <a href="./zh/">中文文档</a>
                    <a href="./en/">English Docs</a>
                </div>
                <p><small>Redirecting automatically... / 正在自动跳转...</small></p>
            </div>
        </body>
        </html>
        EOF

    # Step 7：配置 GitHub Pages 环境
    - name: Setup Pages
      uses: actions/configure-pages@v4

    # Step 8：上传构建产物（public 目录）
    - name: Upload artifact
      uses: actions/upload-pages-artifact@v3
      with:
        # Pages 的根目录
        path: './public'

    # Step 9：部署到 GitHub Pages
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@v4

```


## Git Pages实现
本节一步一步介绍如何实现`Git Pages`，这里需要在Github上开一个空仓库，仓库名称为[doc_site_template](https://github.com/chcnavdev/doc_site_template)，设置为`public`。
![|400](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2010.25.32%402x.png?imageSlim)

仓库创建好后，可以将当前仓库clone到本地，然后将里面的`.git`目录`mv`到项目根目录下：
```bash
 ~/Desktop/testSite >> git clone git@github.com:chcnavdev/doc_site_template.git

~/Desktop/testSite >> mv doc_site_template/.git ./ # .git 拷贝到本地
```

然后就可以将仓库中的内容完全提交：
```bash
~/Desktop/testSite >>  main >> git add .
~/Desktop/testSite >>  main ✚ >> git commit -m "init"

[main (root-commit) f2e04ed] init
 5 files changed, 205 insertions(+)
 create mode 100644 .github/workflows/deploy.yml
 create mode 100644 en/docs/index.md
 create mode 100644 en/mkdocs.yml
 create mode 100644 zh/docs/index.md
 create mode 100644 zh/mkdocs.yml
```

将所有内容`push`到远程仓库：
```bash
~/Desktop/testSite >>  main >> git remote set-url origin git@github-b:chcnavdev/doc_site_template.git ## 这里我的远程地址不太一样，因为我的电脑有多个远程仓库

~/Desktop/testSite >>  main >> git push
Enumerating objects: 13, done.
Counting objects: 100% (13/13), done.
Delta compression using up to 8 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (13/13), 3.03 KiB | 3.03 MiB/s, done.
Total 13 (delta 1), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), done.
To github-b:chcnavdev/doc_site_template.git
 * [new branch]      main -> main
```

`push`完成后，去仓库主页就可以看到已经在自动构建部署了：
![|600](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2010.51.13%402x.png?imageSlim)

![](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2010.51.31%402x.png?imageSlim)
查看构建细节，会发现构建失败，这是因为`Git Pages`没有配置。将其配置如下，然后重新部署即可：
![|600](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2011.15.53%402x.png?imageSlim)

![](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2010.53.38%402x.png?imageSlim)

构建成功后，浏览器输入`用户名.github.io/仓库名`即可进入网站，输入后会自动识别中英文并切换到对应的语言的站点，也可以手动切换语言。对应中英文效果如下：
![](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2011.20.27%402x.png?imageSlim)

![](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2011.22.32%402x.png?imageSlim)

> https://chcnavdev.github.io/doc_site_template

--- 
# 主题定制
官方会提供一些通用颜色的定制，但是官网的主题色并没有，这里基于官网的主题色，用`css`对网站做了一些定制化的修改，同时还需要对配置文件进行一些修改。

`css`主要配置官网主题色，具体如下：
```css
/* 隐藏网站logo */
.md-header__button.md-logo,
.md-nav__title .md-nav__button.md-logo {
    display: none !important;
}

/* 隐藏侧边栏中的logo */
.md-nav--primary .md-nav__title .md-nav__button.md-logo {
    display: none !important;
}

/* CHCNAV 橙色主题配色 - 精确匹配官网色彩 */
:root {
  /* 主要颜色 - CHCNAV 品牌橙色 */
  --md-primary-fg-color:        #FF8C00;
  --md-primary-fg-color--light: #FFA500;
  --md-primary-fg-color--dark:  #FF7F00;
  
  /* 强调色 - CHCNAV 辅助橙色 */
  --md-accent-fg-color:         #FF7F00;
  --md-accent-fg-color--light:  #FFA500;
  --md-accent-fg-color--dark:   #FF6600;
  
  /* 链接颜色 */
  --md-typeset-a-color:         #FF8C00;
  
  /* 按钮和交互元素 */
  --md-typeset-mark-color:      rgba(255, 140, 0, 0.2);
  --md-admonition-fg-color:     #FF8C00;
}

/* 深色模式的橙色主题 */
[data-md-color-scheme="slate"] {
  /* 主要颜色 - 深色模式适配 */
  --md-primary-fg-color:        #FFA500;
  --md-primary-fg-color--light: #FFB84D;
  --md-primary-fg-color--dark:  #FF8C00;
  
  /* 强调色 - 深色模式橙色 */
  --md-accent-fg-color:         #FF8C00;
  --md-accent-fg-color--light:  #FFA500;
  --md-accent-fg-color--dark:   #FF7F00;
  
  /* 链接颜色 */
  --md-typeset-a-color:         #FFA500;
  
  /* 深色模式下的标记和提示 */
  --md-typeset-mark-color:      rgba(255, 165, 0, 0.3);
}
```
该配置文件需要中英文目录中各放一份。放完后目录如下：
```bash
~/Desktop/testSite >>  main >> tree
.
├── en
│   ├── docs
│   │   ├── index.md
│   │   └── stylesheets # 理论上放en目录下.开头隐藏该目录最好，避免后期修改文档的误改
│   │       └── extra.css
│   └── mkdocs.yml
└── zh
    ├── docs
    │   ├── index.md
    │   └── stylesheets # 理论上放en目录下.开头隐藏该目录最好，避免后期修改文档的误改
    │       └── extra.css
    └── mkdocs.yml
```

除此之外还需要修改一下配置文件，目的说明配色路径，这里我同时开启了中英文：
```yml
site_name: My Docs

# 主题配置
theme:
  name: material
  # 设置站点语言为中文
  language: zh
# >>>>>>>>>>>>>>>>>>> 改动处 Start >>>>>>>>>>>>>>>>>>>
  palette:
    # 浅色模式
    - scheme: default
      primary: custom
      accent: custom
      toggle:
        icon: material/brightness-7
        name: 切换到深色模式
    # 深色模式
    - scheme: slate
      primary: custom
      accent: custom
      toggle:
        icon: material/brightness-4
        name: 切换到浅色模式

extra_css:
  - stylesheets/extra.css # 理论上放en目录下.开头隐藏该目录最好，避免后期修改文档的误改
# <<<<<<<<<<<<<<<<<<<< 改动处 End <<<<<<<<<<<<<<<<<<<< 

nav:
  - 首页: index.md

# 额外配置
extra:
  # 多语言切换配置
  alternate:
    - name: English
      link: ../en/
      lang: en

```

改完后直接push即可，等待1分钟左右的构建时间，访问效果如下：
![](https://note-1256273063.cos.ap-shanghai.myqcloud.com/CleanShot%202026-01-15%20at%2011.42.32%402x.png?imageSlim)


# 其他配置
这里把用到的配置一起其他可能后期会用到的配置进行了注释说明：
```yml
# 站点名称（显示在网页左上角、浏览器标题等位置）
site_name: CHCNAV SDK Documentation

# 站点的正式访问地址（非常重要）
# 用于生成 canonical URL、SEO、sitemap、语言切换等
# ⚠️ 英文站点通常对应 /en/
site_url: https://chcnavdev.github.io/en/

# 主题配置（使用 MkDocs Material）
theme:
  # 使用 material 主题
  name: material

  # 浏览器标签页 / 收藏夹图标
  favicon: images/favicon.png

  # 启用的主题功能
  features:
    # 搜索联想提示（输入时自动补全）
    - search.suggest

  # 主题语言（UI 文本语言，如 Search、Copy 等）
  language: en

  # 主题配色方案（支持亮色 / 暗色模式）
  palette:
    # ===== 亮色模式 =====
    - scheme: default          # 默认亮色方案
      primary: custom          # 主色（在 extra.css 中自定义）
      accent: custom           # 强调色（在 extra.css 中自定义）
      toggle:
        # 切换按钮图标（太阳）
        icon: material/brightness-7
        # 鼠标悬浮提示文字
        name: Switch to dark mode

    # ===== 暗色模式 =====
    - scheme: slate            # Material 的暗色方案
      primary: custom
      accent: custom
      toggle:
        # 切换按钮图标（月亮）
        icon: material/brightness-4
        name: Switch to light mode

# 额外引入的自定义 CSS
# 常用于品牌色、字体、布局微调
extra_css:
  - stylesheets/extra.css

# 导航栏结构（左侧菜单 / 顶部菜单）
# 左边是显示名称，右边是对应的 Markdown 文件
nav:
  - Home: index.md
  - Revision History: revision.md
  - Quick Start: quick-start.md
  - Connection: connection.md
  - Receiver Communication: receiver-communication.md
  - CORS Connection: cors-connection.md
  - Coordinate Transformation: coordinate-transformation.md
  - IMU: imu.md
  - AR Stakeout: ar-stakeout.md
  - SLAM: SLAM.md
  - Common Data Types: common-data-types.md

# 额外信息配置（Material 扩展功能）
extra:
  # 多语言切换（右上角语言选择）
  alternate:
    # 中文站点
    - name: 中文               # 显示名称
      link: ../zh/             # 指向中文站点根路径
      lang: zh                 # HTML lang 属性

    # 英文站点
    - name: English
      link: ../en/
      lang: en

# 插件配置
plugins:
  # 内置全文搜索插件
  - search:
      # 搜索语言（影响分词规则）
      # 英文站点使用 en
      lang: en

# Markdown 扩展（增强 Markdown 能力）
markdown_extensions:
  # 代码高亮（pymdownx.highlight）
  - pymdownx.highlight:
      anchor_linenums: true    # 行号可被锚点链接
      linenums: true           # 显示行号
      line_spans: __span       # 为每一行生成 span，方便样式控制

  # 支持围栏代码块内嵌（如 mermaid、tabbed 等）
  - pymdownx.superfences

```