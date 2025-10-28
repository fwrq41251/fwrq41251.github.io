---
title:  shiro登录
date: 2016-05-13
tags:
  - shiro

slug:  shiro-login
---

shiro默认的登录实现使用了用户名和密码进行验证。因为项目项目需要（使用了OAuth 2.0，不需要验证密码），改写了部分登录逻辑。在此记录一下主要用到的几个类。

AuthorizingRealm是验证登录以及获取权限的核心类。这个类是个抽象类，有两个待实现的方法。

```java
AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException;

AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals);
```

我们需要编写自己的Realm类继承这个抽象类并实现以上两个方法。`AuthenticationToken`包括确认用户身份的信息（比如username），以及登录凭证信息（比如password）。

`AuthenticationInfo`包括了登录凭证信息以及凭证信息的验证原则。

`AuthorizationInfo`包括了用户的角色以及权限信息。

需要注意的是AuthorizingRealm的父类里包含了要使用的`AuthenticationToken`类信息，其值是shiro的默认实现，`UsernamePasswordToken.class`。

如果你要实现自己的token，必须重新注入`authenticationTokenClass`。
