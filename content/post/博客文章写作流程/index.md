+++
date = '2025-11-13T15:14:19+08:00'
draft = true
title = '博客文章写作流程'
+++

 # 新建文章 #
```
hugo new content post/文章名/index.md
```
 1. 用 $ VS Code $ 编辑刚生成的 $content/post/$我的新文章.md 文件。
 2. 在终端运行 $hugo server$，然后在浏览器打开$ http://localhost:1313$ 实时预览。

 # 发布上线: #
 ```
 git add .
 git commit -m "Publish: 我的新文章"
 git push
 ```

 完成！ 你已经拥有了一个完全自动化、部署在 GitHub Pages 上的现代化博客系统。