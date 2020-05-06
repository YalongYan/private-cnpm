#### 前言

随着Nodejs开发的项目越来越多，Node项目管理就成了一个需要思考的问题了。如果所有项目都开源统一用 NPM 进行管理也没什么问题，但总有一些是我们不希望的完全开放的代码，作为企业是核心秘密保留在公司内部，这个时候就需要在公司内网也搭建一套 NPM 依赖管理系统。
CNPM正好就提供了这个功能。
CNPM官方githuib地址， 但是官方文档不够全面：[点我](https://github.com/cnpm/cnpmjs.org) 。
大家可以下载我的这份， 我在官方项目基础上进行了配置 [点我](https://github.com/YalongYan/private-cnpm)， 具体配置下面会讲。
官网上写的是在linux环境下，其实在什么环境都可以，这里就讲在mac下部署cnpm服务。
#### 整个过程可以分为如下几步：
###### 1.下载cnpm项目代码。
`git clone [https://github.com/YalongYan/private-cnpm](https://github.com/YalongYan/private-cnpm)
`
或者直接下载压缩包。大家下载官方的git地址也可以，做相应的配置修改就可。

###### 2.下载mysql & 导入数据表
mac下mysql的下载安装参考 [这里](https://www.jianshu.com/p/833f388da8e3)
这里先记下数据库的用户名，密码，下面会用到。
安装 MySql的可视化工具 MySqlWorkBench。
首次打开，MySqlWorkBench, 需要连接数据库，如下，点击加号， 下面的local是我之前连接好的，像这样连接好的， 下次直接双击这个数据库就可直接进入。
![image.png](https://upload-images.jianshu.io/upload_images/8551758-17dccd974e967230.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
依次输入信息，1的名字随便写， 2 3 分别是用户名，密码，填写完，点击4 测试是否能连接上，成功了点击ok。
![image.png](https://upload-images.jianshu.io/upload_images/8551758-d59f497f80d2f7a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
点击如下按钮，创建数据库：

![image.png](https://upload-images.jianshu.io/upload_images/8551758-da91ef72eeb0bc6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
依次填入信息，最后Apply(这个test是数据库名字)

![image.png](https://upload-images.jianshu.io/upload_images/8551758-3afe42a6b5e09274.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后就可以看到新增的数据库了。
![image.png](https://upload-images.jianshu.io/upload_images/8551758-1796ab8789d9f4e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
导入数据表：
![image.png](https://upload-images.jianshu.io/upload_images/8551758-2b656aff8bd24db9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择cnpm项目里，docs目录下的db.sql
记得在最上面加上一行， use test；
test是刚才的数据库名字。然后点击左边起第一个⚡️
![image.png](https://upload-images.jianshu.io/upload_images/8551758-f67cc674cead5e79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这样就导入成功了，如下：
![image.png](https://upload-images.jianshu.io/upload_images/8551758-e57ced56bc60ad9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 3.修改cnpm里面对应的配置
修改config/index.js 里面的一些参数
```
database: {
    // 设置数据库信息
    db: 'test',
    username: 'root',
    password: '123456',

    // the sql dialect of the database
    // - currently supported: 'mysql', 'sqlite', 'postgres', 'mariadb'
    dialect: 'mysql',

    // custom host; default: 127.0.0.1
    // host: '127.0.0.1',
    host: '0.0.0.0',// 用这个 是为了 用电脑ip访问

    // custom port; default: 3306
    port: 3306,

    // use pooling in order to reduce db connection overload and to increase speed
    // currently only for mysql and postgresql (since v1.5.0)
    pool: {
      maxConnections: 10,
      minConnections: 0,
      maxIdleTime: 30000
    },

    dialectOptions: {
      // if your server run on full cpu load, please set trace to false
      trace: true,
    },

    // the storage engine for 'sqlite'
    // default store into ~/.cnpmjs.org/data.sqlite
    storage: path.join(dataDir, 'data.sqlite'),

    logging: !!process.env.SQL_DEBUG,
  },
  enablePrivate: true, // 只有管理员可以发布 npm 包，默认为 false，即任何人都可以发布包
  scopes: ['@company'], // 私有包必须依附于 scope 下, 这个compan可以是公司名字啥的，自己定义。
// default system admins
  admins: {
    // name/密码: email，
   // 这里这个admin 既是用户名， 也是密码
    admin: 'admin@cnpmjs.org',
    dead_horse: 'dead_horse@qq.com',
  },
```
###### 4.启动cnpm项目
`npm install  & npm run start`
访问 [http://127.0.0.1:7002/](http://127.0.0.1:7002/)
![image.png](https://upload-images.jianshu.io/upload_images/8551758-3cbd5dce55c2d2c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
访问[http://127.0.0.1:7001/](http://127.0.0.1:7001/)
返回的是一堆json 就成功了。
这里其实可以吧127.0.0.1 换成本机ip， 这样在局域网内的其他电脑，可以直接通过ip访问我们的服务。

###### 5.发布npm包
我们需要先做两件事
1.为了方便管理npm的源， 我们安装 nrm
```
  $ npm install nrm -g

  $ nrm ls

  # npm ---- https://registry.npmjs.org/
  # cnpm --- http://r.cnpmjs.org/
  # * taobao - https://registry.npm.taobao.org/
  # nj ----- https://registry.nodejitsu.com/
  # rednpm - http://registry.mirror.cqupt.edu.cn/
  # npmMirror  https://skimdb.npmjs.com/registry/
  # edunpm - http://registry.enpmjs.org/
```
上面可以看出可以用的npm仓库地址，*标明当前使用的仓库地址，使用npm use 'name'切换仓库
2.添加本地私有仓库
cnpm提供两个端口：7001和7002，其中7001用于NPM的注册服务，7002用于Web访问
```
  $ nrm add local http://127.0.0.1:7001/

    add registry local success

  $ nrm ls

  # npm ---- https://registry.npmjs.org/
  # cnpm --- http://r.cnpmjs.org/
  # * taobao - https://registry.npm.taobao.org/
  # nj ----- https://registry.nodejitsu.com/
  # rednpm - http://registry.mirror.cqupt.edu.cn/
  # npmMirror  https://skimdb.npmjs.com/registry/
  # edunpm - http://registry.enpmjs.org/
  # local -- http://127.0.0.1:7001/
```
local就是我们刚才添加的本地私有npm仓库，执行
  `nrm use local`
  使用本地仓库

然后就可以正常使用了
```
npm login  // 输入用户名，密码
npm publish 
```
对于如何发布npm包，可以参考我之前写的这篇https://www.cnblogs.com/yalong/p/10388384.html
发布成功后 ，在7002页面可以看到 total packages 增加了。
![image.png](https://upload-images.jianshu.io/upload_images/8551758-ee42e25a4ef8ff31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
 这里只能搜索，才能找到对应的npm包
![image.png](https://upload-images.jianshu.io/upload_images/8551758-74f4332db9d97210.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/8551758-d9343a3f876c64de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###### 6.下载npm包
这里下载就跟正常的下载是一样的， 
`npm install @company/xxx-test`
因为目前我们的npm源 还是local。
以后如果想下载公网的npm， 需要切换源
` nrm use npm`

#### 期间遇到的问题：
出现下面这个问题， 是连不上数据库
```
    code: 'ER_NOT_SUPPORTED_AUTH_MODE',
    errno: 1251,
    sqlMessage: 'Client does not support authentication protocol requested by server; consider upgrading MySQL client',
    sqlState: '08004',
    fatal: true
```
解决办法如下：
1.首先进入mysql命令行 [mac下如何进入mysql命令行](https://jingyan.baidu.com/article/3065b3b6468f68becff8a4e4.html)
2.然后执行如下代码：
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
FLUSH PRIVILEGES;
```
123456换成自己的数据库密码。 重启node服务即可。
参考链接：

[https://www.cnblogs.com/JimmyLuo/p/7079634.html](https://www.cnblogs.com/JimmyLuo/p/7079634.html)

[http://blog.fens.me/nodejs-cnpm-npm/](http://blog.fens.me/nodejs-cnpm-npm/)

[https://www.jianshu.com/p/4e7414f7fcc8](https://www.jianshu.com/p/4e7414f7fcc8)

[https://github.com/zhengyange/blog/blob/master/window-cnpmjs.md](https://github.com/zhengyange/blog/blob/master/window-cnpmjs.md)

[https://www.jianshu.com/p/4e7414f7fcc8](https://www.jianshu.com/p/4e7414f7fcc8)





