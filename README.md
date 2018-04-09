\* **电脑更换或者需要在别的电脑上传文章方法**

1. git clone -b 分支名称 https://github.com/en-nina/en-nina.github.io.git
2. 新电脑需要先重新 ** 安装Nodejs **  ** 安装Git**
3. 进入clone下的文件夹 执行先后执行
   主题官网：https://hexo.io/themes/
   以下是maupassant主题需要安装的插件
   npm install hexo-renderer-pug --save
   npm install hexo-renderer-sass --save
   然后执行 npm install
   经过多次尝试 先执行npm install 后安装插件会删除掉hexo的安装文件（原因不知）
   所以最好最后执行npm install
4. 执行hexo -g  、hexo -d即可