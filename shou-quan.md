应用程序可能需要合并来自多个第三方服务的信息。在这种情况下，应用程序将要求用户“连接”，例如他们的`Facebook`和`Twitter`帐户。

当这种情况发生时，用户将已经被应用程序认证过，并且任何后续的第三方帐户只需要被授权并与用户相关联即可。因为在这种情况下的认证和授权是类似的，所以`Passport`提供了一种兼容这两种方法。

授权通过调用`passport.authorize()`执行。如果授权成功，则策略的验证回调提供的结果将被赋值给`req.account`。现有的登录`session`和`req.user`将不受影响。

```js
app.get('/connect/twitter',
  passport.authorize('twitter-authz', { failureRedirect: '/account' })
);

app.get('/connect/twitter/callback',
  passport.authorize('twitter-authz', { failureRedirect: '/account' }),
  function(req, res) {
    var user = req.user;
    var account = req.account;

    // 关联twitter账号到登录的用户
    account.userId = user.id;
    account.save(function(err) {
      if (err) { return self.error(err); }
      self.redirect('/');
    });
  }
);
```

在回调路由中，您可以看到`req.user`和`req.account`两者的使用。新连接的帐户与登录的用户相关联并保存到数据库。

### 配置

用于授权的策略与用于认证的策略相同。然而，应用程序可能希望通过相同的第三方服务提供认证和授权。在这种情况下，可以通过调用`use()`，在`user()`中覆盖策略的默认名称来使用命名策略。

```js
passport.use('twitter-authz', new TwitterStrategy({
    consumerKey: TWITTER_CONSUMER_KEY,
    consumerSecret: TWITTER_CONSUMER_SECRET,
    callbackURL: "http://www.example.com/connect/twitter/callback"
  },
  function(token, tokenSecret, profile, done) {
    Account.findOne({ domain: 'twitter.com', uid: profile.id }, function(err, account) {
      if (err) { return done(err); }
      if (account) { return done(null, account); }

      var account = new Account();
      account.domain = 'twitter.com';
      account.uid = profile.id;
      var t = { kind: 'oauth', token: token, attributes: { tokenSecret: tokenSecret } };
      account.tokens.push(t);
      return done(null, account);
    });
  }
));
```

在上面的示例中，您可以看到`twitter-authz`策略是找到或创建一个`Account`实例来存储`Twitter`帐户信息。结果将被赋值给`req.account`，允许路由处理帐户与经过身份验证的用户相关联。

### 关联验证回调

上述方法的一个缺点是它需要两个相同策略和支持路由的实例。

为避免这种情况，请将策略的passReqToCallback选项设置为true。启用此选项后，req将作为验证回调的第一个参数传递。

```js
passport.use(new TwitterStrategy({
    consumerKey: TWITTER_CONSUMER_KEY,
    consumerSecret: TWITTER_CONSUMER_SECRET,
    callbackURL: "http://www.example.com/auth/twitter/callback",
    passReqToCallback: true
  },
  function(req, token, tokenSecret, profile, done) {
    if (!req.user) {
      // Not logged-in. Authenticate based on Twitter account.
    } else {
      // Logged in. Associate Twitter account with user.  Preserve the login
      // state by supplying the existing user after association.
      // return done(null, req.user);
    }
  }
));
```

使用req作为参数传递，验证回调可以使用请求的状态来定制身份验证过程，使用单个策略实例和一组路由来处理身份验证和授权。例如，如果用户已经登录，则可以关联新的“已连接”帐户。还可以使用在req上设置的任何其他应用程序特定属性，包括req.session。

  


  


















