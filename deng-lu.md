`Passport`在`req`上暴露了一个`login()`函数，又叫`logIn()`可用于建立登录`session`。

```js
req.login(user, function(err) {
  if (err) { return next(err); }
  return res.redirect('/users/' + req.user.username);
});
```

登录操作完成后，`user`将被赋值到给`req.user`中。

注：`passport.authenticate()`中间件自动调用`req.login()`。当用户注册时，主要使用此功能，在此期间可以调用`req.login()`来自动登录新注册的用户。

