## Nextjs 和 Nodejs 服务端进行通信

### 简介
    1、Next.js是react框架，负责UI部分，具有约定大于配置的特点；

### 开发环境Demo
    1、需求：用户通过浏览器可以添加一个用户，每次添加完成后，在界面展示已添加用户数量。在展示区域，点击查询用户数量接口，就会请求服务端保存的用户列表，并展示在页面下方。

    2、技术选型：
        1、服务端：nodejs
            express:简单好用的web 框架
            nodemon:热部署
        2、UI端：next.js
            脚手架： npx create-next-app my-app
            请求库：fetch
            css: tailwindcss
            代理插件：http-proxy-middleware
    3、开发阶段：
        1、方案：开发阶段和生产阶段，程序run的方式完全不同。开发阶段，nodejs服务端和nextjs app分别运行在不同的端口上。这种方式前后端分离，并且开发测试起来很快。

        nextjs app 和 nodejs服务端要正常交互，还要借助运行在nextjs端的`http-proxy-middleware`服务器的力量。      

        2、项目结构
        为了方便开发和测试，我们将nodejs 和 nextjs app 放到一个文件夹中，并通过文件夹名api 和 my-app来区分nodejs服务端和 nextjs app。每个文件夹下，各自维护package.json和node_moudles。这种方式好在问题不会串，方便调试。

        3、开发nodejs服务端
        使用express监听3080端口，创建两个接口，分别是:
            /api/users : 查询所有用户
            /api/user :  添加用户接口

        4、开发nextjs app
            1、开发ui界面。
                在ui界面中获取请求服务中的两个函数，并将返回值setState到组件中
            2、开发请求服务，这里将把调用nodejs 接口的过程封装成两个函数。分别是：
            export async function getAllUsers() //使用fetch 调用接口
            export async function createUser(data)//使用fetch调用接口
        5、nextjs app 与 node接口间的交互
            【4：中并没有指定具体的url】
            1、配置代理：通过代理，我们可以将所有api调用代理到nodejs的api上去。
            2、配置代理的两种方式：
                2.1 配置式：直接在react app的package.json中加上proxy：url:port 如："proxy": "http://localhost:3080"
                2.2 编程式：
                    1、修改nextjs 的启动脚本
                    2、在nextjs中安装http代理中间件
                    3、使用http代理中间件创建代理服务server.js。将所有/api/
                        接口都代理到nodejs上。
            3、为http代理服务添加启动脚本

            4、启动项目：
                1、启动nextjs app
                2、启动nodejs 接口服务

## 生产环境部署：
    开发环境中，借助http代理插件才能和nodeapi 进行交互。开发环境中，不能这样，要先build nextjs app，然后将结果放到nodeserver端使用。     

    1、编译nextjs app ，编译成功后会在my-app下生成out文件        
    2、修改nodejs 接口端，使用express.static 加载out文件夹
    3、修改nodejs服务端，通过/加载index.html

    效果：最终启动nodejs服务端， 你会在【3080端口】访问到整个应用。也就是说3000端口你不用启了。

    注意：在生成环境下，一旦你想修改nextjs app的任何东西，你修改完后，必须build一下才会起作用。

## 总结
    1、There are so many ways we can build Next.js apps and ship them for production.
    
    2、One way is to build the Next.js app with NodeJS.
    
    3、In the development phase, we can run Next UI and Nodejs on separate ports.
    
    3、The interaction between these two happens with proxying all the calls to API.
    
    4、In the production phase, you can build the Next app and put all the assets in the build folder and load it with the node server.
    
    5、Nodejs act as a web server as well in the above case. You need to let express know where are all the Next build assets are.
    
    6、We can package the application in a number of ways.
    You can deploy the packaged zip folder in any node environment.

## BUG 列表
```
1、> NODE_ENV='development' node server.js

'NODE_ENV' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
```
原因：并非代码bug 而是os引起的。在win中，同时执行两条命令要用&&,
set NODE_ENV=development && node dev-server.js
