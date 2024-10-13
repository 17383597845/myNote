条件：
受害者网站引用我们的链接用户可以点击
受害者引用我们链接的a标签没有使用rel="noopener noreferrer"属性

```
1. **使用 `rel="noopener noreferrer"` 属性**：
    
    - 在链接中添加 `rel="noopener noreferrer"`，可以防止新打开的标签页通过 `window.opener` 访问父标签页。
    - `noopener`：阻止新标签页获取对父标签页的 `window.opener` 引用，从而防止操纵父页面。
    - `noreferrer`：不仅阻止 `window.opener`，还阻止浏览器发送 HTTP Referer 信息，从而进一步保护用户的隐私。
    
    html
    
    复制代码
    
    `<a href="https://third-party-site.com" target="_blank" rel="noopener noreferrer">帮助文档</a>`
    
2. **设置 `sandbox` 属性**：
    
    - 使用 `iframe` 或其他嵌入机制时，也可以通过 `sandbox` 属性限制新页面对父页面的访问。
    
    html
    
    复制代码
    
    `<a href="https://third-party-site.com" target="_blank" sandbox="allow-scripts">帮助文档</a>`
    

通过这种方式，恶意网站将无法利用 `window.opener` 控制用户的父页面，保护用户免受钓鱼攻击。
```