`Passport`是一个基于`Node.js`的认证中间件。它旨在服务于认证请求。当书写模块时，封装是一种很好的思想，所以`Passport`将所有其他功能委托给应用程序。这种分离使代码清洁和可维护，并使`Passport`非常容易集成到应用程序中。

在现代的网络应用中，用户认证可以采取各种形式。传统上，用户通过用户名和密码进行登录。随着社交网络的兴起，使用`OAuth`提供商（如`Facebook`或`Twitter`）的单点登录已成为流行的验证方法。暴露`API`的服务通常需要基于`token`的凭据来保证访问的安全。

Passport可以识别每个应用唯一的身份认证要求。认证机制，又称为策略，可以被打包为单独的模块。应用程序可以选择使用哪些策略，而不会产生不必要的依赖。

尽管认证很复杂，但代码不一定要复杂。

```js
app.post('/login', passport.authenticate('local', { successRedirect: '/',
                                                    failureRedirect: '/login' }));
```

### 安装

```bash
$ npm install passport
```



