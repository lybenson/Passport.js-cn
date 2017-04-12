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

注意：使用`Flash`消息需要一个`req.flash()`函数。 Express 2.x提供了此功能，但是从Express 3.x中删除。所以在Express 3.x建议使用[connect-flash](https://github.com/jaredhanson/connect-flash)中间件来提供这个功能。

### 禁用session

### 自定义回调



