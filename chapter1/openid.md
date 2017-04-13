`OpenID`是联合身份验证的开发标准。访问网站时，用户会显示其`OpenID`以登录。然后，用户使用他们选择的`OpenID`提供者进行身份验证，该提供者去确认用户的身份。该网站验证该断言以便对用户进行签名。

`OpenID`的支持由`passport-openid`模块提供。

### 安装

```js
$ npm install passport-openid
```

### 配置

使用`OpenID`时，必须指定返回的`URL`和域名`realm`。`returnURL`是`OpenID`认证后要重定向到的`URL`。`realm`表示验证有效的`URL`空间的一部分。通常是网站的根`URL`。

```js
var passport = require('passport')
  , OpenIDStrategy = require('passport-openid').Strategy;

passport.use(new OpenIDStrategy({
    returnURL: 'http://www.example.com/auth/openid/return',
    realm: 'http://www.example.com/'
  },
  function(identifier, done) {
    User.findOrCreate({ openId: identifier }, function(err, user) {
      done(err, user);
    });
  }
));
```

`OpenID`认证的验证回调接收包含用户声明的`identifier`参数。

### 表单

表单被放置在网页上，允许用户输入`OpenID`并登录

```html
<form action="/auth/openid" method="post">
    <div>
        <label>OpenID:</label>
        <input type="text" name="openid_identifier"/><br/>
    </div>
    <div>
        <input type="submit" value="Sign In"/>
    </div>
</form>
```

### 路由

`OpenID`身份验证需要两个路由。第一个路由接受包含`OpenID`标识符的表单提交。在认证期间，用户将被重定向到其OpenID提供者。第二条路由是用户在与OpenID提供者进行身份验证后将被返回的URL。

```js
// 接受OpenID标识符并将用户重定向到其OpenID验证提供者去验证。
// 完成后，提供者将重定向, 用户回到应用程序的路由/auth/openid/return
app.post('/auth/openid', passport.authenticate('openid'));

// OpenID提供已将用户重定向到应用程序
// 通过验证断言来完成认证过程,如果有效，用户将被登录，否则认证失败
app.get('/auth/openid/return',
  passport.authenticate('openid', { successRedirect: '/',
                                    failureRedirect: '/login' }));
```



