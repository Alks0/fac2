# Factory AI Proxy

一个 OpenAI 兼容的反向代理服务器，支持将请求转发到 Factory AI 和 Claude API

## 功能特性

- **多端点支持**
  - OpenAI 兼容格式端点：`/v1/chat/completions`
  - Claude 原生格式端点：`/v1/messages`

- **多模型支持**
  - Factory AI 模型（通过 OpenAI 端点）
  - Claude 系列模型（两个端点都支持）
  - Bedrock 模型（自动识别并处理）

- **特殊功能**
  - Claude 思考模式：模型名包含 `-thinking` 后缀时自动启用（16k thinking tokens）
  - 1M token context 支持（通过 anthropic-beta header）
  - 流式响应支持
  - 多模态支持（文本和图片）
  - 工具调用支持（自动转换格式）

## 安装

1. 克隆仓库
```bash
git clone <repository-url>
cd fac2
```

2. 安装依赖
```bash
npm install
```

3. 配置环境变量
复制 `.env.example` 到 `.env` 并填写配置：
```bash
cp .env.example .env
```

## 配置说明

在 `.env` 文件中配置以下环境变量：

```env
# Factory AI API 密钥（多个用分号分隔）
FACTORY_API_KEYS=your_key1;your_key2;your_key3

# 代理访问密钥（可选，用于限制访问）
PROXY_ACCESS_KEYS=proxy_key1;proxy_key2

# 自定义代理密钥请求头（默认：X-Proxy-Key）
PROXY_KEY_HEADER=X-Proxy-Key

# 代理端口（默认：8001）
PROXY_PORT=8001

# 调试上游请求（可选）
DEBUG_UPSTREAM=true
```

## 运行

### 开发模式
```bash
npm run dev
```

### 生产模式
```bash
npm run build
npm start
```

## API 使用

### OpenAI 兼容格式端点

**端点**: `POST http://localhost:8001/v1/chat/completions`

**请求示例**:
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "stream": false,
  "max_tokens": 1000
}
```

**支持的特性**:
- 思考模式：使用 `claude-3-5-sonnet-20241022-thinking` 模型名
- 文件上传：支持 multipart/form-data 格式
- 工具调用：OpenAI 格式的 tools 会自动转换为 Claude 格式

### Claude 原生格式端点

**端点**: `POST http://localhost:8001/v1/messages`

**请求示例**:
```json
{
  "model": "claude-3-5-sonnet-20241022",
  "messages": [
    {"role": "user", "content": "Hello!"}
  ],
  "max_tokens": 1000,
  "system": "You are a helpful assistant"
}
```

**特性**:
- 直接支持 Claude 原生请求格式
- 保留所有原始功能（tools、thinking 等）

## 认证方式

1. **使用自己的 API Key**
   ```bash
   Authorization: Bearer your-api-key
   ```

2. **使用配置的 Factory API Keys**
   - 不提供 Authorization header，系统会自动轮询使用配置的密钥

3. **使用代理访问密钥**（如果配置了 PROXY_ACCESS_KEYS）
   ```bash
   X-Proxy-Key: your-proxy-key
   # 或
   Authorization: Bearer your-proxy-key
   ```


## 工具脚本

- `factory.ps1` - 检查 Factory API 密钥使用情况

## 注意事项

1. **安全性**：请妥善保管 API 密钥，不要将包含真实密钥的 `.env` 文件提交到版本控制系统
2. **合规性**：本项目通过注入特定系统提示词来绕过某些限制，请合理使用
3. **错误处理**：如遇到 403 错误，可能是系统提示词被检测，请检查日志中的提示词内容

## 技术栈

- TypeScript
- Node.js (原生 HTTP 模块)
- ESM 模块系统
