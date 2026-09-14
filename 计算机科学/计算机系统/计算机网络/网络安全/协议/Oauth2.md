oatuh2就是一种授权机制，数据的所有者告诉系统同意第三方应用进入系统获取这些数据，系统从而产生一个**令牌**（token），用来代替密码，允许第三方应用使用。
令牌区别于密码，有下面的优势：
1、便于发放收回（手动收回、自动过期）
2、能够控制访问权限
3、不会有泄露等安全问题
根据发放令牌方式的不同，可以将oauth的授权方式分为四种：
- 授权码（authorization-code）
- 隐藏式（implicit）
- 密码式（password）：
- 客户端凭证（client credentials）
不论哪一种方式，三方应用都需要提前在客户端中进行备案，提前获得client_id和client_secret用于后续的授权，其实这两个信息就可以看作特殊的账号密码，只不过只能用来获取授权。
# 授权码
授权码模式的核心就是先获取一个授权码，再用授权码、client_id、client_secret来请求token。
授权码的作用体现在：
1、在前端请求授权码，在后端请求token，保证了token的安全，授权码应该是一次性的，绑定的。
2、只有有授权码，后端才知道数据所有者是同意授权的。
思考：token安全吗，client_id、授权码都在前端传输了，通过这几个信息攻击者不是自己也可以拿到token？
第一步：
A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。
```javascript
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```
上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。
第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。
```javascript
https://a.com/callback?code=AUTHORIZATION_CODE
`````

上面 URL 中，`code`参数就是授权码。
第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。
```javascript
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```
上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。
第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。
```javascript
{    
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```
上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。