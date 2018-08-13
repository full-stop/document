## 工具 ##

* verdaccio
* nrm
* pm2

## 特点 ##

`verdaccio` 的特点：

* 不同步拉取npm库，占据大量硬盘，没有硬盘被撑爆的问题；
* 安装配置极其简单,不需要数据库；
* 支持配置上游registry配置，拉取即缓存；
* 支持forever及pm2守护进程管理；
* 私有npm package 管理
* 支持 docker 等应用容器

`verdaccio` 是上一个 `sinopia` 的交叉分支。


## 安装 verdaccio ##

```
npm i verdaccio -g
```

安装好后，执行 `verdaccio` 即可以看到本地 `NPM` 的web管理界面。


## 更换 registry ##

不推荐

```
npm config set registry http://
```

推荐

```
npm i nrm -g

nrm list | nrm ls
nrm add [name] [http://url]
nrm use [name]
nrm del [name]
```

## 创建NPM账号 ##

```
npm adduser | npm adduser --registry  http：// localhost：4873 /
username:***
password:***
```

其它用户相关命令：
```
npm login  #登陆
npm logout #退出
npm whoami #查看当前用户
```

## 发布package ##

```
npm publish
```

升级版本号

```
npm version patch #升级补丁版本号
npm version minor #升级副版本号
npm version major #升级主版本号
```

## verdaccio - profile ##

`verdaccio` 的配置文件是默认存放在用户目录中的，在window上其路径是：`~\Users\Administrator\.config\verdaccio`

* storage     : 存放 npm包的目录。
* config.yaml : `verdaccio` 的配置文件
* htpasswd    ： 用于保存注册的用户。

**config.yaml说明**

``` yaml
##设置NPM包的存放目录
storage: ./storage   

# 配置WEB UI界面
web :
    title : '搭建私有NPM'
    #logo : logo.png

## 设置用户验证的文件。
auth:                
  htpasswd:
    file: ./htpasswd
    max_users: 1000   #默认为1000，改为-1，禁止注册

# 设置其它的npm注册源(registry)
uplinks:
  npmjs:
    url: https://registry.npmjs.org/

#配置权限管理
packages:
  '@*/*':
    #表示哪一类用户可以对匹配的项目进行安装 【$all 表示所有人都可以执行对应的操作，$authenticated 表示只有通过验证的人可以执行对应操作，$anonymous 表示只有匿名者可以进行对应操作（通常无用）】
    access: $all
     #表示哪一类用户可以对匹配的项目进行发布
    publish: $authenticated

  '*':
    #表示哪一类用户可以对匹配的项目进行安装
    access: $all

    #表示哪一类用户可以对匹配的项目进行发布
    publish: $authenticated

    # 如果一个npm包不存在，它会去询问设置的代理。
    proxy: npmjs

# 日志输出设置
logs:
  - {type: stdout, format: pretty, level: http}
  #- {type: file, path: verdaccio.log, level: info}

#修改监听的端口
#listen: 0.0.0.0:4873  

```

## PM2启动 verdaccio ##

```
pm2 start verdaccio
```

## 还需要... ##

* 在服务器上搭建
* 按需进行用户权限分组？
* Nginx代理

> http://www.verdaccio.org/