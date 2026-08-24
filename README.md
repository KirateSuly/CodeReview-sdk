# openai-code-review

基于 **GitHub Actions + 智谱 AI（ChatGLM）** 的自动代码评审工具。每当代码 push 或提交 PR 到 `master` 分支时，自动获取本次提交的 `git diff`，调用大模型对代码进行评审，将评审结果提交到日志仓库，并通过微信公众号推送通知。

## 核心流程

1. **获取变更** —— 读取最近一次提交的 `git diff`
2. **AI 评审** —— 调用智谱 ChatGLM（默认 `glm-5.2`）生成评审意见
3. **记录结果** —— 将评审结果以 Markdown 形式提交并推送到日志仓库
4. **消息通知** —— 通过微信公众号模板消息推送评审日志链接

## 项目结构

```
openai-code-review
├── openai-code-review-sdk      # 核心 SDK，打包成 jar 执行评审
├── openai-code-review-test     # 测试模块
└── .github/workflows
    ├── main-remote-jar.yml     # 下载远程 jar 运行（正式使用）
    ├── main-maven-jar.yml      # 本地 Maven 构建 jar 运行
    └── main-local.yml          # 本地 javac 编译运行（调试用）
```

## 使用方法

### 1. 前置准备

- 一个 **智谱 AI（open.bigmodel.cn）** 账号，用于获取 API Key
- 一个 **微信公众号测试号**，用于接收评审通知
- 一个用于**存放评审日志**的 GitHub 仓库

### 2. 配置 Secrets

进入仓库 **Settings → Secrets and variables → Actions → New repository secret**，添加以下密钥：

| Secret 名称 | 说明 | 获取方式 |
|---|---|---|
| `CODE_REVIEW_LOG_URI` | 存放评审日志的仓库地址 | 例如 `https://github.com/<your>/openai-code-review-log` |
| `CODE_TOKEN` | 有日志仓库读写权限的 GitHub Token | https://github.com/settings/tokens |
| `WEIXIN_APPID` | 微信公众号测试号 AppID | 微信公众平台测试号页 |
| `WEIXIN_SECRET` | 微信公众号测试号 AppSecret | 微信公众平台测试号页 |
| `WEIXIN_TOUSER` | 接收通知的 openid | 微信公众平台测试号页 |
| `WEIXIN_TEMPLATE_ID` | 模板消息 ID | 微信公众平台测试号页 |
| `CHATGLM_APIHOST` | ChatGLM 接口地址 | `https://open.bigmodel.cn/api/paas/v4/chat/completions` |
| `CHATGLM_APIKEYSECRET` | 智谱 API Key（`id.secret` 格式） | https://open.bigmodel.cn/usercenter/apikeys |

### 3. 启用

确认 `.github/workflows/main-remote-jar.yml` 中触发分支为 `master`，然后向 `master` 分支 push 代码或提交 PR，即可自动触发评审。


## 配置说明

所有配置均在运行时通过**环境变量**注入，不在代码中硬编码。字段含义如下：

| 环境变量 | 说明 |
|---|---|
| `GITHUB_REVIEW_LOG_URI` | 评审日志仓库地址 |
| `GITHUB_TOKEN` | 推送日志用的 GitHub Token |
| `COMMIT_PROJECT` / `COMMIT_BRANCH` | 仓库名 / 分支名（自动获取） |
| `COMMIT_AUTHOR` / `COMMIT_MESSAGE` | 提交者 / 提交信息（自动获取） |
| `WEIXIN_*` | 微信模板消息相关配置 |
| `CHATGLM_APIHOST` / `CHATGLM_APIKEYSECRET` | 智谱大模型接口地址与密钥 |

## 技术栈

- Java 8 / Maven / Spring Boot 2.7.12
- JGit（Git 操作）、fastjson2、auth0 JWT、Guava
- 智谱 AI ChatGLM API、微信公众号模板消息

## 支持的模型

默认使用 `glm-5.2`
