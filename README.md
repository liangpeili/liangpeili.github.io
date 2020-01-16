npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为 hexo）;

6. 修改\_config.yml 中的 deploy 参数，分支应为 master；

7. 依次执行 git add .、git commit -m "..."、git push origin hexo 提交网站相关的文件；

8. 执行 hexo g -d 生成网站并部署到 GitHub 上。
