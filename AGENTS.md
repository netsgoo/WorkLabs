# WorkLabs 开发者与智能体工作手册

WorkLabs 是面向高效办公与个人量化工具的免构建单文件 Web 应用仓库。每个子目录均作为一个完全独立、开箱即用的纯前端工具提供。

## 关键约束
- **交付形式**：单 HTML 文件内嵌全部资源，无本地构建，仅用公共 CDN。
- **行情规范**：腾讯 JSONP 动态 `<script>` 节点载入后必须立即 `s.remove()` 防泄漏；A股成交额原始单位为万元。
- **走势图基线**：有数据默认展首项，无数据彻底隐藏；点击行仅切聚焦不折叠。

## Git 推送
```bash
powershell -Command "$t = [System.Environment]::GetEnvironmentVariable('GITHUB_TOKEN_NETSGOO', 'Machine'); git -c http.proxy=http://127.0.0.1:7890 -c credential.helper= push https://${t}@github.com/netsgoo/WorkLabs.git main"
```
