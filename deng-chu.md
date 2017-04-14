`Passport`在`req`上暴露了一个`logout()`函数，又叫`logOut()`。当在任意的路由中需要中断登录`session`时被调用。调用`logout()`将删除`req.user`属性并清除登录`session`

```js
app.get('/logout', function(req, res){
  req.logout();
  res.redirect('/');
});
```



