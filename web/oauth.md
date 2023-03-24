# OAuth2

> OAuth2是一种授权协议，它允许用户授权第三方应用访问他们的资源，而不需要分享他们的用户名和密码。

## 为什么需要OAuth

### 举个例子

举个例子，被裁程序员再就业到小区当保安，每天面对需要进入小区的访客询问他们的身份给他们开门。例如快递小哥需要经常进出小区，但是每次都要保安去帮忙开门，身为保安大哥身心俱疲，快递小哥也抱怨非常麻烦。但是小区门禁的密码自然是不能随便告诉访客的，那有没有一种机制可以让快递小哥方便地进入小区呢。

聪明的保安大哥想到了一种授权机制，需要进入小区的快递小哥来找保安大哥申请授权码，下次快递小哥只需要自己输入授权码，验证通过后即可进入小区。并且这个授权码还可绑定其他权限，例如该授权码不仅能进入大门，还能进入13号楼，但是不能去其他楼。

同时授权码是有有效期的，过期无法使用，也保证了安全。保安大哥和快递小哥都非常满意。

### 应用场景

三方登录是我们经常遇到的场景，我们直接使用现有的第三方账号登录某个网站，而无需重新注册。例如我们开发一个网站，想要支持直接使用 GitHub 账号直接登录，那么 GItHub 就提供了 OAuth 的方式为我们提供了授权。

## OAuth2的基本流程

1. 客户端向资源所有者请求授权。
2. 资源所有者同意授权，并重定向到授权服务器。
3. 授权服务器验证资源所有者的身份，并询问是否同意授予客户端所需的权限。
4. 资源所有者同意授予权限，并重定向回客户端，附带一个授权码（Authorization Code）或一个访问令牌（Access Token），取决于所使用的授权类型（Grant Type）。
5. 如果客户端收到了授权码，它需要向授权服务器请求换取访问令牌，并提供自己的身份凭证（Client Credentials）。
6. 授权服务器验证客户端的身份和授权码，并返回一个访问令牌和一个可选的刷新令牌（Refresh Token）给客户端。
7. 客户端使用访问令牌向资源服务器请求资源。
8. 资源服务器验证访问令牌，并返回所请求的资源给客户端。  

## 四种授权模式

OAuth2支持四种不同的授权类型，分别适用于不同的场景：

- 授权码模式（Authorization Code Grant）：适用于有自己后台服务并能保护自己身份凭证安全性的客户端，如网站或移动应用。这是最常用也最安全的模式，因为它可以避免将访问令牌暴露在浏览器或设备上。这种模式需要两次重定向和一次后台请求才能获取访问令牌。
- 简化模式（Implicit Grant）：适用于没有自己后台服务或不能保护自己身份凭证安全性的客户端，如单页面应用或浏览器插件。这种模式只需要一次重定向就能获取访问令牌，但是存在较高风险被窃取或滥用。因此，建议使用较短时间有效期并限制范围较小权限范围(Scope) 的访问令牌。
- 密码模式（Password Grant）：适用于与资源所有者有高度信任关系并能直接获取其用户名和密码(例如电子邮件服务) 的客户端。这种模式不需要重定向，只需要一次后台请求就能获取访问令牌。但是这种模式存在较高风险泄露用户敏感信息或被钓鱼攻击。因此，建议只在特殊情况下使用，并尽量采用其他更安全(例如双因素认证) 的方式验证用户身份。
- 客户端凭证模式（Client Credentials Grant）：适用于客户端以自己的名义，而不是代表用户访问受保护资源的情况。在这种情况下，客户端直接向授权服务器进行身份验证，并获得访问令牌。这种授权类型通常用于客户端访问自己拥有的资源，而不是代表用户访问资源。

## 授权码模式

授权码模式是最常用和最安全的模式，因此本次我们主要介绍该模式

上文已经介绍了OAuth2的基本流程，假设我们搭建了一个网站就叫`my-site`，域名为`https://my-site.com`我们想通过`GitHub` 进行登录，那我要如何做呢？

首先我们要进行一些准备工作，那就是到 [GitHub](https://github.com/settings/applications/new "GitHub new OAuth application")上注册 OAuth application，需要告诉我们网站的域名`my-site.com`已经授权后的回调`url`我们定为`https://my-site.com/auth/callback`。注册成功后会生成一个 `client_id` 和 `client_secret`后续会使用到。

> Ok, 有了上述的准备工作就可以开始进行OAuth2授权的流程了。

### 请求授权码

当我们在`my-site`中点击`使用 GitHub登录`时，我们会跳转到GitHub的授权请求页面来获取授权码

```bash
GET https://github.com/login/oauth/authorize
```

请求参数

| 名称           | 类型     | 说明                          |
| ------------ | ------ | --------------------------- |
| client_id    | string | 注册应用时生产的 client id          |
| redirect_uri | string | 获取授权码后回调的uri，即我们注册时填写的回调url |
| scope        | string | 授权范围                        |
| state        | string | 不可猜测的随机字符串，它用于防止跨站请求伪造攻击    |

在跳转的 GitHub 页面用户完成登录授权后 GitHub 会将浏览器重定向到 `redirect_uri` 并附加上`code`即授权码和`state`:`https://my-site.com/auth/callback?code=abc&status=123`。获得授权码后即可进行下一步获取 Access Token。

### 获取 Access Token

拿到授权码即可到GitHub上获取访问令牌即Access Token

```bash
POST https://github.com/login/oauth/access_token
```

请求参数

| 名称            | 类型     | 说明                 |
| ------------- | ------ | ------------------ |
| client_id     | string | 注册应用时生产的 client id |
| client_secret | string | 注册应用时对应的密钥         |
| code          | string | 授权码                |

该请求则会返回 `access_token`，通过 `token` 即可访问 GitHub Api 获取用户信息等相关授权范围内的数据。

通常情况下，获取 `access_token` 的过程会放在服务端来保证`access_token`和`client_secret`的安全性。

### 请求GitHub数据

获取了`access_token`后即可使用该`token`获取GitHub授权范围内的数据了，例如获取GitHub的用户信息

```bash
# header
# Authorization: token_type access_token
Authorization: Bearer OAUTH-TOKEN
GET https://api.github.com/user
```
