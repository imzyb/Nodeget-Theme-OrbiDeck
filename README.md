# NodeGet Orbit Deck Theme

Version: `1.1.2`

Orbit Deck 是一个为 NodeGet 探针系统设计的自定义主题。它通过 `custom.js` 接管前端运行时 UI，重新组织节点卡片、筛选、搜索、详情页、动态图表和延时曲线展示。

## 功能特性

- 全新的探针墙卡片布局，突出在线状态、系统、区域、资源占用、流量和更新时间。
- 节点详情页包含资源三环、近 120 秒趋势图、系统信息、网络与负载信息。
- 支持 Ping / TCP Ping 延时曲线，多个信源合并到同一张图中展示。
- 延时曲线支持鼠标悬停查看该时间戳下各信源延时。
- 动态数据局部更新，避免定时刷新造成整页跳动和输入框打断。
- 支持在线、压力、离线状态筛选。
- 支持标签高亮筛选，默认保留全部节点显示。
- 支持搜索节点名称、UUID、来源、地区、系统、标签等字段。
- 系统图标以内联 SVG 渲染，不依赖运行环境直接访问图标文件。
- 国家/地区字段会尽量匹配为国旗和国家码，台湾节点按中国国旗显示。

## 文件结构

```text
.
├── index.html              # 主题入口
├── config.json             # NodeGet 后端连接配置
├── custom.css              # Orbit Deck 样式
├── custom.js               # 主题运行时逻辑
├── os-icons.js             # 内联 Linux/系统 SVG 图标
├── logo.png                # 站点图标
├── assets/                 # 官方主题构建产物
├── linux-logo-icon/        # 原始系统 SVG 图标
└── NodeGet-OrbitDeck.zip   # 可上传的主题压缩包
```

## 安装

1. 使用仓库中的 `NodeGet-OrbitDeck.zip`。
2. 在 NodeGet 后台上传或替换主题文件。
3. 确认 `config.json` 中的 `site_tokens` 已配置正确的后端地址和 token。
4. 如果浏览器仍显示旧样式，请强制刷新页面或清理主题缓存。

也可以自行重新打包：

```powershell
Compress-Archive -Path README.md,index.html,config.json,custom.css,custom.js,os-icons.js,logo.png,assets,linux-logo-icon -DestinationPath NodeGet-OrbitDeck.zip -Force
```

## 配置示例

```json
{
  "site_name": "Orbit Deck",
  "site_logo": "",
  "theme_name": "orbit-deck",
  "theme_version": "1.1.2",
  "theme_repo": "",
  "theme_config": {
    "version": "1.1.2",
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

主题通过 WebSocket JSON-RPC 连接 NodeGet 后端：

- 初始加载读取节点 UUID、元数据、静态系统信息和动态资源信息。
- 动态资源默认每 2 秒局部更新一次。
- 延时曲线默认每 30 秒读取最近任务结果。
- 普通刷新只更新卡片和详情页中的数值、进度条、趋势图和更新时间。
- 搜索、筛选、标签、打开/关闭详情、节点列表变化时才触发完整渲染。

## 延时曲线

延时曲线通过 `crontab_get` 和 `task_query` 读取 Ping / TCP Ping 任务结果。详情页会按任务类型分组，同一类型的多个信源合并到一张曲线图中。

鼠标悬停曲线时会显示：

- 当前时间戳
- 每个信源当时最接近的延时值
- 对应信源颜色的小圆点
- 竖向游标

## 图标说明

系统图标来自 `linux-logo-icon/`，并写入 `os-icons.js`。运行时由 `custom.js` 直接内联 SVG，因此不依赖 `./linux-logo-icon/*.svg` 是否能被浏览器直接访问。

如需重新生成 `os-icons.js`：

```powershell
node -e "const fs=require('fs'); const path=require('path'); const dir='linux-logo-icon'; const icons={}; for (const file of fs.readdirSync(dir)) { if (file.endsWith('.svg')) icons[path.basename(file,'.svg')] = fs.readFileSync(path.join(dir,file),'utf8').replace(/<\?xml[^>]*>/,'').trim(); } fs.writeFileSync('os-icons.js', 'window.ORBIT_OS_ICONS = '+JSON.stringify(icons)+';\n', 'utf8');"
```

## 开发检查

修改脚本后建议运行：

```powershell
node --check custom.js
node --check os-icons.js
```

然后重新打包 `NodeGet-OrbitDeck.zip`。
