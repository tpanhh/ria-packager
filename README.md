#前端打包系统:批量压缩,合并js,css. #
 1. 合并`require('a/b/c.js');`
 2. 合并`@import url("a/b/c.css");` 重新计算背景图片相对地址. 剔除重复@import (目前策略是保留第一个css；以前是保留最后一个)
 3. 计算文件md5值，用于缓存版本号。
    1. 替换背景图片地址 `background-image: url(../../img/sprite-new.png?v=md5-hash);`
    2. 生成js，css 文件内容md5映射(`md5_mapping.json`)，可用于更新或者回滚版本号。

#通过npm安装:
 1.  安装老的稳定版，用于支持mobile等老模块化工程:  `npm install ria-packager@1.5.0` 
 2.  安装最新版，支持less集成及新工程目录结构: `npm install ria-packager` 
 3.  如果是安装到 **全局**，即使用`-g`选项： `sudo npm install -g ria-packager`，则可以使用 **ria-packager** 这个系统命令：
   1.  **package project** : `ria-packager -from fromDir -to toDir [-verbose or -v]`
   2.  **start    server** : `ria-packager -start`
   3.  **stop     server** : `ria-packager -stop`

#构建静态资源（合并，压缩js，css）:
 `node index.js -from ~/workspace/litb_ria/mobile/trunk/   -to /tmp/mobile/ -v `
 1. -from 参数 指明要打包的工程根目录
 2. -to 参数 指明输出目录（可以是任意临时目录）

#在线打包部署（方便不习惯命令行的用户，目前只支持linux系统）
 1. 访问 `工程名称/deploy` 路径，如`mobile/deploy` 可在线打包mobile工程为`mobile.zip`可供下载
 
#前端资源独立发布上线(url根目录可变)#
 1. 使用 **resource_xyz** 形式发布，如 **mobile** 工程打包会会变成 **mobile_90278** ,其中90278是mobile工程svn主干最新版本号。
 2. page页面使用`<link href="{{{cdn}}}/{{{resource}}}/??{{{main_css}}},{{{skin_css}}},{{{i18n_css}}}"/>`引用css。
 3. page页面使用`<script type="text/javascript" src="{{{cdn}}}/{{{resource}}}/??{{{i18n_js}}},{{{main_js}}}"></script>`引用js。
 4. {{{cdn}}},{{{skin_css}}},{{{i18n_css}}},{{{i18n_js}}}由模板数据决定.
 5. {{{resource}}}在开发期会被替换成当前工程名称，如mobile。发布时，打包系统会自根据最新svn版本号，来替换{{{resource}}}为对应的 **resource_xyz** ，如 **mobile_90278** 。
 6. 开发环境及打包系统会自动替换 {{{main_css}}}和{{{main_js}}}为模板对应的主css和js。

#前端资源独立发布上线(url根目录不变)#
 1. 如cdn支持固定资源目录，则打包时可以使用md5 hash为单个资源版本号，
 如`{{{cdn}}}/{{{resource}}}/??i18n/js/en.js,page/a/b.js?v=635116ee02ab32fd`，
 其中resource是固定工程目录，如 **/ria/mobile** 
 2. 注意： [nginx-http-concat](https://github.com/taobao/nginx-http-concat) 中 If a third ? is present it's treated as version string. 
 3. 注意： **CDNs use pull-based caching, not push-based replication**
 4. 使用 **合并后的整体静态文件** 的内容md5 hash作为控制缓存的版本号。
 5. 如果合并路径中仅有单个文件，则可使用该文件自身md5 hash做版本号。
 6. 这种方法对不变的资源缓存利用率能大幅提升，不至于因为单个文件改变就使所有资源缓存失效。

 
#辅助开发服务器（用于开发测试，联调）
1. cd 目标目录, 如`cd /data/ria/` 该目标目录`/data/ria/`即设置服务器为 **documentRoot** . 默认端口为 **8888**.
2. 启动服务器: `ria-packager -start` or `node lib/server/httpd.js`
3. 浏览器访问 /admin/debug 即可设置服务器环境为开发模式，此时按需动态合并js，css，但不压缩不混淆代码。
4. 浏览器访问 /admin/release 即可设置服务器环境为生产发布模式，此时按需动态合并，压缩（混淆）js，css。
5. 支持按照 [nginx-http-concat](https://github.com/taobao/nginx-http-concat) 的规范来动态合并静态资源，合并后的资源可使用独立版本号控制缓存。如：
  1. `http://127.0.0.1:8888/mobile/??i18n/js/en.js,page/checkout_address_process/checkout_address_process.js`
  2. `http://127.0.0.1:8888/mobile/??page/checkout_address_process/checkout_address_process.css,theme/blue/skin.css?v=99129a3f2430cb5a`

##模板测试数据及自定义模板容器：##
1. 渲染widget和pagelet时，会在模板文件父目录下查找_test/_layout.html，如果存在该模板，就使用它作为wiget的父模板。
2. 模板文件父目录下 _test/下所有.json文件会自动显示在模板数据select中，供切换以测试不同数据渲染效果。
3. 模板文件父目录下 _test/下与模板文件同名的.json文件为默认渲染模板所使用的数据文件。
