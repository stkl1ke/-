# 使用Nginx、tomCat打包项目部署至阿里云服务器的过程

需求将老旧官网穿透后查看效果，选择在服务器上部署。

## 阿里云esc服务器配置

1. 购买服务器
2. 打开控制台配置，安全组的配置规则，添加80、443、22
3. 设置登陆密码和远程连接vnc的密码
4. VNC远程连接
5. 开机后搭建环境jdk和Tomcat和Nvinx

## jdk环境配置

**配置系统环境变量**：
（1）**JAVA_HOME**：服务器存放jdk文件的路径
（2）**CLASSPATH**: %JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar

   (3)**Path**添加: %JAVA_HOME%\bin **和** %JAVA_HOME%\jre\bin

cmd 输入:**java -version** ，看是否出现版本消息，如图说明配置成功。

## Tomcat配置

bin文件夹cmd运行startup.bat

conf文件夹，打开server.xml文件,port改为80

打包后的项目，dist文件夹改成项目名移动到webapp文件夹里

在浏览器输入localhost:8080 出现小猫页面说明Tomcat部署成功。后面跟''/项目名''说明项目安装

```

```

![image-20231129111303519](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20231129111303519.png)

### Nvinx配置

nginx/conf/nginx.conf  

1.注销掉 root html和index index.html index.htm 这两行（不注释不知道有没有问题 我没试过）

2.在location / {} 里面 加上proxy_pass http://localhost:8080/项目名/  （这里最后一个/要打上）

（下面的设置最大文件需要的可以设置一下  10m代表最大上传为10M） 

```java
server {
        listen       80;
        #server_name  访问的域名，默认localhost
        server_name  localhost;
        #server_name  www.itargets.cn;
        #charset koi8-r;
        #access_log  logs/host.access.log  main;
        location / {
            #去掉默认静态
            #root   html;
            #index  index.html index.htm;
            #添加映射
            proxy_pass  http://localhost:8080/debate/;
            #上传文件的最大  设置为10M
            #proxy_temp_file_write_size 10m;
        }
        #error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
```

好了  以后你的域名就等于你设置的值了

即 http://location:8080/项目名/

![image-20231129111400521](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20231129111400521.png)

## 打包Vue项目

### 项目配置

项目根目录下创建vue.config.js文件

```js

module.exports = {
    //打包
    publicPath: './',
    outputDir: 'test', //打包输出目录
    assetsDir: './static', //放置生成的静态资源
    filenameHashing: true, // 生成的静态资源在它们的文件名中包含了 hash 以便更好的控制缓存
    lintOnSave: false, //设置是否在开发环境下每次保存代码时都启用 eslint验证
    productionSourceMap: false,// 打包时不生成.map文件
    
    // 解决跨域配置
    devServer: {                //记住，别写错了devServer//设置本地默认端口  选填
        port: 8080,
        proxy: {                 //设置代理，必须填
            '/api': {              //设置拦截器  拦截器格式   斜杠+拦截器名字，名字可以自己定
                target: 'http://localhost:9090',     //代理的目标地址(后端设置的端口号)
                changeOrigin: true,              //是否设置同源，输入是的
                pathRewrite: {                   //路径重写
                    '/api': ''                     //选择忽略拦截器里面的单词
                }
                /*也就是在前端使用/api可以直接替换为(http://localhost:9090)*/
            }
        }
    },
}
```

查看路由中(router/index.js)是否使用history,是的话修改为hash。或者将mode直接注掉，因为默认使用hash。

```javascript

const router = new VueRouter({
  /*mode: 'history',*/
  mode: 'hash',
  routes:[]
})
 
```

最后run build