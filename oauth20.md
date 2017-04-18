`OAuth 2.0`（由`RFC 6749`正式指定）提供授权框架，允许用户授权访问第三方应用程序。授权后，应用程序将发出令牌作为验证凭证。这种方式有两个主要的安全优势：

1. 应用程序不需要存储用户的用户名和密码。

2. 令牌有受限制的范围（例如：只读访问）。

这些优点对于确保`Web`应用程序的安全性尤其重要，使`OAuth 2.0`成为`API`认证的主要标准。

使用`OAuth 2.0`保护`API`端点时，必须执行三个不同的步骤：

1. 应用程序请求用户访问受保护资源的权限。

2. 如果用户授权成功，则向应用发出令牌。

3. 应用程序使用令牌去访问受保护的资源。

### 发布token

`OAuth2orize`是`Passport`的一个兄弟项目，为实现`OAuth 2.0`授权服务器提供了一个工具包。

授权过程有着复杂的步骤，涉及对应用和用户的认证，并提示用户权限，这样来确保为用户做出明智的决定提供足够的细节。

此外，由实现者决定在应用程序上可以对访问的范围进行哪些限制，以及随后执行这些限制。

作为一个工具包，`OAuth2orize`不会尝试做出实现的决定。本指南不涵盖这些问题。

### 认证令牌

`OAuth 2.0`提供了一个框架，其中可以发出任意可扩展的令牌类型集合。在实践中，只有特定的令牌类型被广泛使用。

## Bearer Tokens {#bearer-tokens}

`bearer tokens`是`OAuth 2.0`中最广泛发行的令牌类型。实际上，很多的实现都假设`bearer tokens`是发行的唯一的令牌类型。

`bearer tokens`可以使用`passport-http-bearer`模块进行认证。

#### 安装

```bash
$ npm install passport-http-bearer
```

#### 配置

```js
passport.use(new BearerStrategy(
  function(token, done) {
    User.findOne({ token: token }, function (err, user) {
      if (err) { return done(err); }
      if (!user) { return done(null, false); }
      return done(null, user, { scope: 'read' });
    });
  }
));
```

`bearer tokens`的验证回调接受`token`作为参数。调用`done`方法时，可以传递可选的`info`，`info`信息将被设置在`req.authInfo`中。info通常用于传达`token`的范围，并且可以在进行控制访问检查时使用。

#### 保护终点

```js
app.get('/api/me',
  passport.authenticate('bearer', { session: false }),
  function(req, res) {
    res.json(req.user);
  });
```

使用`bearer`策略的`passport.authenticate()`去保护API。 `API`通常不需要`session`，因此可以禁用它们。

