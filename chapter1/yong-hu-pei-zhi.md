当使用第三方服务（如`Facebook`或`Twitter`）进行身份验证时，用户个人资料信息通常可用。每个服务往往具有不同的方式去编码信息。为了使整合更容易，`Passport`可以尽可能地规范配置文件信息。

可用的常见字段如下表所示。

`provider` {`String`}    用户认证的提供商（`facebook`，`twitter`等）

`id` {`String`}     由服务提供商生成的用户的唯一标识符

`displayName` {`String`}    该用户的名称，适当显示

`name` {`Object`}

```
 famileName {String}     用户的姓氏

 givenName {String}     用户的名称

 middleName {String}     用户的中间名
```

`emails` {`Array`}\[n\]

```
value {String}    实际的电子邮件地址
type {String}    电子邮件地址（家庭，工作等）的类型
```

`photos` {`Array`}\[n\]

```
 vaule {String}   图片的地址
```

注: 不是所有上述字段都可以从每个服务提供商获得，一些提供商可能包含这里未描述的其他信息，有关详细信息，请参阅`provider-specific`文档

