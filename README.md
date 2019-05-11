# blogsite

1. 因为有一些主题（theme）是git submodule，所以要递归clone。

        git clone --recurse https://github.com/JaydenH215/blogsite.git

2. 修改网页浏览

        //进入网站根目录
        cd blogsite

        //leancloud依赖该插件
        npm install eslint --save

        //用于统计功能
        npm install hexo-leancloud-counter-security --save

        hexo g
        hexo s

3. 部署

        npm install hexo-deployer-git --save
        hexo deploy