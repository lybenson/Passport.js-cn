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

注意：策略必须在路由中使用之前进行配置。有关详细信息，请继续阅读有关配置的章节

