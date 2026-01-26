# 前端安全最佳实践

## 前端安全概述

随着Web应用变得越来越复杂，前端安全问题也日益突出。虽然服务器端安全一直备受关注，但前端安全同样不容忽视。现代Web应用中，前端承担了越来越多的业务逻辑，因此成为攻击者的重要目标。

## 主要安全威胁

### 1. 跨站脚本攻击（XSS）

XSS是最常见的前端安全漏洞之一，攻击者通过在页面中注入恶意脚本，窃取用户数据或执行未授权操作。

#### 类型
- 存储型XSS：恶意脚本存储在服务器数据库中
- 反射型XSS：恶意脚本通过URL参数传递
- DOM型XSS：通过操纵DOM节点触发

#### 防护措施
```javascript
// 输入验证和过滤
function sanitizeInput(input) {
  const div = document.createElement('div');
  div.textContent = input;
  return div.innerHTML;
}

// 使用CSP（内容安全策略）
// 在HTTP头中设置: Content-Security-Policy: default-src 'self'
```

### 2. 跨站请求伪造（CSRF）

CSRF攻击利用用户的身份在后台执行非预期的操作。

#### 防护措施
- CSRF Token验证
- SameSite Cookie属性
- 验证Referer头

```javascript
// 示例：使用CSRF Token
fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCsrfToken()
  },
  body: JSON.stringify(data)
});
```

### 3. 点击劫持（Clickjacking）

攻击者通过透明的iframe覆盖在合法页面上来欺骗用户点击。

#### 防护措施
- 设置X-Frame-Options头
- 使用Frame-Options策略

### 4. 内容安全策略（CSP）绕过

CSP是重要的安全策略，但如果配置不当可能被绕过。

## 安全编码实践

### 1. 输入验证和输出编码

```javascript
// 安全的输入处理
function validateAndSanitize(userInput) {
  // 长度限制
  if (userInput.length > MAX_INPUT_LENGTH) {
    throw new Error('Input too long');
  }
  
  // 字符白名单验证
  const validPattern = /^[a-zA-Z0-9\s\-_]+$/;
  if (!validPattern.test(userInput)) {
    throw new Error('Invalid characters in input');
  }
  
  // HTML转义
  return userInput
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;');
}
```

### 2. 安全的DOM操作

```javascript
// 使用安全的DOM方法
const element = document.createElement('div');
element.textContent = userInput; // 使用textContent而不是innerHTML

// 或者使用模板字符串配合安全库
const template = `<p>${escapeHtml(userInput)}</p>`;
```

### 3. HTTP安全头配置

```javascript
// 服务器端设置的安全头
app.use((req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '1; mode=block');
  res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  next();
});
```

## 前端依赖安全管理

### 1. 依赖审查

- 定期更新依赖库
- 使用工具检查已知漏洞（如npm audit）
- 选择维护活跃的库

### 2. CDN安全

- 使用SRI（Subresource Integrity）验证资源完整性
- 仅从可信CDN加载资源

```html
<script src="https://cdn.example.com/library.js" 
        integrity="sha384-..." 
        crossorigin="anonymous"></script>
```

## 认证和会话管理

### 1. Token管理

```javascript
// 安全的token存储
const secureStorage = {
  setToken(token) {
    // 不要在localStorage中存储敏感token
    sessionStorage.setItem('authToken', token);
    // 或使用httpOnly cookie
  },
  
  getToken() {
    return sessionStorage.getItem('authToken');
  },
  
  clearToken() {
    sessionStorage.removeItem('authToken');
  }
};
```

### 2. 会话超时处理

```javascript
let inactivityTimer;

function resetInactivityTimer() {
  clearTimeout(inactivityTimer);
  inactivityTimer = setTimeout(() => {
    // 登出用户
    logout();
  }, INACTIVITY_TIMEOUT);
}

document.addEventListener('mousemove', resetInactivityTimer);
document.addEventListener('keypress', resetInactivityTimer);
```

## 安全工具和测试

### 1. 自动化安全测试

- OWASP ZAP
- Burp Suite
- SonarQube

### 2. 静态代码分析

- ESLint安全规则
- NodeJSScan
- Retire.js

## 最佳实践清单

- [ ] 验证所有用户输入
- [ ] 对输出进行适当的编码
- [ ] 实施内容安全策略
- [ ] 使用HTTPS协议
- [ ] 设置适当的安全头
- [ ] 实施CSRF保护
- [ ] 定期更新依赖
- [ ] 使用安全的认证机制
- [ ] 进行安全代码审查
- [ ] 执行渗透测试

## 安全事件响应

### 监控和日志

```javascript
// 安全事件监控
window.addEventListener('error', (event) => {
  // 记录错误信息，但不要泄露敏感数据
  reportSecurityEvent({
    type: 'client_error',
    message: event.message,
    url: window.location.href,
    timestamp: Date.now()
  });
});
```

## 总结

前端安全是一个持续的过程，需要开发团队的持续关注和投入。通过实施上述最佳实践，可以大大降低Web应用面临的安全风险。记住，安全不是一次性的任务，而是需要在整个开发生命周期中持续考虑的重要因素。