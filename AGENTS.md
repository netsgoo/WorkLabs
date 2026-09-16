# WorkLabs 工作手册

## 架构与数据规范
- **单文件交付**：每个工具均为自包含单 HTML 文件，内嵌 CSS/JS/SVG，无本地构建流水线，依赖仅走公共 CDN。
- **行情接口**：腾讯行情接口（JSONP，动态 `<script>` 节点加载后必须立即 `s.remove()` 卸载防内存泄漏）；腾讯分时接口（原生 Fetch 跨域）。A股原始成交额单位为万元。
- **走势图基线**：列表中有数据时始终默认展示首项分时图，无数据时彻底隐藏；点击表格行仅用于切换聚焦，不触发折叠。

## Git 远程同步规范
- **代理与弹窗隔离**：Git 远程操作必须附加 `-c http.proxy=http://127.0.0.1:7890 -c credential.helper=`，避免连接超时与 GUI 弹窗阻塞。
- **认证令牌调用**：访问令牌存储于系统环境变量（Machine 作用域）`GITHUB_TOKEN_NETSGOO`，通过 `[System.Environment]::GetEnvironmentVariable('GITHUB_TOKEN_NETSGOO', 'Machine')` 提取，严禁明文打印。
- **标准推送命令**：
  ```bash
  powershell -Command "$t = [System.Environment]::GetEnvironmentVariable('GITHUB_TOKEN_NETSGOO', 'Machine'); git -c http.proxy=http://127.0.0.1:7890 -c credential.helper= push https://${t}@github.com/netsgoo/WorkLabs.git main"
  ```
