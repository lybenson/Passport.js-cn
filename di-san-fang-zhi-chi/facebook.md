`Facebook`策略允许用户使用其`Facebook`帐户登录到`Web`应用程序。在内部，`Facebook`身份验证工作使用`OAuth 2.0`

对`Facebook`的支持由`passport-facebook`模块实现。

### 安装

```bash
$ npm install passport-facebook
```

### 配置

为了使用`Facebook`身份验证，您必须先在`Facebook`开发者平台创建一个应用程序。创建完成，应用程序将被分配一个`App ID`和`App Secret`。并且还必须要填写重定向地址，`Facebook`将在批准你的应用访问后重定向到该地址。

```js
var passport = require('passport')
  , FacebookStrategy = require('passport-facebook').Strategy;

passport.use(new FacebookStrategy({
    clientID: FACEBOOK_APP_ID,
    clientSecret: FACEBOOK_APP_SECRET,
    callbackURL: "http://www.example.com/auth/facebook/callback"
  },
  function(accessToken, refreshToken, profile, done) {
    User.findOrCreate(..., function(err, user) {
      if (err) { return done(err); }
      done(null, user);
    });
  }
));
```

`Facebook`认证的验证回调函数接收`accessToken`，`refreshToken`和`profile`参数。`profile`包含由`Facebook`提供的用户信息。

注：出于安全考虑，重定向的`URL`必须与在`Facebook`开发者平台上填写的重定向地址一样。

### 路由

`Facebook`身份验证需要两条路由。第一条路由将用户重定向到`Facebook`，第二条路线是`Facebook`在登录后将重定向用户的`URL`。

```js
app.get('/auth/facebook', passport.authenticate('facebook'));

app.get('/auth/facebook/callback',
  passport.authenticate('facebook', { successRedirect: '/',
                                      failureRedirect: '/login' }));
```

请注意，回调路由的URL与配置策略时指定的`callbackURL`选项的`UR`L相匹配。

### 权限

如果您的应用程序需要扩展权限，则可以通过设置`scope`选项来申请。

```js
app.get('/auth/facebook',
  passport.authenticate('facebook', { scope: 'read_stream' })
);
```

可以将多个权限指定为数组。

```js
app.get('/auth/facebook',
  passport.authenticate('facebook', { scope: ['read_stream', 'publish_actions'] })
);
```

### 链接

可以在网页上放置一个链接或按钮，允许使用`Facebook`进行一键登录

```html
<a href="/auth/facebook">Login with Facebook</a>
```



