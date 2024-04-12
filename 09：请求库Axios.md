## 09：请求库Axios

> - `Axios`是一个基于`promise`网络请求库，作用域`node.js`和浏览器中
> - `Axios`在服务端它使用原生`node.js http`模块，而在客户端（浏览器）则使用`XMLHttpRequests`
> - `Axios`可以拦截请求和响应、转换请求和响应数据、取消请求、自动转换`JSON`数据
> - `Axios`安装方式：`npm install axios`



### 9.1：`Axios`常用配置项

> 以下这些是创建请求时最常用的配置选项：
>
> *提示：只有`url`是必须的，如果没有指定`method`，则默认是`GET`*



```json
{
    url : '/user',//请求的服务器地址URL
    method : 'GET',//请求方式，默认值是GET
    baseURL : 'https://some-domain.com/api/',//如果URL不是绝对地址，发送请求时在url前方加上 baseURL
    headers : { 'X-Requested-Width':'XMLHttpRequest' },//自定义请求头
    params : { ID:12345 },//与请求一起发送的URL参数
    data : { firstName:'Fred' },//作为请求体被发送的数据，仅适用'PUT' 'POST' 'DELETE' 'PATCH'
    timeout : 1000,//请求超过这些毫秒，则会中断。默认是0（永不中断）
    responseType : 'json',//期待服务器返回的数据类型：'arraybuffer' 'document' 'json' 'text' 'stream'，浏览器专属：'blob'，默认值'json
    /**
    * 允许在向服务器发送前，修改请求数据，它只能用于：'PUT' 'POST' 'PATCH'这几个请求方法
    * 在其内部可以对发送的data进行任意转换处理
    * */
    transformRequest : [function(data, headers) {
        return data;
    }],
    /**
    * 在传递给 then/catch 前，允许修改响应数据
    * 可以对接收的data进行任意转换处理
    * */
    transformResponse : [function(data) {
        return data;
    }]
 }
```



### 9.2：发送请求

> 可以向`axios`传递相关配置来创建请求



#### 9.2.1：`axios( config )`

```js
// 1.发送一个POST请求
axios({
    method : 'POST',
    url : '/example-url/_'
    // 其他配置
})

// 2. 发送一个GET请求
axios({
    method : 'GET', // 请求方式可省略不写
    url : '/example-url/_'
    // 其他配置
})
```



#### 9.2.2：`axios( url[, config] )`

```js
// 发送一个post请求
axios(
	'/example-url/__',
    {
        method : 'POST',
        // 其他配置
    }
)

// 发送一个get请求
axios(
	'/example-url/__',
    {
        method : 'GET',
        // 其他配置
    }
)
```



#### 9.2.3：请求方式别名

> - 为了方便起见，已经为所有支持的请求方法提供了别名
> - 1. `axios.request( config )`
>   2. `axios.get( url[, config] )`
>   3. `axios.delete( url[, config] )`
>   4. `axios.head( url[, config] )`
>   5. `axios.options( url[, config] )`
>   6. `axios.post( url[, data[, config]] )`
>   7. `axios.put( url[, data[, config]] )`
>   8. `axios.patch( url[, data[, config]] )`
> - *注意：在使用别名方法时，`url`、`method`、`data`这些属性都不必在`config`中指定*



### 9.3：自定义创建实例

> `axios.create( [config] )`：调用`crerate`函数传入自定义配置，来创建自定义`axios`实例



```js
// axiosInstance.js
import axios from 'axios'

const request = axios.create({
    baseURL : 'https://some-domain.com/api/',
    timrout : 1000,
    headers : {
        'X-Custom-Header' : 'foobar'
    }
});

export default request;
```

```js
// 使用自定义实例发送请求

import request from 'axiosInstance.js'
// 1：
request({
    url : '/examel-url/__',
    method : 'POST',
    // __其他配置__
})
// 2:
request('/examel-url/__', {
    method : 'POST',
    // __其他配置__
}) 
// 3：
request.post(
	'/examel-url/__',
    // 请求方法体
    // __其他配置__
)
```



### 9.4：响应数据

> - 发送请求通过`.then(response => { })`来获取服务器响应的数据
> - `response`响应式结构：
>   1. `data`：服务器提供的响应<font color=red>（最重要）</font>
>   2. `status`：来自服务器响应的http状态码，成功则为`200`，请求地址不存在则为`404`，服务器异常则是`500`，请求方式错误是`405`。
>   3. `statusText`：来自服务器相应的http状态信息
>   4. `headers`：服务器响应头
>   5. `config`：请求的配置信息
>   6. `request`：生成此响应的请求，在`node.js`中它是最后一个`ClientRequest`实例，在浏览器中则是`XMLHttpRequest`实例



**代码演示：**

```js
//1：
axios({
    method : 'GET',
    url : 'http://www.ss.com',
    // 其他配置
}).then(response=> {
    console.log(response);
});

//2：
axios.get('http://www.s.s', {
    // 其他配置
}).then(response=> {
    console.log(response);
})
```



### 9.5：请求错误处理

> 发送请求后，使用`.catch(error => { })`来处理此次请求异常，请求成功发出且浏览器也相应了状态码，但状态代码超出了`2xx`的范围



**代码演示：**

```js
axios({
    method : 'GET',
    url : 'httpsadsdsd.sds.sss',
    // 配置
}).catch(error => {
    console.log('请求失败');
});
```



### 9.6：解决跨域问题

> 1. 跨域：指的是浏览器不能执行其他网站的脚本；它是由浏览器的同源策略造成的，是浏览器对`javascript`施加的安全限制
> 2. 同源策略：是指协议，域名，端口都要相同，其中一个不同就会产生`跨域`
> 3. 浏览器为了安全问题一般都限制了跨域访问，也就是不允许跨域请求资源，如果未处理跨域访问则会在请求时控制台出现`Access-Control-Allow-Origin...`的报错信息
> 4. 如何处理跨域问题，可在`vite`项目的`vite-config.js`文件中添加`proxy`代理（server选项中）



**代码演示：**

```js
export defoult defineConfig({
    // 服务
    server : {
        // 代理
        proxy : {
            '/ok' : {
                target : 'https://www/api.com/api', //代理后台服务器地址
                changeOrigin : true, //允许跨域
                rewrite : path=> path.replace(/^\/ok/, '') // 将请求地址中的 /ok 替换成空
            }
        }
    }
});
```

