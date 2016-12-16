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
