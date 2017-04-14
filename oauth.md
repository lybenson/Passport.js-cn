`OAuth`（由`RFC 5849`正式指定）为用户提供了一种方式，允许第三方应用程序访问其数据，而不会将其密码暴露给这些应用程序。

该协议大大提高了Web应用程序的安全性。特别是，`OAuth`在引起对外部服务密码泄露的潜在危险方面非常重要。

虽然`OAuth 1.0`仍然被广泛使用，但它已被`OAuth 2.0`所取代。建议使用`OAuth 2.0`。

使用`OAuth`保护API端点时，必须执行三个不同的步骤:

1. 应用程序请求用户访问受保护资源的权限。

2. 如果用户授权成功，则向应用发出令牌。

3. 应用程序使用令牌来验证访问受保护的资源。

### 发出令牌token

[OAuthorize](https://github.com/jaredhanson/oauthorize)，是`Passport`的同类型项目。其提供了一个实现`OAuth`服务提供商的工具包。

授权过程有着复杂的步骤，涉及对应用和用户的认证，并提示用户权限，这样来确保为用户做出明智的决定提供足够的细节。

此外，由实现这决定在应用程序上可以对访问的范围进行哪些限制，以及随后执行这些限制。

作为一个工具包，`OAuthorize`不会尝试做出实现决的定。本指南不涵盖这些问题，但强烈建议部署OAuth的服务对所涉及的安全性考虑有完全的了解。

### 认证令牌\(Authenticating Tokens\)

一旦发出，`OAuth`令牌可以使用`passport-http-oauth`模块进行身份验证。

#### 安装

```bash
$ npm install passport-http-oauth
```

#### 配置

```js
passport.use('token', new TokenStrategy(
  function(consumerKey, done) {
    Consumer.findOne({ key: consumerKey }, function (err, consumer) {
      if (err) { return done(err); }
      if (!consumer) { return done(null, false); }
      return done(null, consumer, consumer.secret);
    });
  },
  function(accessToken, done) {
    AccessToken.findOne({ token: accessToken }, function (err, token) {
      if (err) { return done(err); }
      if (!token) { return done(null, false); }
      Users.findById(token.userId, function(err, user) {
        if (err) { return done(err); }
        if (!user) { return done(null, false); }
        // fourth argument is optional info.  typically used to pass
        // details needed to authorize the request (ex: `scope`)
        return done(null, user, token.secret, { scope: token.scope });
      });
    });
  },
  function(timestamp, nonce, done) {
    // validate the timestamp and nonce as necessary
    done(null, true)
  }
));
```

与其他策略相比，`OAuth`需要两个回调。在`OAuth`中，请求应用程序的标识符和用户特定的令牌都被编码为凭证。

第一个回调被称为“消费者回调”，用于查找发出请求的应用程序，包括分配秘钥`secret`。

第二个回调被称为“令牌回调”，用于识别用户以及令牌对应的秘钥。消费者提供的secret和令牌回调用于计算签名，如果与请求签名不匹配，则认证失败。

最后的“认证回调”是可选的，可以通过检查请求中使用的时间戳和随机数来防止重复的攻击。

#### 保护端点

```js
app.get('/api/me',
  passport.authenticate('token', { session: false }),
  function(req, res) {
    res.json(req.user);
  });
```

使用`token`策略的passport.authenticate\(\)去保护API。 API通常不需要session，因此可以禁用它们。

