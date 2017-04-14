与定义`HTTP`的身份验证框架一起，`RFC 2617`还定义了`Basic`和`Digest`认证方案。这两种方案都使用用户名和密码作为验证用户的凭据，并且通常用于保护`API`。

应该注意的是，依赖用户名和密码信用可能会产生不利的安全影响，特别是在服务器和客户端之间没有高度的信任的情况下。在这种情况下，建议使用`OAuth 2.0`等授权框架。

`Basic`和`Digest`方案的支持由`passport-http`模块提供。

### 安装

```bash
$ npm install passport-http
```

### Basic认证

`Basic`方案使用用户名和密码来认证用户，这些凭据以纯文本格式传输，因此建议使用`HTTPS`去实现方案。

#### 配置

```js
passport.use(new BasicStrategy(
  function(username, password, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) { return done(err); }
      if (!user) { return done(null, false); }
      if (!user.validPassword(password)) { return done(null, false); }
      return done(null, user);
    });
  }
));
```

`Basic`认证的回调接受用户名和密码参数。

#### 保护端点

```js
app.get('/api/me',
  passport.authenticate('basic', { session: false }),
  function(req, res) {
    res.json(req.user);
  });
```

使用Basic策略的passport.authenticate\(\)去保护API。 API通常不需要session，因此可以禁用它们。

### Digest认证

`Digest`方案使用用户名和密码来验证用户。它对`Basic`方案的主要好处在于它使用了一个`challenge-response`范例来避免发送密码。

#### 配置

```js
passport.use(new DigestStrategy({ qop: 'auth' },
  function(username, done) {
    User.findOne({ username: username }, function (err, user) {
      if (err) { return done(err); }
      if (!user) { return done(null, false); }
      return done(null, user, user.password);
    });
  },
  function(params, done) {
    // validate nonces as necessary
    done(null, true)
  }
));
```

`Digest`策略使用两个回调，其中第二个是可选的。

第一个回调，被称为“秘密回调”，接受用户名和`done`函数以及相应的密码。该密码用于计算哈希值，如果与请求中包含的匹配不符，则验证失败。

第二个“验证回调”接收随机数参数，可以检查以避免重复攻击。

#### 保护端点

```js
app.get('/api/me',
  passport.authenticate('digest', { session: false }),
  function(req, res) {
    res.json(req.user);
  });
```

使用`Digest`策略的`passport.authenticate()`去保护`API`端点。 `API`通常不需要`session`，因此可以禁用它们。











