# MyCV - 个人简历在线编辑器

一款基于 Vue3 + Element Plus 开发的个人简历在线编辑工具，支持实时预览、PDF 导出、拖拽排序等功能。

![Vue3](https://img.shields.io/badge/Vue-3.4+-green.svg)
![Element Plus](https://img.shields.io/badge/Element%20Plus-2.5+-blue.svg)
![Vite](https://img.shields.io/badge/Vite-5.0+-yellow.svg)
![License](https://img.shields.io/badge/License-MIT-orange.svg)

## ✨ 功能特性

### 📝 编辑功能

- **实时预览**：编辑内容实时同步到预览区域
- **多模块管理**：支持基本信息、个人简介、技术栈、教育背景、证书荣誉、工作经历、项目经验七大模块
- **拖拽排序**：各模块内容支持拖拽排序，带有可视化插入位置指示
- **动态增删**：支持添加、删除各模块的条目

### 🎨 界面特性

- **响应式设计**：适配不同屏幕尺寸
- **现代化 UI**：采用 Element Plus 组件库，界面简洁美观
- **绿色主题**：专业的简历配色方案
- **分页预览**：支持显示 PDF 分页线，方便调整内容布局

### 📄 导出功能

- **PDF 导出**：一键导出高质量 PDF 简历
- **智能分页**：自动处理多页 PDF 的分页逻辑
- **图片支持**：导出的 PDF 包含头像等图片内容

## 🚀 技术栈

- **前端框架**：Vue 3.4+
- **UI 组件库**：Element Plus 2.5+
- **构建工具**：Vite 5.0+
- **PDF 生成**：html2canvas + jsPDF
- **图标库**：@element-plus/icons-vue

## 📦 安装与运行

### 环境要求

- Node.js 16+
- npm 或 yarn

### 安装步骤

1. **克隆项目**

```bash
git clone https://github.com/yourusername/MyCV.git
cd MyCV
```

2. **安装依赖**

```bash
npm install
```

3. **启动开发服务器**

```bash
npm run dev
```

4. **构建生产版本**

```bash
npm run build
```

## 📖 使用指南

### 编辑简历

1. 点击右上角"编辑模式"按钮进入编辑状态
2. 在各标签页中填写或修改简历内容
3. 点击"预览模式"查看效果

### 拖拽排序

- **技术栈分类**：拖拽分类标题进行排序
- **工作经历**：拖拽整个工作经历区块调整顺序
- **项目经验**：拖拽项目区块调整顺序
- **教育背景**：拖拽教育经历区块调整顺序
- **证书荣誉**：拖拽证书区块调整顺序

> 💡 拖拽时会显示蓝色插入指示线，清晰标示插入位置

### 导出 PDF

1. 点击右上角"导出 PDF"按钮
2. 等待生成完成（约 1-2 秒）
3. PDF 文件将自动下载到本地

### 显示分页线

点击"显示分页线"按钮，页面将显示红色的分页标记线，帮助您了解内容在 PDF 中的分页位置，便于调整内容布局。

## 🏗️ 项目结构

```
MyCV/
├── public/                 # 静态资源
│   └── cv-logo.svg        # 网站图标
├── src/
│   ├── App.vue            # 主应用组件
│   ├── main.js            # 入口文件
│   └── style.css          # 全局样式
├── index.html             # HTML 模板
├── package.json           # 项目配置
├── vite.config.js         # Vite 配置
└── README.md              # 项目说明
```

## 🎯 核心功能实现

### 拖拽排序

采用 HTML5 Drag and Drop API 实现，支持：

- 基于鼠标 Y 坐标的智能插入位置计算
- 可视化插入指示器
- 流畅的拖拽动画效果

### PDF 导出

使用 html2canvas + jsPDF 实现：

- 高清晰度截图（scale: 2）
- 跨域图片处理（CORS）
- 多页 PDF 自动分页

### 响应式数据绑定

使用 Vue 3 Composition API：

- `ref` 和 `reactive` 管理状态
- `computed` 计算属性
- 实时双向数据绑定

## 📝 数据格式

简历数据采用 JSON 格式存储，主要结构如下：

```javascript
{
  basic: {
    name: "姓名",
    title: "职位",
    subtitle: "副标题",
    email: "邮箱",
    phone: "电话",
    location: "地址",
    github: "GitHub",
    avatar: "头像URL"
  },
  introduction: "个人简介",
  skillCategories: [...],  // 技术栈分类
  education: [...],        // 教育背景
  certificates: [...],     // 证书荣誉
  workExperience: [...],   // 工作经历
  projects: [...],         // 项目经验
  skillProgress: [...]     // 技能熟练度
}
```

## 🔧 自定义配置

### 修改主题颜色

在 `src/App.vue` 中修改 CSS 变量：

```css
.header-section {
  background: linear-gradient(135deg, #10b981 0%, #059669 100%);
}
```

### 添加新模块

1. 在 `resumeData` 中添加新数据结构
2. 在编辑面板添加表单
3. 在预览区域添加展示组件

## 📄 许可证

[MIT](LICENSE) License © 2024 [Your Name]

## 🤝 贡献指南

欢迎提交 Issue 和 Pull Request！

1. Fork 本项目
2. 创建您的特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开一个 Pull Request

## 📮 联系方式

- 邮箱：your.email@example.com
- GitHub：[yourusername](https://github.com/yourusername)

---

如果这个项目对您有帮助，请给个 ⭐ Star 支持一下！
