`OAuth`是一种标准协议，允许用户授权对`Web`和桌面或移动应用程序的`API`访问。一旦获得访问权限，授权应用程序就可以代表用户使用`API`​​。`OAuth`也成为委托认证的流行机制。

`OAuth`有两种主要风格，两种都被广泛使用。

`OAuth`的初始版本由`Web`开发人员组成的的团队所开发的一个开放的标准。他们开发了`Oauth1.0`，之后被`OAuth1.0a`取代。这项工作现已由`IETF`标准化为`RFC 5849`。

`Web`授权协议工作组最近的努力集中在定义`OAuth 2.0`。由于标准化的漫长工作，提供者们已经开始部署符合各种草案的实现，每个草案的语法略有不同。

幸运的是，`Passport`屏蔽了处理`OAuth`变体的复杂性的应用程序。在许多情况下，可以使用特定于提供者的策略，而不是下面描述的通用`OAuth`策略。这减少了必要的配置，并且兼容任何提供者特有的特点。请参阅`Facebook`，`Twitter`或提供商列表，以获得首选用途。

`OAuth`的支持由`passport-oauth`提供。

### 安装

```bash
$ npm install passport-oauth
```

### Oauth 1.0

`OAuth 1.0`是一个涉及多个步骤的委托认证策略。首先，必须要有请求令牌，接下来，用户被重定向到服务提供商以授权访问，最后，在授予权限之后，用户被重新定向回应用程序，并且请求令牌\(request token\)被交换为访问令牌\(access token\)。请求访问的应用程序由`key`和`secret`来标识。

#### 配置

当使用通用`OAuth`策略时，`key`和`secret`和回调地址作为选项被传入。

```js
var passport = require('passport')
  , OAuthStrategy = require('passport-oauth').OAuthStrategy;

passport.use('provider', new OAuthStrategy({
    requestTokenURL: 'https://www.provider.com/oauth/request_token',
    accessTokenURL: 'https://www.provider.com/oauth/access_token',
    userAuthorizationURL: 'https://www.provider.com/oauth/authorize',
    consumerKey: '123-456-789',
    consumerSecret: 'shhh-its-a-secret'
    callbackURL: 'https://www.example.com/auth/provider/callback'
  },
  function(token, tokenSecret, profile, done) {
    User.findOrCreate(..., function(err, user) {
      done(err, user);
    });
  }
));
```

基于`OAuth`的策略的验证回调接收`token`，`tokenSecret`和`profile`参数。`token`是访问令牌，`tokenSecret`是其相应的秘钥。`profile`将包含由服务提供商提供的用户信息。

#### 路由

`OAuth`身份验证需要两个路由。第一个路由启动`OAuth`事务并将用户重定向到服务提供商。第二个路由是用户在与提供商进行身份验证之后被重定向到的`URL`。

```js
// 将用户重定向到OAuth提供商进行身份验证.
// 完成后, 提供商将重定向用户回到应用程序到：/auth/provider/callback
app.get('/auth/provider', passport.authenticate('provider'));

// OAuth提供商已将用户重定向到应用程序
// 尝试获取access token来完成身份验证过程
// 如果授权成功，用户将登录
// 否则,授权失败
app.get('/auth/provider/callback',
  passport.authenticate('provider', { successRedirect: '/',
                                      failureRedirect: '/login' }));
```

#### 链接

一个链接或按钮可以被放置在一个网页上，点击时开始认证过程。

```html
<a href="/auth/provider">Log In with OAuth Provider</a>
```

### Oauth 2.0

`OAuth 2.0`继承于`OAuth 1.0`，旨在克服早期版本中发现的缺点。认证流程基本相同。用户首先被重定向到服务提供商以授权访问。之后授权成功后，用户跳转回应用并携带可用于数据访问的`access token`。请求访问的应用程序（称为客户端）其是由`ID`和`secret`标识。

#### 配置

当使用通用`OAuth 2.0`策略时，客户端`ID`，客户端秘钥和回调被指定为可选项。

```js
var passport = require('passport')
  , OAuth2Strategy = require('passport-oauth').OAuth2Strategy;

passport.use('provider', new OAuth2Strategy({
    authorizationURL: 'https://www.provider.com/oauth2/authorize',
    tokenURL: 'https://www.provider.com/oauth2/token',
    clientID: '123-456-789',
    clientSecret: 'shhh-its-a-secret'
    callbackURL: 'https://www.example.com/auth/provider/callback'
  },
  function(accessToken, refreshToken, profile, done) {
    User.findOrCreate(..., function(err, user) {
      done(err, user);
    });
  }
));
```

基于`OAuth 2.0`的策略的验证回调函数接受`accessToken`，`refreshToken`和`profile`参数。`refreshToken`可用于获取新的`accessToken`，如果提供商不返回`refreshToken`，则可能未定义。`profile`包含由服务提供商提供的用户信息。

#### 路由

`OAuth 2.0`身份验证需要两条路由。第一个路由将用户重定向到服务提供商。第二个路由是用户在与提供商进行身份验证之后被重定向到的`URL`。

```js
// 将用户重定向到OAuth 2.0提供商进行身份验证.
// 完成后, 提供商将重定向用户回到应用程序到：/auth/provider/callback
app.get('/auth/provider', passport.authenticate('provider'));

// OAuth 2.0提供商已将用户重定向到应用程序
// 尝试获取access token来完成身份验证过程
// 如果授权成功，用户将登录
// 否则,授权失败
app.get('/auth/provider/callback',
  passport.authenticate('provider', { successRedirect: '/',
                                      failureRedirect: '/login' }));
```

#### 范围

当使用`OAuth 2.0`请求访问时，访问范围由`scope`选项控制。

```js
app.get('/auth/provider',
  passport.authenticate('provider', { scope: 'email' })
);
```

可以将多个`scope`指定为数组

```js
app.get('/auth/provider',
  passport.authenticate('provider', { scope: ['email', 'sms'] })
);
```

范围选项的值是提供商特定的。有关支持的范围的详细信息，请咨询提供商的文档。

#### 链接

一个链接或按钮可以被放置在一个网页上，点击时开始认证过程

```html
<a href="/auth/provider">Log In with OAuth 2.0 Provider</a>
```











