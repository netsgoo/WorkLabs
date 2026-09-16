# WorkLabs 开发者与智能体工作手册

WorkLabs 是面向高效办公与个人量化工具的免构建单文件 Web 应用仓库。每个子目录均作为一个完全独立、开箱即用的纯前端工具提供。

## 项目架构准则

### 1. 单文件交付约束（Single-file Delivery）
- **物理架构**：各子工具（如 `ding/index.html`）必须保持为单一、自包含的 HTML 文件。HTML 结构、CSS 样式、JavaScript 逻辑与 SVG 图标必须全部聚合在该文件中。
- **免构建与开箱即用**：禁止引入本地 Node.js 编译流水线（如 Webpack、Vite）或外置物理 CSS/JS 文件。所有第三方库必须通过带备用源的公共 CDN 引入（如 cdnjs / jsDelivr），确保用户本地双击或静态托管即可完整运行。
- **全局门户联动**：根目录 [index.html](file:///D:/Dev/WorkLabs/index.html) 作为工具导航总控面板，新增或重构工具后需同步更新卡片链接。

### 2. 标的监控与数据源规范（Market APIs）
- **标的范围**：当前仅支持 A股 与 港股。代码规范化通过 `normalizeCode` 统一映射为小写加前缀（如 `sh600519`、`sz002594`、`bj832978`、`hk00700`）。
- **实时行情通道**：腾讯金融行情接口 `https://qt.gtimg.cn/q={codes}`。通过动态创建 `<script>` 标签发起 JSONP 批量查询。
- **内存回收原则**：动态插入的 `<script>` 节点必须在 `onload` 和 `onerror` 回调完成后立即调用 `s.remove()` 彻底从 DOM 树卸载，避免高频轮询引起节点内存泄漏。
- **分时图接口**：腾讯分时接口 `https://web.ifzq.gtimg.cn/appstock/app/minute/query?code={code}`，使用原生 `fetch` 处理跨域响应。
- **单位换算准则**：A股原始字段成交额为万元，计算与显示时需正确转换。

### 3. 走势图基线行为准则（Chart Baseline）
- **首项对齐准则**：列表中存在监控数据时，走势图始终默认展开并渲染首条标的分时数据。
- **空列表收起准则**：当监控列表为空时，走势图卡片彻底隐藏。
- **选中与折叠分离**：点击表格行仅用于切换聚焦标的，不做折叠切换；仅通过显式收起按钮收起图表。

---

## 验证与测试守门规范（Verification Gates）

### 1. 离线语法与结构校验
修改单文件 HTML 中的脚本后，必须在终端执行 Node.js 脚本沙箱编译验证，确保无 JavaScript 语法错误：
```bash
node -e "const fs = require('fs'); const html = fs.readFileSync('ding/index.html', 'utf8'); const script = html.substring(html.indexOf('<script>') + 8, html.lastIndexOf('</script>')); new (require('vm').Script)(script); console.log('Syntax verification: PASSED');"
```

### 2. 真实浏览器端到端实机验证
涉及 UI 布局、用户交互、图表联动或网络数据时，必须调用 `chrome-devtools-mcp` 连接实际浏览器环境：
- 打开目标页面：`navigate_page` 访问 `file:///D:/Dev/WorkLabs/ding/index.html`。
- 运行时报错检查：调用 `list_console_messages` 确认 Console 消息为 0 报错。
- 交互行为测试：调用 `evaluate_script` 模拟点击、数据变更与报警触发。
- 视口核查：调用 `take_screenshot` 截图并查看，确保视觉呈现无断裂。

---

## Git 与远程仓库同步协议（Git Operations）

### 1. 网络代理与认证隔离（Proxy & Credentials）
直连 GitHub 存在网络握手超时风险，且全局凭证管理器容易触发桌面弹窗挂起。执行 Git 远程操作时需严格应用以下隔离参数：
- 注入本地代理：`-c http.proxy=http://127.0.0.1:7890`
- 禁用阻塞弹窗：`-c credential.helper=`

### 2. 认证令牌作用域（Access Token）
- GitHub 推送令牌存储于 Windows 系统环境变量（Machine 作用域）的 `GITHUB_TOKEN_NETSGOO`。
- 子进程未预加载时，需通过以下方式提取：
```powershell
[System.Environment]::GetEnvironmentVariable('GITHUB_TOKEN_NETSGOO', 'Machine')
```
- 安全红线：执行任何命令与日志输出时，严禁打印、记录或回显该令牌本身。

### 3. 标准推送范式（Verified Push Recipe）
通过 Node.js 脚本或安全命令执行带代理的令牌推送：
```javascript
const { execSync } = require('child_process');
const token = execSync("powershell -NoProfile -Command \"[System.Environment]::GetEnvironmentVariable('GITHUB_TOKEN_NETSGOO', 'Machine')\"", { encoding: 'utf8' }).trim();
execSync(`git -c http.proxy=http://127.0.0.1:7890 -c credential.helper= push https://${token}@github.com/netsgoo/WorkLabs.git main`, { stdio: 'inherit' });
```
