# Spring-security-remember-me

Tags: Spring

---

## Simple Hash-Based Token Approach

### 安全说明

> cookie直接保存在浏览器，任何用户都可以在该token过期之前通过该token进行自动登录。



### cookie组成

```
base64(username + ":" + expirationTime + ":" +
md5Hex(username + ":" + expirationTime + ":" password + ":" + key))

username:          As identifiable to the UserDetailsService
password:          That matches the one in the retrieved UserDetails
expirationTime:    The date and time when the remember-me token expires, expressed in milliseconds
key:               A private key to prevent modification of the remember-me token
```

### spring-security.xml

> 只需要添加remember-me标签

``` xml
   <http auto-config="true" use-expressions="true">
        <intercept-url pattern="/pages/*" access="hasRole('ROLE_USER')" />
        <form-login
                login-page="/login.html"
                default-target-url="/pages/welcome.html"
                authentication-failure-url="/pages/login.html?error"
                login-processing-url="/defaultLogin"
                username-parameter="username"
                password-parameter="password" />
        <logout
                logout-url="/logout"
                logout-success-url="/login.html"
                invalidate-session="true" />

        <remember-me key="myAppKey"/>
        <http-basic />
        <csrf disabled="true"/>
    </http>
```

### remember-me标签常用配置

> remember-me-parameter默认为`'remember-me'`！！！和版本3之前不一样。

|配置属性|说明|默认|
|---|---|---|
|remember-me-parameter|checkbox的name|remember-me|
|remember-me-cookie|保存在浏览器的cookie名字|remember-me|
|key|用来防止修改token的一个key|a secure random value|
|token-validity-seconds|cookie有效时间（单位s）|14天|
|data-source-ref|指定数据源|*|

### 表单

``` html
<form action="/defaultLogin" method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="checkbox" name="remember-me">remember me
    <input type="submit" value="Login">
</form>
```

## Persistent Token Approach

### 安全说明

> * 一但用户使用remember-me登录，服务端将有一个表保存登录信息，包括usename、series、token、last_used，浏览器同样有cookie。当下一次使用cookie登录成功后，将重新生成新的token，替换原有token，序列号不变。

> * 如果检查cookie时，cookie中包含的username和序列号跟数据库中保存的匹配，但是token不匹配,将删除数据库中与当前用户相关的所有token记录

### 数据库建表

``` sql
create table persistent_logins (username varchar(64) not null,
								series varchar(64) primary key,
								token varchar(64) not null,
								last_used timestamp not null)
```

### spring-security.xml

``` xml
<http>
...
<remember-me data-source-ref="dataSource"/>
</http>
```




