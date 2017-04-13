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

这个例子介绍一个重要的概念。策略需要验证回调，目的是找到拥有一组有效的用户。

当`Passport`验证请求时，它将解析请求中包含的凭据\(请求中的参数\)。然后，它将使用这些凭据作为参数调用验证回调，在本例中为`username`和`password`。如果凭据可用，则验证回调函数调用`done`函数，以向`Passport`提供已验证的用户

```js
return done(null, user);
```

如果凭证无效\(例如，如果密码不正确\)，则应使用`false`调用完成，而不是用户来指示身份验证失败。

```js
return done(null, false);
```

可以提供附加的信息消息来表明失败的原因。显示这个信息对于提示用户去修改代码是很有用的。

```js
return done(null, false, { message: 'Incorrect password.' });
```

最后，如果在验证凭据时发生异常\(例如，如果数据库不可用\)，`done`应该提供一个`error`参数。

```js
return done(err);
```

注：区分可能发生的两个失败情况很重要，一个是服务器异常，其中`err`会被设置为非空值。另一个失败情况是认证失败，其是正常的错误。需确保err保持为空，使用最后的参数传递其他详细信息。

通过以这种方式委派，验证回调使Passport数据库不可知。应用程序可以自由选择如何存储用户信息，而不需要认证层施加任何假设

### 中间件

在Connect或Express-based的应用中，需要passport.initialize（）中间件来初始化Passport。如果您的应用程序使用持久性登录会话，也必须使用passport.session（）中间件。

```js
app.configure(function() {
  app.use(express.static('public'));
  app.use(express.cookieParser());
  app.use(express.bodyParser());
  app.use(express.session({ secret: 'keyboard cat' }));
  app.use(passport.initialize());
  app.use(passport.session());
  app.use(app.router);
});
```

注：该支持会话支持是完全可选的，尽管建议大多数应用程序。如果启用，请务必在passport.session（）之前使用express.session（），以确保以正确的顺序恢复登录会话。

### Sessions

在典型的Web应用程序中，用于验证用户的凭据只能在登录请求期间传输。如果认证成功，将通过用户浏览器中设置的cookie建立和维护会话。

每个后续请求将不包含凭据，而是唯一标识会话的cookie。为了支持登录session，Passport将会将会话中的用户实例序列化和反序列化。

```js
passport.serializeUser(function(user, done) {
  done(null, user.id);
});

passport.deserializeUser(function(id, done) {
  User.findById(id, function(err, user) {
    done(err, user);
  });
});
```

在该示例中，仅将用户ID序列化到session中，保持会话中存储的数据量较小。当接收到后续请求时，该ID用于查找用户并将被存储到req.user中。

序列化和反序列化逻辑由应用程序提供，允许应用程序选择适当的数据库和/或对象映射器，而不需要认证层的强制。

