### MOCK造数据的常用方法
#### Mock.mock( rurl?, rtype?, template|function(options) )  拦截请求
- rurl为要拦截的地址，可以为字符串或者正则表达式     可选参数
> 比如/\/domain\/list\.json/、
- rtype为要拦截的请求类型 可选参数
- template 模板数据
- function  拦截到后要执行的函数
   参数 options：指向本次请求的 Ajax    选项集。
```
 Mock.mock('http://goods/goodAll', () => { 
  return data;
 })
```
###  Mock.setup
mock帮我们拦截的时候的延时
```
Mock.setup({
   'timeout':800-1000
 })
 ```
#### Mock.Random  生成假数据
Mock.Random 是一个工具类，用于生成各种随机数据。     Mock.Random 的方法在数据模板中称为“占位符”，引用格式为 @占位符(参数 [, 参数]) 。
##### 常用的随机数据
1. [a,b]之间的整数：
```
Random.integer(0, 10)   'num|0-10':0
```
2. 随机生成长度为a-b之间的文字：  
```
Random.ctitle(a,b)   @ctitle(a,b)
```
3. 生成随机布尔值：     
```
Random.Boolean()     @Boolean
```
4. 生成随机图片：   
```
Random.image()         'image':"@Image('100x40','#c33', '#ffffff','小北鼻')"
```
5. 生成随机字符串（英文）
```
Random.string(pool, min, max) @string(pool, min, max)
Random.string( 7, 10 )
// => "UuGQgSYk"
```
6. 随机生成一个整形数组
```
Random.range(start,end,step) 
生成start到end（不包括end），每隔step一个数据     
Random.range(10)
// => [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
Random.range(3, 7)
// => [3, 4, 5, 6]
Random.range(1, 10, 2)
// => [1, 3, 5, 7, 9]
Random.range(1, 10, 3)
// => [1, 4, 7]
```

- 参数 start：必选。数组中整数的起始值。
- 参数 stop：可选。数组中整数的结束值（不包含在返回值中）。
- 参数 step：可选。数组中整数之间的步长。默认值为 1。

7.随机生成中文名字
```
Random.cname()    @cname
```



### 举个栗子 
```
const result = {
    data: Mock.mock({
      'name': "@ctitle(6,12)",//生成长度6--12的文字
      'count|20-20000':20,//生成20-20000的整数
      "id|+1":1000,//生成ID，自增1,初始值为1000
      'timeStamp':Date.now(),//生成时间戳
      'shopTel': /^1(5|3|7|8)[0-9]{9}$/, //生成随机电话号,可以用正则表达式
      'sections|4':[
        {
          'count_down|30-100':30,
          'image':"@Image('100x40','#c33', '#ffffff','小北鼻')",
          'open':'@Boolean'
        }
      ],
      'reward':'@word',//生成随机的英文
      'nick':/钻石|幸运签|财神棒/,
      'open':'@Boolean()'

    }),
}
```
### 项目中完整的使用栗子
#### mock.js文件
> 采用自己拦截的方法，不用mock的拦截，因为如果有很多个接口的话就要写很多Mock.mock，自己拦截的话就使用对象的属性来匹配对应的数据，然后再写个延迟函数包裹，模拟网络请求返回数据的延时。
```
// import Mock from "mockjs";
const Mock = require('mockjs');
const Random = Mock.Random;
// 用 @ 来标识其后的字符串是占位符。
// 占位符 引用的是 Mock.Random 中的方法。
//可以认为@相当于Mock.Random的简写

// 要返回promise，因为我们正常的get/post用axios封装后是返回promise的
const delay = (data) =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve(data);
    }, 1000);
  });
//根据url匹配要返回的数据
const URLMap = {
  "/getHumid": () => ({
    dm_error: 0,
    error_msg: "操作成功",
    data: Mock.mock({
      'name': "@ctitle(6,12)",//生成长度6--12的文字
      'count|20-20000':20,//生成20-20000的整数
      "id|+1":1000,//生成ID，自增1,初始值为1000
      'timeStamp':Date.now(),//生成时间戳
      'shopTel': /^1(5|3|7|8)[0-9]{9}$/, //生成随机电话号,可以用正则表达式
      'sections|4':[
        {
          'count_down|30-100':30,//生成30--100的整数
          'image':"@Image('100x40','#c33', '#ffffff','小北鼻')",
          'open':'@Boolean'
        }
      ],
      'reward':'@word',//生成随机的英文
      'nick':/钻石|幸运签|财神棒/,//生成钻石|幸运签|财神棒中的其中一个
      'open':'@Boolean()'//生成随机布尔值

    }),
  }),

};
console.log(URLMap['/getHumid']());

 export default {
   get: (url) => delay(URLMap[url]),
   post: (url) => delay(URLMap[url]),
 };
```
#### api.js文件  接口文件
```
import srcConfig from 'src/config';//入口匹配配置文件
import { getRequestsByRoot } from 'axios-service';//使用axios-service给我们二次封装的axios
import Mock from './mock';
const isMock = true;
const { get, post } = isMock
  ? Mock
  : getRequestsByRoot({ root: srcConfig.APIS.serviceRoot });
  //getRequestsByRoot({ root: srcConfig.APIS.serviceRoot });返回一个对象，里面包含封装好的get/post请求方法
//get请求
const getxxx = get(url,null)
//post请求
export const postxxx = (params, data) =>
  post('xxxxxx', null, {
    params,
    data,
  })();
//如果想要以服务端返回的错误文案显示toast，要把默认的dataKey设置为null
export const postVote = (params, data) =>
  post(
    '/api/activity/grand_2020/spark/premiere/vote',
    {
      codeKey: 'dm_error',
      msgKey: 'error_msg',
      dataKey: null,
    },
    {
      params,
      data,
    }
  )();
```
