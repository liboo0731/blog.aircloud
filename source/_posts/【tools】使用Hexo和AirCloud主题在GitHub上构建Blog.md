---
title: 【tools】使用Hexo和AirCloud主题在GitHub上构建Blog
date: 2018-10-24 14:51:39
tags: 
    - tools
    - hexo
    - aircloud
    - github
---

### Hexo常用命令

```bash
#新建一篇博客
hexo new post "blog title"
#删除生成的public文件夹
hexo clean
#生成public文件夹
hexo g
#启动本地访问服务，默认：http://localhost:4000
hexo s
#部署到GitHub项目中，需要预先配置
hexo d
```

### Hexo配置文件（_config.yml）

```yaml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: STRIVELIBOO Blog
subtitle: 既然选择了远方，便只顾风雨兼程！
author: liboo
language: zh
timezone:

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: https://liboo0731.github.io
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

#Custom Setting Start

# Site settings
# 网站综合内容设置：
SEOTitle: STRIVELIBOO的博客 | STRIVELIBOO's Blog
email: striveliboo@163.com
description: "李博的博客"
keyword: "LIBOO"

# SNS settings
# 一些社交平台地址，支持以下几种：
zhihu_username:     li-bo-2-25
github_username:    liboo0731

# Build settings
anchorjs: true                          # if you want to customize anchor. check out line:181 of `post.html`

sidebar-avatar: img/avatar.jpg      # use absolute URL, seeing it's used in both `/` and `/about/`

# Friends
# 友情链接
friends: [
    {
        title: "STRIVELIBOO_博客园",
        href: "https://www.cnblogs.com/liboo/"
    },{
        title: "搞事情的LIBOO_CSDN",
        href: "https://blog.csdn.net/weixin_40108079"
    }
]

#comment:
#  type: gitment
#  id: your-id-created-by-https://github.com/settings/applications/new
#  secret: your-secret-created-by-https://github.com/settings/applications/new
#  owner: aircloud
#  repo: hexo-aircloud-blog

comment:
   type: disqus
   script: 'https://airclouds-blog.disqus.com/embed.js'


# The following content is not recommended to modify
# 搜索数据文件路径设置，不建议改动：
search:
  path: search.json
  field: post

# 文章样式(是否首行缩进)：
post_style:
  indent: default

# Custom Setting End
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archive
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: true
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: true
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 8
pagination_dir: page

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: aircloud

# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:liboo0731/liboo0731.github.io.git
  branch: master

```

