验证请求与调用`passport.authenticate()`一样简单，并需要指定使用哪种策略。`authenticate()`函数签名是标准的`Connect`中间件，可以方便地用作`Express`应用程序中的路由中间件。

```js
app.post('/login',
  passport.authenticate('local'),
  function(req, res) {
    // If this function gets called, authentication was successful.
    // `req.user` contains the authenticated user.
    res.redirect('/users/' + req.user.username);
  });
```

默认情况下，如果身份验证失败，`Passport`将以`401`未授权状态进行响应，并且不会调用任何其他路由处理程序。

