# NodeGet Orbit Deck Theme

Version: `1.1.0`

Orbit Deck 是一个为 NodeGet 探针系统设计的自定义状态页主题。它不是简单换色，而是通过 `custom.js` 接管前端运行时 UI，重新组织节点卡片、筛选、搜索、详情页和动态图表展示。

## 特性

- 全新的探针墙卡片布局，突出在线状态、系统、资源占用、流量和更新时间。
- 节点详情页包含资源三环、近 120 秒趋势图、系统信息、网络与负载信息。
- 动态数据局部更新，避免定时刷新时整页跳动。
- 支持按在线、压力、离线状态筛选。
- 支持标签高亮筛选，默认保留全部节点显示。
- 支持搜索节点名称、UUID、来源、地区、系统、标签等字段。
- 系统图标以内联 SVG 渲染，不依赖部署环境的静态图标路径。
- 国家/地区字段会尽量匹配为国旗和国家码。

## 文件结构

```text
.
├── index.html              # 主题入口
├── config.json             # NodeGet 后端连接配置
├── custom.css              # Orbit Deck 样式
├── custom.js               # 主题运行时逻辑
├── os-icons.js             # 内联 Linux/系统 SVG 图标
├── logo.png                # 站点图标
├── assets/                 # 原版主题构建产物
├── linux-logo-icon/        # 原始系统 SVG 图标
└── NodeGet-OrbitDeck.zip   # 可上传的主题压缩包
```

## 安装

1. 使用仓库中的 `NodeGet-OrbitDeck.zip`。
2. 在 NodeGet 后台上传或替换主题文件。
3. 确认 `config.json` 中的 `site_tokens` 已配置正确的后端地址和 token。

也可以自行重新打包：

```powershell
Compress-Archive -Path index.html,config.json,custom.css,custom.js,os-icons.js,logo.png,assets,linux-logo-icon -DestinationPath NodeGet-OrbitDeck.zip -Force
```

## 配置

`config.json` 示例：

```json
{
  "site_name": "Orbit Deck",
  "site_logo": "",
  "theme_name": "orbit-deck",
  "theme_version": "1.1.0",
  "theme_repo": "",
  "theme_config": {
    "version": "1.1.0",
    "footer": "Orbit Deck for NodeGet"
  },
  "site_tokens": [
    {
      "name": "master server node 1",
      "backend_url": "wss://your-nodeget-backend.example.com",
      "token": "YOUR_BACKEND_TOKEN"
    }
  ]
}
```

不要把真实 token 提交到公开仓库。

## 刷新逻辑

主题会通过 WebSocket JSON-RPC 连接 NodeGet 后端：

- 初始加载：读取节点 UUID、元数据、静态系统信息。
- 动态刷新：每 2 秒查询一次动态资源数据。
- 局部更新：普通刷新只更新卡片和详情页中的数值、进度条、趋势图和更新时间。
- 完整渲染：仅在搜索、筛选、标签、打开/关闭详情、可见节点列表变化时触发。

这样可以避免传统 `innerHTML` 整页重绘导致的页面跳动和输入框打断。

## 图标说明

系统图标来自 `linux-logo-icon/`，并在构建时写入 `os-icons.js`。运行时由 `custom.js` 直接内联 SVG，因此不依赖 `./linux-logo-icon/*.svg` 能否被浏览器直接访问。

如果更新或新增图标，可以重新生成 `os-icons.js`：

```powershell
node -e "const fs=require('fs'); const path=require('path'); const dir='linux-logo-icon'; const icons={}; for (const file of fs.readdirSync(dir)) { if (file.endsWith('.svg')) icons[path.basename(file,'.svg')] = fs.readFileSync(path.join(dir,file),'utf8').replace(/<\?xml[^>]*>/,'').trim(); } fs.writeFileSync('os-icons.js', 'window.ORBIT_OS_ICONS = '+JSON.stringify(icons)+';\n', 'utf8');"
```

## 已知限制

- 延时曲线暂时移除。当前主题不调用 `crontab_get` 或 `task_query`，等 NodeGet 官方前端/接口稳定后再接回。
- 国家旗帜依赖节点的 `metadata_region` 文本或国家码；无法识别时会显示原始地区文本。
- 详情页趋势基于前端运行期间采集的最近 60 个动态点，刷新页面后会重新积累。

## 开发检查

修改脚本后建议运行：

```powershell
node --check custom.js
node --check os-icons.js
```

然后重新打包 `NodeGet-OrbitDeck.zip`。
