###问题说明：
nodejs 开启两个服务A和B，这两个服务都使用session来存储用户信息，但是问题出现了，当我在A中登录用户信息admin之后，又跑到B中进行刷新页面（B处于未登录状态）<br/>
然后又跑到A中刷新页面，发现页面中的session已经失效了。
###session配置
```
app.use(session({
    secret: 'ucenter',
    resave: true,
    saveUninitialized: true,
    cookie: { secure: false , maxAge: 1000 * 60 * 30}
}));
```
这个是刚开始的session配置，如果只运行A，不运行B，session没有出现session丢失的情况，反之亦然。
### 问题解决
重新调整session设置，让session存储在redis或者mongodb数据库中<br/>
安装:`npm install connect-redis --save` <br/>
更改session配置
```
const session = require("express-session");
const RedisStore = require('connect-redis')(session);
let options = {
    "host": "127.0.0.1",
    "port": "6379",
    "ttl": 60 * 30 // 过期时间，单位秒
}
app.use(session({
    store: new RedisStore(options),
    secret: 'ucenter',
    resave: true,
    saveUninitialized: false,
    cookie: { secure: false , maxAge: 1000 * 60 * 30}
}));
```
让session持久化之后，发现当一个IP地址同时访问A和B，发现还是会出现同样的情况，具体是什么原因呢？<br/>
查看A和B登录时候的sessionid，发现sessionid已经不再是刚刚登录时的sessionid了，而是变成了后登录的sessionid，express中的sessionid默认存储在cookie的connect.sid，我想应该是两个sessionid 都存储在cookie的connet.sid ， 所以被覆盖了，因此还得更改session配置
```
let options = {
    "host": "127.0.0.1",
    "port": "6379",
    "ttl": 60 * 30,
    "prefix": "asess:"
}
// 设置session
app.use(session({
    store: new RedisStore(options),
    secret: 'ucenter',
    name: 'u_sessionid', // 将connect.sid 更改成u_sessionid
    resave: true,
    saveUninitialized: false,
    cookie: {maxAge: 1000 * 60 * 30}
}));
```
哈哈，一切都大功告成了！！
