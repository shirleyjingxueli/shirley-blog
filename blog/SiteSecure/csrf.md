# 网站安全-CSRF攻击
  CSRF（Cross-site request forgery）,又称为“跨站请求伪造”。是指黑客引诱用户打开黑客的网站，在黑客的网站中，利用用户的登录状态发起的跨站请求。简单来讲，**CSRF 攻击就是黑客利用了用户的登录状态，并通过第三方的站点来做一些坏事**

  **和 XSS 不同的是，CSRF 攻击不需要将恶意代码注入到用户的页面，仅仅是利用服务器的漏洞和用户的登录状态来实施攻击。**

## 实施 CSRF 攻击的方式
### 自动发起 Get 请求
```html
  <!DOCTYPE html>
  <html>
    <body>
      <h1>黑客的站点：CSRF攻击演示</h1>
      <img src="https://time.geekbang.org/sendcoin?user=hacker&number=100">
    </body>
  </html>
  <!--将接口隐藏在了img标签内，当图片加载时，就会自动请求-->
```
### 自动发起 POST 请求
  黑客在自己的站点上伪造 POST 请求，当用户打开黑客网站时，自动提交 POST 请求
  ```html
    <!DOCTYPE html>
    <html>
    <body>
      <h1>黑客的站点：CSRF攻击演示</h1>
      <form id='hacker-form' action="https://time.geekbang.org/sendcoin" method=POST>
        <input type="hidden" name="user" value="hacker" />
        <input type="hidden" name="number" value="100" />
      </form>
      <script> document.getElementById('hacker-form').submit(); </script>
    </body>
    </html>
    <!--黑客在网站中构建了自动提交表单的方式，实现跨站点 POST 数据提交 -->
  ```
### 引诱用户点击链接
  ```html
    <div>
      <img width=150 src=http://images.xuejuzi.cn/1612/1_161230185104_1.jpg> </img> </div> <div>
      <a href="https://time.geekbang.org/sendcoin?user=hacker&number=100" taget="_blank">
        点击下载美女照片
      </a>
    </div>
    <!--点击链接，就触发了黑客设置的接口-->
  ```  

## CSRF 攻击的必要条件
- 目标站点有 CSRF 漏洞
- 用户登录过目标站点，并且在浏览器上保持着该站点的登录状态
- 需要用户打开一个第三方站点，可以是黑客的站点，也可以是一些论坛

## 避免 CSRF 攻击的方式
### 充分利用好 Cookie 的 SameSite 属性
  因为黑客会利用用户的登录状态来发起 CSRF 攻击，**而 Cookie 正是浏览器和服务器之间维护登录状态的一个关键数据**，所以先考虑从 Cookie做文章

  * 使用 SameSite: Strict | Lax (默认值) | None;

  Strict: 浏览器会完全禁止第三方 Cookie

  Lax: 在跨站点的情况下，从第三方站点的链接打开和从第三方站点提交 Get 方式的表单这两种方式都会携带 Cookie。但如果在第三方站点中使用 Post 方法，或者通过 img、iframe 等标签加载的 URL，这些场景都不会携带 Cookie。

  None: 在任何情况下都会发送 Cookie 数据。

  在 HTTP 响应头中，通过 set-cookie 字段设置 Cookie 时，可以带上 SameSite 选项, 如下：
  ```http
  set-cookie: 1P_JAR=2019-10-20-06; expires=Tue, 19-Nov-2019 06:36:21 GMT; path=/; domain=.google.com; SameSite=none
  ```

  **根据实际情况将关键 Cookie 设置为 Strict 或者 Lax 模式，这样在跨站请求时，关键的 Cookie 就不会被发送到服务器，从而使得黑客的 CSRF 攻击失效**

### 验证请求的来源站点
  **服务端验证请求来源的站点。** 由于 CSRF 攻击大多数来自第三方站点，因此服务器可以禁止来自第三方站点的请求。

  **通过 Referer 和 Origin 属性来判断**

  1. Referer 是 HTTP 请求头中的一个字段，记录了该 HTTP 请求的来源地址。（注意：Referer可以设置为不用上传。）

  2. Origin 属性：在一些重要的场合，比如通过 XMLHttpRequest、Fecth 发起跨站请求或者通过 Post 方法发送请求时，都会带上 Origin 属性。

    Origin 属性只包含了域名信息，并没有包含具体的 URL 路径，这是 Origin 和 Referer 的一个主要区别。

  **因此，服务端应优先判断 Origin, 如果请求头中没有 Origin，再根据实际情况判断是否使用 Referer 值。**  
  
### CSRF Token
  使用 CSRF Token 来验证。
  * 浏览器向服务端发起请求时，服务器生成一个 CSRF Token. CSRF Token 其实就是服务器生成的字符串，然后将该字符串植入到返回的页面中。
  * 浏览器端如果要发起操作请求，需要带上页面中的 CSRF Token，然后服务器会验证该 Token 是否合法。如果是从第三方站点发出的请求，那么将无法获取到 CSRF Token 的值，所以即使发出了请求，服务器也会因为 CSRF Token 不正确而拒绝请求。

## 小结
要发起 CSRF 攻击需要具备三个条件：目标站点存在漏洞、用户要登录过目标站点和黑客需要通过第三方站点发起攻击。

根据这三个必要条件，我们又介绍了该如何防止 CSRF 攻击，具体来讲主要有三种方式：充分利用好 Cookie 的 SameSite 属性、验证请求的来源站点和使用 CSRF Token。这三种方式需要合理搭配使用，这样才可以有效地防止 CSRF 攻击。