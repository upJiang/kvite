总结：

1.Vite，一个基于浏览器原生ES模块的开发服务器。利用浏览器去解析模块，在服务器端按需编译返回，完全跳过了打包这个概念，服务器随起随用。同时另有有Vue文件支持，还搞定了热更新，而且热更新的速度不会随着模块增加而变慢。
2.Vite要求项目完全由ES模块模块组成，common.js模块不能直接在Vite上使用。因此不能直接在生产环境中使用。在打包上依旧还是使用rollup等传统打包工具。
3.Vite的基本实现原理，就是启动一个koa服务器拦截浏览器请求ES模块的请求。通过路径查找目录下对应文件的文件做一定的处理最终以ES模块格式返回给客户端
4.使用esbuild打包工具（go语言，go这样的静态语言会比动态语言快很多）

步骤：
首先启动一个koa服务器，对首页(index.html)、js文件、裸模块比如"vue"、vue文件等进行分别处理
1.先返回index.html,然后再index.html中去加载main.js,在main.js中再去加载其它文件
2.加载main.js中的裸模块，比如"vue",vite会通过预打包，将vue模块的内容打包到node_modules中，然后替换路径，
import {createApp} from 'vue'
import {createApp} from '/@modules/vue'
通过 /@modules标识去node_module中查找并返回相对地址
3.加载vue文件，当Vite遇到一个.vue后缀的文件时。由于.vue模板文件的特殊性，它被分割成template，css，脚本模块三个模块进行分别处理。最后放入script，template，css发送多个请求获取。
vite遇到.vue后缀时，会使用vue中的compiler方法进行解析并返回

文件的执行顺序：
localhost ==》 client(websocket) ==> main.js ==> env.js ==> vue.js(裸模块vue) ==> app.vue ==> 最后就是执行里面的路由，组件，ui库等


热更新原理
Vite的热加载原理，实际上就是在客户端与服务端建立了一个websocket链接，当代码被修改时，服务端发送消息通知客户端去请求修改模块的代码，完成热更新。
查看network,在localhost后会执行client文件，就是在这里建立webcocket实现热更新，然后再进入main.js


服务端原理
服务端做的就是监听代码文件的更改，在适当的时机向客户端发送websocket信息通知客户端去请求新的模块代码。

客户端原理
Vite的websocket相关代码在处理html中时被编写代码中

查看演示代码命令：node kvite/kvite.js

