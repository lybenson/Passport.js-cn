网站认证用户最广泛使用的方式是通过使用`username`和`password`。该机制的支持由`Passport-local`模块提供

### 安装

```bash
$ npm install passport-local
```

### 配置

```js
var passport = require('passport')
  , LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function(err, user) {
      if (err) { return done(err); }
      if (!user) {
        return done(null, false, { message: 'Incorrect username.' });
      }
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect password.' });
      }
      return done(null, user);
    });
  }
));
```

本地认证的验证回调接受用户名和密码参数，通过登录表单提交给应用程序。

### 表单

表单被放置在网页上，允许用户输入他们的凭据并登录。

```html
<form action="/login" method="post">
    <div>
        <label>Username:</label>
        <input type="text" name="username"/>
    </div>
    <div>
        <label>Password:</label>
        <input type="password" name="password"/>
    </div>
    <div>
        <input type="submit" value="Log In"/>
    </div>
</form>
```

### 路由

登录表单通过`POST`方法提交给服务器。使用`authenticate()`与`local`策略处理登录请求。

```js
app.post('/login',
  passport.authenticate('local', { successRedirect: '/',
                                   failureRedirect: '/login',
                                   failureFlash: true })
);
```

将`failureFlash`选项设置为true可以指示`Passport`使用上面的`verify`回调设置的消息选项来显示错误消息。

### 参数

默认情况下，`LocalStrategy`希望在名为`username`和`password`的参数中查找凭据。如果您的网站更喜欢以不同的方式命名这些字段，则可以使用选项来更改默认值。

```js
passport.use(new LocalStrategy({
    usernameField: 'email', //提交的字段名为email
    passwordField: 'passwd' //提交的字段名为passwd
  },
  function(username, password, done) {
    // ...
  }
));
```



