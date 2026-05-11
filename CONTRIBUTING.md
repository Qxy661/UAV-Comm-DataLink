# 贡献指南

感谢你对本项目的关注！以下是参与贡献的方式。

---

## 如何贡献

### 提交 Issue

- **Bug 报告：** 发现文档错误、链接失效、代码示例无法运行等问题
- **内容建议：** 希望补充的主题、论文推荐、技术点扩展
- **学习反馈：** 学习过程中的困惑点、难以理解的章节

### 提交 Pull Request

1. Fork 本仓库
2. 创建特性分支：`git checkout -b feature/your-feature`
3. 提交更改：`git commit -m "Add: 简要描述"`
4. 推送到远程：`git push origin feature/your-feature`
5. 创建 Pull Request

---

## 文档规范

### 格式要求

- 每篇文档以 `# 标题` 开头，紧跟 `> 预计阅读：X 分钟 | 前置知识：xxx`
- 使用 `---` 分隔主要章节
- 章节编号使用 `## 1.`、`## 2.` 格式
- 表格、Mermaid 图表、ASCII art 适当使用
- 结尾包含 `思考题` 及可折叠的 `<details><summary>参考答案</summary>`

### 语言规范

- 中文为主，英文技术术语保留原文
- 首次出现的缩写需给出全称，如：MAVLink（Micro Air Vehicle Link）
- 代码注释使用英文

### 代码示例

- Python 代码使用 Python 3.8+ 语法
- C/C++ 代码注明编译器版本
- 所有代码示例需可运行或明确标注为伪代码

---

## 本地预览

```bash
# VS Code 安装插件
# - Markdown Preview Enhanced
# - Markdown Mermaid

# 或使用 mdbook
cargo install mdbook
mdbook serve
```

---

## 行为准则

- 尊重每位贡献者
- 技术讨论以事实和数据为依据
- 欢迎不同意见，保持建设性讨论
