# [RESTful](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
在当下前后端分离，前端设备又层出不穷(手机、平板、桌面电脑。。。)的时代，为了能够更好的前后端配合，就需要一套较为合理的api接口规范。
## 如何设计一套合理、好用的api
1. 协议  
API与用户通讯协议，总是使用HTTPs协议
2. 域名  
尽量将API部署在专用域名之下
3. 版本  
应该将API的版本号放入URL中(github采用的方式)  
![](http://ww1.sinaimg.cn/large/9da83df8gy1fknhb508yfj20h6012wec.jpg)
```
https：//api.example.com/v1/
```  
另外一种方式是：将版本信息放在http头中，但放在url中更加直观
4. 路径(终点)  
路径又称为“终点”，表示API的具体地址。  
在RESTful中，每一个网址代表一种资源(resource)，所以网址中==不能出现动词，只能是名词==，**而且所用名词往往与数据库的表名对应**；一般来说数据库表都代表着同一种记录的“集合”，所以API中的名词也应该使用复数。
```  
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees
```
5. HTTP动词(请求方式)  
对于资源的具体操作类型，由HTTP动词表示
常用的HTTP动词
```
1. GET(SELECT)：从服务端获取资源
2. POST(CREATE/INSERT)：在服务器新建一个资源
3. PUT(UPDATE)：在服务器更新资源(客户端提供改变后的完整资源)
4. PATCH(UPDATE)：在服务器更新资源(客户端提供改变的属性)
5. DELETE(DELETE)：从服务器删除资源
```
另外两个不常用到的HTTP动词(请求方式)
```
1. HEAD:获取资源的元数据(关于数据的数据,eg:历史)
2. OPTIONS：获取信息，关于资源的哪些属性是客户端可以改变的
```
栗子🌰
```
GET /zoos：列出所有动物园
POST /zoos：新建一个动物园
GET /zoos/ID：获取某个指定动物园的信息
PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID：删除某个动物园
GET /zoos/ID/animals：列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
```
在TP5中的资源路由也是根据不同的请求方式，进行不同的处理
6. 过滤信息
如果记录数量过多，需要分页，API需要提供参数，对返回结果进行过滤
```
GET /zoos：列出所有动物园

POST /zoos：新建一个动物园

GET /zoos/ID：获取某个指定动物园的信息

PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）

PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）

DELETE /zoos/ID：删除某个动物园

GET /zoos/ID/animals：列出某个指定动物园的所有动物

DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
```
7. 状态码
服务器向客户端返回的状态码和提示信息
```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。

201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。

202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）

204 NO CONTENT - [DELETE]：用户删除数据成功。

400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。

401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。

403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。

404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。

406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。

410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。

422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。

500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```
8. 错误处理
如果状态码是4XX，就应该向用户返回错误信息
```
{
    error:"提示信息"
}
```
9. 返回结果  
针对不同的操作，服务器所返回的结果规范
```
GET /collection：返回资源对象的列表（数组）

GET /collection/resource：返回单个资源对象

POST /collection：返回新生成的资源对象

PUT /collection/resource：返回完整的资源对象

PATCH /collection/resource：返回完整的资源对象

DELETE /collection/resource：返回一个空文档
```