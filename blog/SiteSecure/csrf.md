# 页面安全-CSRF攻击
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
  因为黑客会利用用户的登录状态来发起 CSRF 攻击，**而 Cookie 正是浏览器和服务器之间维护登录状态的一个关键数据**，所以先考虑从 Cookie 做文章

  * Secure 属性
  
    标记为 Secure 的 Cookie 只应通过被 HTTPS 协议加密过的请求发送给服务端。它永远不会使用不安全的 HTTP 发送（本地主机除外），这意味着中间人攻击者无法轻松访问它。不安全的站点（在 url 中带有 http：）无法使用 Secure 属性设置 cookie。

  * 使用 SameSite: Strict | Lax (默认值) | None;

  Strict: 浏览器会完全禁止第三方 Cookie

  Lax: 在跨站点的情况下，从第三方站点的链接打开和从第三方站点提交 Get 方式的表单这两种方式都会携带 Cookie。但如果在第三方站点中使用 Post 方法，或者通过 img、iframe 等标签加载的 URL，这些场景都不会携带 Cookie。

  None: 在任何情况下都会发送 Cookie 数据。

  在 HTTP 响应头中，通过 set-cookie 字段设置 Cookie 时，可以带上 SameSite 选项, 如下：
  ```http
  set-cookie: 1P_JAR=2019-10-20-06; expires=Tue, 19-Nov-2019 06:36:21 GMT; path=/; domain=.google.com; SameSite=none; Secure
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

### 对 Cookie 进行双重验证
  具体步骤：
  - 身份验证
    * 用户登录：当用户登录时，服务端随机生成一个令牌（例如 JWT 或者 sessionId）, 并将其存储在服务器的数据库中，同时将该令牌发给浏览器端作为 cookie
    * cookie 传递: 浏览器在后续的每次请求中都会携带 cookie
    * 验证 cookie: 服务器在每次请求中验证 cookie 中的令牌，如果令牌有效并且与已知用户关联，那么用户被视为已经身份验证。
  - 数据完整性验证
    * 为了验证 cookie 的数据完整性，服务端一般会使用加密和签名技术。
      1. 在生成 cookie 时，服务器对 cookie 进行加密（非对称加密和对称加密）
      2. 生成一个签名，签名中包含 cookie 和密钥
      3. 将加密后的 cookie 和签名一起发送给浏览器
    * 在每个请求中，服务器将验证 cookie 数据的签名，以确保数据没有被更改过。  
    
## 小结
要发起 CSRF 攻击需要具备三个条件：目标站点存在漏洞、用户要登录过目标站点和黑客需要通过第三方站点发起攻击。

根据这三个必要条件，我们又介绍了该如何防止 CSRF 攻击，具体来讲主要有三种方式：充分利用好 Cookie 的 SameSite 属性、验证请求的来源站点和使用 CSRF Token。这三种方式需要合理搭配使用，这样才可以有效地防止 CSRF 攻击。

## QA
### CSRF token工作原理
  CSRF token 的基本工作原理：

  - **生成 CSRF token：** 服务器在用户登录时生成一个唯一的 CSRF token，**并将其关联到用户的会话**。

  - **嵌入 CSRF token：** 服务器将生成的 CSRF token 嵌入到每个表单中，可以通过隐藏字段或者设置请求头的方式。

  - **提交请求：** 用户在进行敏感操作时，浏览器会自动将表单中的 CSRF token 包含在请求中，发送给服务器。

  - **验证 CSRF token：** 服务器接收到请求后，**会从用户的会话中获取正确的 CSRF token，**并与请求中的 CSRF token 进行比较。

  - **验证通过：** 如果两个 CSRF token 匹配，服务器会执行请求中的操作。

  - **验证失败：** 如果 CSRF token 不匹配，服务器会拒绝执行请求，防止被攻击者利用。

  **通过使用 CSRF token，可以防止攻击者伪造请求并执行非法操作，因为攻击者无法获取到有效的 CSRF token。这种方式增加了请求的安全性，确保操作只能由合法用户发起。**

### 实现 CSRF token 注意事项
  在实现 CSRF token 时，需要注意以下几点：

  - CSRF token 应该是随机且足够复杂，以增加猜测的难度。

  - CSRF token 应该与用户的会话关联，保证每个用户拥有独立的 CSRF token。

  - CSRF token 应该在每次请求中都进行验证，确保每个请求都经过有效的安全检查。

### CSRF token 如何发送给服务端
  CSRF token 发送给服务器的两种方式：
  
  1. **嵌入到表单的隐藏字段：** 在 HTML 表单中，可以添加一个隐藏字段来包含 CSRF token。当用户提交表单时，浏览器会将隐藏字段的值自动包含在请求中发送给服务器。服务器在接收到请求时，通过解析请求数据来获取 CSRF token。

  ```html
    <form action="/submit" method="POST">
      <input type="hidden" name="csrf_token" value="CSRF_TOKEN_VALUE">
      <!-- 其他表单字段 -->
      <button type="submit">提交</button>
    </form>
  ```

  2. **设置请求头：** 另一种常见的方法是将 CSRF token 设置为请求头的一部分。在发送请求之前，客户端代码可以将 CSRF token 添加到请求头中，然后将请求发送给服务器。服务器在接收到请求时，通过读取请求头来获取 CSRF token。

  ```js
    const xhr = new XMLHttpRequest();
    xhr.open('POST', '/submit');
    xhr.setRequestHeader('X-CSRF-Token', 'CSRF_TOKEN_VALUE');
    // 其他请求配置
    xhr.send();  
  ```
  **无论是隐藏字段还是请求头，服务器都需要相应的机制来解析请求数据，并从中提取 CSRF token 进行验证。具体的实现方式可能因后端框架或库的不同而有所差异。**

### CSRF token 为什么能够防止跨站请求攻击？
  CSRF token 能够防止跨站请求攻击（CSRF）的原因是**攻击者无法获取到合法的 CSRF token，从而无法成功伪造请求。**

  由于 CSRF token 是由服务器生成并与用户会话关联的，攻击者无法获取到合法的 CSRF token。攻击者无法直接从其他站点或攻击代码中获取到用户的 CSRF token 值，因为浏览器会对跨域请求进行限制。

  在进行 CSRF 攻击时，攻击者会尝试伪造请求，但由于缺乏合法的 CSRF token，服务器将拒绝执行这些请求。**CSRF token 的唯一性和随机性使得攻击者无法预测或猜测有效的 CSRF token 值，从而有效地阻止了 CSRF 攻击。**

  总结起来，CSRF token 防止跨站请求攻击的关键在于攻击者无法获取到合法的 CSRF token，而服务器在每次请求中要求验证 CSRF token 的合法性。这种机制确保了只有合法的请求（携带正确的 CSRF token）才能被服务器接受和执行，从而有效地保护了应用程序免受 CSRF 攻击的危害。

### CSRF token 是如何与用户会话关联的？
  当用户成功登录到应用程序时，服务器会为该用户创建一个会话，并在服务器端存储与该会话相关的信息。这些信息可以包括用户的身份验证状态、权限、会话标识符等。通常，会话信息会存储在服务器的内存或持久化存储中，以便在用户与服务器之间的交互中进行身份验证和状态管理。

  在生成 CSRF token 时，服务器会将生成的 CSRF token 关联到用户的会话信息中。这样，每个用户会话都有一个唯一的 CSRF token，用于验证该用户发起的请求的合法性。

  当用户进行敏感操作时，浏览器会自动将 CSRF token 包含在请求中，无论是作为隐藏字段嵌入到表单中还是设置为请求头的一部分。服务器在接收到请求后，会从用户的会话中提取正确的 CSRF token，并与请求中的 CSRF token 进行比较，以验证请求的合法性。

  通过将 CSRF token 与用户会话关联，可以确保每个用户拥有独立且有效的 CSRF token。这种关联机制保证了 CSRF token 的唯一性和有效性，并提供了一种有效的方式来验证用户请求的合法性，从而防止 CSRF 攻击。

### CSRF token 和 用户会话信息区别
  **在生成 CSRF token 的阶段，生成的 CSRF token 和用户成功登录后存储在服务器的会话信息是不同的。**

  当用户成功登录后，服务器会创建一个会话并分配一个会话标识符（session identifier），该标识符通常以 cookie 的形式发送给用户的浏览器，并在浏览器和服务器之间建立会话状态。

  同时，服务器会在会话信息中存储与该用户相关的数据，如用户的身份验证状态、权限、会话过期时间等。这些会话信息通常存储在服务器的内存或持久化存储中，以便在用户与服务器之间的交互中进行身份验证和状态管理。

  在生成 CSRF token 时，服务器会创建一个随机且唯一的 CSRF token，并将其关联到用户的会话信息中。这个 CSRF token 是专门用于防止 CSRF 攻击的安全令牌，用于验证用户请求的合法性。

  虽然 CSRF token 和用户会话信息都存储在服务器端，但它们是不同的数据。会话信息是用于管理用户身份和状态的数据，而 CSRF token 是用于防止 CSRF 攻击的一种安全机制。

  通过将 CSRF token 关联到用户的会话信息中，可以确保每个用户拥有独立且有效的 CSRF token，并且在验证请求的合法性时可以从会话中获取正确的 CSRF token 进行比较。这种关联机制提供了一种有效的方式来保护应用程序免受 CSRF 攻击的威胁。