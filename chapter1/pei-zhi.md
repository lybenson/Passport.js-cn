需要三个配置以使用`Passport`进行身份验证:

1. 认证策略
2. 中间件
3. `session`\(可选\)

### 策略\(Strategies\)

`Passport`使用策略来认证请求，策略的范围从验证用户名和密码，委托认证使用`OAuth`或联合身份验证使用`OpenID`。

在要求`Passport`验证请求之前，必须配置应用程序使用的策略\(可配置多个策略\)。

策略及其配置通过`use()`函数提供。例如，以下使用`LocalStrategy`进行用户名和密码认证。

```js
var passport = require('passport')
  , LocalStrategy = require('passport-local').Strategy;

passport.use(new LocalStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function (err, user) {
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

### 验证回调

这个例子介绍一个重要的概念。策略需要验证回调，目的是找到拥有一组凭据的用户。

当`Passport`验证请求时，它将解析请求中包含的凭据。然后，它将使用这些凭据作为参数调用验证回调，在本例中为`username`和`password`。如果凭据可用，则验证回调函数调用`done`函数，以向验证的用户提供`Passport`

```js
return done(null, user);
```



