# blogsite

1. 因为有一些主题（theme）是git submodule，所以要递归clone。

        git clone --recurse https://github.com/JaydenH215/blogsite.git

2. 修改网页浏览

        cd blogsite
        hexo g
        hexo s

3. 部署

        npm install hexo-deployer-git --save
        hexo deploy