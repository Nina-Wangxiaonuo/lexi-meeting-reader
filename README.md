# LEXI · 会议里哪里不会点哪里

一个只聚焦会议纪要阅读的轻量原型：用户主动选中不理解的词或句子后，才出现解释入口。应用不会预先识别或标记全部 AI 术语。

## 当前版本包含

- 粘贴并载入一份会议纪要
- 左侧会议纪要存档目录，支持新增、切换和本地保存
- 左侧保留知识图谱入口
- 鼠标拖选一个词或同一段中的一句话
- 选区旁出现“解释一下 / 加入图谱”工具栏
- 基于当前段落上下文展示 Mock 解释
- 对已解释内容保留艺术化手绘划线
- 支持两个快捷追问和自由输入追问
- 追问以 Chat 气泡形式保留多轮历史，并按会议与选区存档
- 追问区提供“允许联网搜索”开关，搜索结果以来源链接显示
- 使用 `localStorage` 保存用户载入的会议、已划线 HTML 和聊天记录
- 保留知识图谱入口（当前为占位，下一阶段接入真实节点）
- 左侧模型设置入口，可输入自己的 DeepSeek API Key
- 配置 Key 后，解释请求会把当前会议全文一起交给 DeepSeek；没有 Key 时继续使用 Mock 演示模式

## 产品边界

这不是笔记软件，也不会自动给会议正文标满知识点。核心流程只有：

1. 阅读会议纪要；
2. 主动选中自己不懂的内容；
3. 点击“解释一下”；
4. 根据会议上下文理解；
5. 需要时再加入知识图谱。

## 本地运行

这是一个无构建工具的静态项目。直接打开 `index.html` 即可，也可以启动静态服务器：

```bash
python3 -m http.server 8080
```

然后访问 `http://localhost:8080`。

## 部署到 GitHub Pages

1. 在 GitHub 新建一个仓库，例如 `lexi-meeting-reader`。
2. 将本目录中的文件推送到仓库默认分支：

```bash
git init
git add index.html README.md .nojekyll
git commit -m "feat: add meeting selection explainer"
git branch -M main
git remote add origin https://github.com/<你的用户名>/lexi-meeting-reader.git
git push -u origin main
```

3. 打开仓库的 **Settings → Pages**。
4. 在 **Build and deployment** 中选择：
   - Source: `Deploy from a branch`
   - Branch: `main`
   - Folder: `/ (root)`
5. 保存后等待 GitHub Pages 完成部署。

访问地址通常是：

```text
https://<你的用户名>.github.io/lexi-meeting-reader/
```

## DeepSeek 1.0 与安全边界

当前页面支持两种模式：

- 没有 Key：使用 Mock 解释，可以直接体验完整交互；
- 在左侧“模型设置”输入 Key：调用 `https://api.deepseek.com/chat/completions`，模型会读取当前会议全文后解释选区。

API Key 不会写入 `index.html` 或 GitHub 仓库。默认只保存在当前浏览器会话；只有勾选“在此浏览器记住 Key”时才会写入浏览器本地存储。不要在公共电脑上勾选，也不要把 Key 发给别人。

当前直连模式适合个人本地测试，因为浏览器请求会暴露你自己的 Key。正式多人部署时，应改为服务端代理。

### 联网搜索

`deepseek-chat` 的普通对话接口本身不会自动浏览网页。页面中的“允许联网搜索”会调用你在“模型设置”中填写的搜索代理地址。没有配置代理时，开关不会伪装成已联网，而是提示先完成配置。

搜索代理约定：

```text
POST <你的搜索代理地址>
Content-Type: application/json
```

请求：

```json
{
  "query": "用户最新问题",
  "selectedText": "当前划词",
  "meetingTitle": "当前会议标题"
}
```

返回：

```json
{
  "results": [
    {
      "title": "结果标题",
      "url": "https://example.com/page",
      "snippet": "搜索摘要"
    }
  ]
}
```

前端最多取 5 条结果，将摘要作为会议外部补充交给 DeepSeek，并在回答下方显示来源。系统提示词要求模型区分“会议原文”和“联网补充”，不能把搜索内容说成会议中已经发生的事实。

推荐下一阶段使用：

- GitHub：代码仓库
- Vercel / Cloudflare Workers：服务端 API
- Supabase：会议、选区、对话和知识图谱数据
- DeepSeek / OpenAI 兼容 API：由服务端调用

建议的服务端接口：

```text
POST /api/explain
```

请求体：

```json
{
  "meetingId": "meeting_001",
  "selectedText": "RAG",
  "paragraphContext": "需要查资料的走 RAG 这条链路。",
  "meetingText": "完整会议纪要"
}
```

## 下一步

- 将浏览器直连改为服务端 `/api/explain` 和 `/api/chat` 代理
- 将搜索代理接到服务端（例如使用 Tavily、SerpAPI 或自建搜索服务）
- 保存解释记录和追问对话
- 用户主动点击“加入图谱”时才创建知识节点
- 单独制作知识图谱页面，不侵入会议阅读主流程

## 提示词设计

模型能力边界、结构化输出格式和第一版系统提示词草案参见 `PROMPTS.md`。核心原则是：用户没有主动选中内容时，不调用模型，也不预测用户会有多少个陌生词。
