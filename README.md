##### 前言

随着Nodejs开发的项目越来越多，Node项目管理就成了一个需要思考的问题了。如果所有项目都开源统一用 NPM 进行管理也没什么问题，但总有一些是我们不希望的完全开放的代码，作为企业是核心秘密保留在公司内部，这个时候就需要在公司内网也搭建一套 NPM 依赖管理系统。
CNPM正好就提供了这个功能。
#### 1. 全局安装cnpm
  `npm install -g cnpm --registry=https://registry.npm.taobao.org`
这里cnpm源先设置为淘宝镜像
#### 2. 安装mysql数据库
- 同时推荐安装MySql的可视化工具，MySql Workbench
- 然后开启MySQL->打开GUI界面->创建一个新的数据库，导入docs/db.sql
- 记住新建的数据库名字，用户名， 密码
- 修改config/index.js 下database里面对应的参数
```
  database: {
    db: 'cnpmjstest',
    username: 'root',
    password: '123456',
```
#### 3. 运行服务
`npm install  & npm run start`
然后就找到本电脑的ip，把下面的ip地址换成自己的ip就可以访问了
[http://10.60.110.185:7002/](http://10.60.110.185:7002/)
####  4.使用
- 切换cnpm 的源
`cnpm config set registry http://10.60.110.185:7002 `
- cnpm login 进行登录
默认用户信息在config/index.js 里面
```
// default system admins
  admins: {
    // name: email
    fengmk2: 'fengmk2@gmail.com',
    admin: 'admin@cnpmjs.org',
    dead_horse: 'dead_horse@qq.com',
  },
```
由于上面修改了cnpm的源， 如果其他情况下需要用的淘宝的cnpm， 需要重新设置
`cnpm config set registry https://registry.npm.taobao.org `

#### 期间遇到的问题：
###### 一. 出现下面这个问题， 是连不上数据库
```
    code: 'ER_NOT_SUPPORTED_AUTH_MODE',
    errno: 1251,
    sqlMessage: 'Client does not support authentication protocol requested by server; consider upgrading MySQL client',
    sqlState: '08004',
    fatal: true
```
解决办法如下：
1.首先进入mysql命令行， [mac下如何进入mysql命令行](https://jingyan.baidu.com/article/3065b3b6468f68becff8a4e4.html)
2.然后执行如下代码：
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
FLUSH PRIVILEGES;
```
123456换成自己的数据库密码。 重启node服务即可。



#### 参考链接：

[https://www.cnblogs.com/JimmyLuo/p/7079634.html](https://www.cnblogs.com/JimmyLuo/p/7079634.html)

[http://blog.fens.me/nodejs-cnpm-npm/](http://blog.fens.me/nodejs-cnpm-npm/)

[https://www.jianshu.com/p/4e7414f7fcc8](https://www.jianshu.com/p/4e7414f7fcc8)



