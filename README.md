# FreshKeep - 食品保质期记录2

FreshKeep 是一款简洁、现代的食品保质期管理工具。它可以帮助您轻松记录食品的过期时间，并通过直观的状态提醒，确保您在食品过期前及时处理，减少浪费。

![FreshKeep Preview](https://via.placeholder.com/1200x600?text=FreshKeep+Food+Expiration+Tracker)

## ✨ 主要功能

- **🚀 快速录入**：极简的输入界面，快速添加食品名称、过期日期及提醒天数。
- **📊 智能分类**：自动将食品标记为“新鲜”、“临期”或“已过期”，并按时间排序。
- **🔍 搜索与筛选**：支持关键词搜索及状态筛选（全部、临期、过期）。
- **🔔 系统通知**：支持浏览器原生通知，在应用打开或处于后台时提醒您注意临期食品。
- **💾 本地持久化**：所有数据均存储在浏览器的 LocalStorage 中，无需登录，隐私安全。
- **📱 响应式设计**：适配移动端与桌面端，提供流畅的交互体验。

## 🛠️ 技术栈

- **框架**: [Vue 3](https://vuejs.org/) (Composition API)
- **构建工具**: [Vite](https://vitejs.dev/)
- **图标**: [Lucide Vue Next](https://lucide.dev/guide/packages/lucide-vue-next)
- **样式**: 原生 CSS (采用现代设计语言，如毛玻璃效果、流畅过渡动画)

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/your-username/freshkeep.git
cd freshkeep
```

### 2. 安装依赖

```bash
npm install
```

### 3. 启动开发服务器

```bash
npm run dev
```

启动后，在浏览器访问 `http://127.0.0.1:5173`。

### 4. 项目打包

```bash
npm run build
```

打包后的文件将生成在 `dist` 目录中。

## 📝 使用说明

1. **添加食品**：在左侧（或顶部）面板输入食品名称，选择过期日期。
2. **设置提醒**：默认提前 3 天提醒，您可以根据需要调整。
3. **开启通知**：点击“允许通知”按钮，FreshKeep 会在食品临期或过期时弹出系统提醒。
4. **数据清理**：点击食品卡片右侧的垃圾桶图标即可删除记录。

## 🛡️ 隐私声明

FreshKeep 完全在您的浏览器本地运行。我们不会收集、存储或上传您的任何个人数据或食品记录。

## 📄 开源协议

本项目采用 [MIT License](LICENSE) 许可。
