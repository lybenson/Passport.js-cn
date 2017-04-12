验证请求与调用`passport.authenticate()`一样简单，并需要指定使用哪种策略。`authenticate()`函数签名是标准的`Connect`中间件，可以方便地用作`Express`应用程序中的路由中间件。

```js
app.post('/login',
  passport.authenticate('local'),
  function(req, res) {
    // 如果这个函数被调用则说明授权成功
    // `req.user` 包含已认证的用户
    res.redirect('/users/' + req.user.username);
  });
```

默认情况下，如果身份验证失败，`Passport`将以`401`未授权状态进行响应，并且不会调用任何其他路由处理程序。如果认证成功，将调用下一个处理程序，并将`req.user`属性设置为经过身份验证的用户。

注意：策略必须在路由中使用之前进行配置。有关详细信息，请继续阅读有关配置的章节。

### 重定向

通常在验证请求后发出重定向

```js
app.post('/login',
  passport.authenticate('local', { successRedirect: '/',
                                   failureRedirect: '/login' }));
```

在这种情况下，重定向选项会覆盖默认行为。成功认证后，用户将被重定向到主页，如果身份验证失败，用户将被重定向回登录页面。

## 消息提示 {#flash-messages}

重定向通常与消息提示组合，以向用户显示状态信息。

```js
app.post('/login',
  passport.authenticate('local', { successRedirect: '/',
                                   failureRedirect: '/login',
                                   failureFlash: true })
);
```

将`failureFlash`选项设置为`true`会指示`Passport`使用策略验证回调给出的消息来显示错误消息。这通常是最好的方法，因为验证回调可以最准确地确定身份验证失败的原因。

或者，可以设置自定义的消息提示。

```js
assport.authenticate('local', { failureFlash: 'Invalid username or password.' });
```

当认证成功时显示成功消息，可以使用`successFlash`选项。

```js
passport.authenticate('local', { successFlash: 'Welcome!' });
```

注意：使用消息提示需要一个`req.flash()`函数。 `Express 2.x`提供了此功能，但是从`Express 3.x`中删除。所以在`Express 3.x`建议使用`connect-flash`或`express-flash`中间件来提供这个功能。

### 禁用session

验证成功后，Passport将建立持久的登录session。这对于通过浏览器访问Web应用程序的用户的常见情况非常有用。但是，在某些情况下，不需要session支持。例如，API服务器通常需要为每个请求提供凭据。在这种情况下，会话支持可以通过将会话选项设置为false来安全地禁用

```js
app.get('/api/users/me',
  passport.authenticate('basic', { session: false }),
  function(req, res) {
    res.json({ id: req.user.id, username: req.user.username });
  });
```

### 自定义回调

如果内置选项不足以处理身份验证请求，则可以提供自定义回调以允许应用程序处理成功或失败

```js
app.get('/login', function(req, res, next) {
  passport.authenticate('local', function(err, user, info) {
    if (err) { return next(err); }
    if (!user) { return res.redirect('/login'); }
    req.logIn(user, function(err) {
      if (err) { return next(err); }
      return res.redirect('/users/' + user.username);
    });
  })(req, res, next);
});
```

在此示例中，请注意，`authenticate()`从路由处理程序中调用，而不是用作路由中间件。

如果认证失败，`user`将被设置为`false`。如果发生异常，将设置`err`。将传递一个可选的`info`参数，其中包含策略验证回调提供的其他详细信息。

回调可以使用提供的参数来处理认证结果，请注意，使用自定义回调时，应用程序有责任建立`session`\(通过调用`req.login()`\)并发送响应。











