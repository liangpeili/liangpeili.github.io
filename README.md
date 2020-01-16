# 说明

hexo 分支为源文件， master 分支为生成的静态文件。

如果切换电脑，只需要在新电脑先全局安装 hexo-cli，然后克隆本仓库，并执行 npm install 即可。

文章发布流程：

1. 依次执行 git add .、git commit -m "..."、git push origin hexo 提交网站相关的文件；
2. 执行 hexo g -d 生成网站并部署到 GitHub 上。
