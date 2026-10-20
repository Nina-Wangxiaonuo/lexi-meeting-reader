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
- 使用 `localStorage` 保存用户载入的会议与已划线 HTML
- 保留知识图谱入口（当前为占位，下一阶段接入真实节点）

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

## 接入真实 AI 时的安全要求

当前页面使用 Mock 解释，故可以直接部署到 GitHub Pages。

接入 DeepSeek、OpenAI 或其他模型时，**不要把 API Key 写入 `index.html` 或前端 JavaScript**。GitHub Pages 是纯静态托管，任何前端变量都会暴露给访问者。

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

- 将 Mock 解释替换为真实模型接口
- 保存解释记录和追问对话
- 用户主动点击“加入图谱”时才创建知识节点
- 单独制作知识图谱页面，不侵入会议阅读主流程

## 提示词设计

模型能力边界、结构化输出格式和第一版系统提示词草案参见 `PROMPTS.md`。核心原则是：用户没有主动选中内容时，不调用模型，也不预测用户会有多少个陌生词。
