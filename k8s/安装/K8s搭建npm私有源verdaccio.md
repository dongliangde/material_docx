# 服务端

## 一、准备工作

- 安装docker

  `docker -v`验证docker是否安装

## 二、使用docker搭建npm私有仓库

1. 拉取 Verdaccio 的 docker 镜像

   ```
   docker pull verdaccio/verdaccio
   ```

2. 在根目录下创建 docker 文件

   ```
   mkdir -p ~/docker/data
   cd ~/docker/data
   ```

3. 从 git 拉取示例到 data 到目录下

   ```
   git clone https://github.com/verdaccio/docker-examples
   cd ~/docker/data/docker-examples
   ```

4. 移动配置文件

   ```
   mv docker-local-storage-volume ~/docker/verdaccio
   ```

5. 设置文件夹权限

   ```
   chown -R 10001:65533 ~/docker/verdaccio
   ```

6. 启动镜像

   ```
   docker run --name verdaccio -itd -v ~/docker/verdaccio:/verdaccio -p 4873:4873 verdaccio/verdaccio
   ```

7. 验证是否npm私有仓库是否部署成功

   打开serverip:4873，出现以下画面表示已经部署成功

# 客户端

## 一、npm私有包管理

1. 设置npm镜像源为私有仓库地址

   ```
   npm set registry serverip:4873
   ```

2. 注册用户

   ```
   npm adduser
   ```

   输入username、password以及email

3. 登录

   ```
   npm login
   ```

   输入username、password以及email

4. 上传/更新私有包

   ```
   npm publish
   ```

   - 更新私有包时，必须修改版本号

   - 使用**package.json**文件的**files**字段指定发布哪些文件或目录

     npm publish 的时候会把项目目录里面**所有文件**都publish到npm仓库中， 但是往往有一部分目录和文件不想发布上去，比如项目的源码、编译脚本等等信息。

   ```
   "files": [
       "lib/",
       "bin/",
       "buildin/",
       "declarations/",
       "hot/",
       "web_modules/",
       "schemas/",
       "SECURITY.md"
   ]
   ```

5. 删除私有包

   ```
   npm unpublish <package-name> --force
   ```

## 二、优化package.json配置文件

1. description

   字符串，软件包的简要描述。

2. author

   包作者信息，包含name、email、url等信息。

3. files

   指定发布哪些文件或目录。

4. private

   是否是私有包，如果private为true，npm会**拒绝发布**。

## 三、使用nrm快速切换源

npm包有很多的镜像源，有的源有的时候访问失败，有的源可能没有最新的包等等，所以有时需要切换npm的源，nrm包就是解决快速切换问题的。

nrm可以帮助您在不同的npm源地址之间轻松快速地切换。

1. 安装

   ```
   npm install -g nrm
   ```

   nrm内置了npm、cnpm、taobao等源

   ```
   nrm ls
   ```

2. 添加源

   ```
   nrm add <registry-name> <url>
   ```

3. 删除源

   ```
   nrm del <registry-name>
   ```

4. 切换npm使用的源

   ```
   nrm use taobao // 切换淘宝镜像
   ```

# DEMO

*在http://localhost:4873部署私有仓库*

## 一、包的创建与发布

- 创建demo项目，在main文件夹下创建index.js，该文件导出外部可用

  ```
  // main/index.js
  export const name = "name in main/index.js"
  ```

- package.json文件夹指定main字段（**包入口文件**）、files（**发布的文件或目录地址**）等信息

  ```
  {
    "name": "demo",
    "version": "1.0.0",
    "description": "hanyun demo",
    "main": "main/index.js", // 默认为index.js，可自定义其他文件 
    "files": ["main"], // 指定发布的文件或者目录地址
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "hanyun",
    "license": "ISC",
    "dependencies": {
      "echarts": "^4.8.0",
      "react": "^16.13.1",
      "webpack": "^4.43.0"
    }
  }
  ```

- 设置npm私有仓库

  ```
  npm config set registry http://localhost:4873
  ```

- 注册用户

  ```
  npm adduser
  ```

  输入username、password以及email

- 登录

  ```
  npm login
  ```

  输入username、password以及email

- 发布包

  ```
  npm publish
  ```

  出现以下信息，表示发布成功

  ```
  npm notice
  npm notice package: demo@1.0.0
  npm notice === Tarball Contents ===
  npm notice 137B main/index.js
  npm notice 343B package.json
  npm notice === Tarball Details ===
  npm notice name:          demo
  npm notice version:       1.0.0
  npm notice package size:  415 B
  npm notice unpacked size: 480 B
  npm notice shasum:        23b56f8a539af70011818147aa9e9f4c727b9609
  npm notice integrity:     sha512-btO+//rxNNzN6[...]1RL0EqPms3IeQ==
  npm notice total files:   2
  npm notice
  + demo@1.0.0
  ```

## 二、包的安装与使用

- 设置cnpm私有仓库

  ```
  cnpm config set registry http://localhost:4873
  ```

- 安装demo包

  ```
  cnpm install --save demo
  ```

- import导入包

  ```
  import {name} from 'demo'
  console.log(name) // 输出name in main/index.js
  ```

 



 

 

 