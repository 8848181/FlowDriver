# Flow Driver 🌊

**Flow Driver** 是一个隐蔽传输系统，设计用于通过常见云存储平台（如 Google Drive）隧道网络流量（SOCKS5）。它通过利用合法的 API 流量，在限制性环境中实现可靠通信。

***

## ⚠️ 免责声明

**中文**：本项目仅供个人使用和研究目的。请勿用于非法用途，也勿在生产环境中使用。作者不对任何滥用行为负责。

***

## 工作原理

### 中文
Flow Driver 通过将云存储文件夹当作数据队列来工作：
1. **客户端**：捕获本地的 SOCKS5 请求，将它们打包成紧凑的**二进制协议**。这些二进制“数据包”会被上传到指定的 Google Drive 文件夹。
2. **服务端**：持续轮询 Drive 文件夹。当发现客户端请求时，下载它，建立到目标的真实 TCP 连接，并将结果作为响应文件发回。

***

## 安装与配置

### 前置条件
- **Go**（1.25 或更高版本）
- **Google Drive API 凭证**：需要 `credentials.json`（OAuth2）文件
- **共享文件夹（自动）**：如果留空 `google_folder_id`，工具会自动创建名为 **"Flow-Data"** 的文件夹，并将 ID 保存到你的配置文件中！

### 1. 获取凭证
按照 [Google Drive API Go 快速入门](https://developers.google.com/workspace/drive/api/quickstart/go) 的说明，或按以下步骤操作：

**中文：**
1. **启用 API**：访问 [Google Cloud Console](https://console.cloud.google.com/)，创建项目，启用 **Google Drive API**。
2. **配置同意屏幕**：进入 "APIs & Services" > "OAuth consent screen"。填写应用名称和用户支持邮箱（品牌信息）。
3. **创建凭证**：进入 "Credentials" > "Create Credentials" > **OAuth client ID**。选择 **Desktop App** 作为应用类型。
4. **下载 JSON**：下载客户端密钥文件并重命名为 `credentials.json`。
5. **发布应用（可选但推荐）**：如果应用状态为 "Testing"，令牌将每 7 天过期。进入 OAuth 同意屏幕，点击 "Publish App" 使授权永久有效。

### 2. 编译二进制文件
```bash
go build -o bin/client ./cmd/client
go build -o bin/server ./cmd/server
```

### 3. 配置

基于提供的示例创建 `config.json`：

**客户端配置 (`client_config.json`)：**
```json
{
  "listen_addr": "127.0.0.1:1080",
  "storage_type": "google",
  "google_folder_id": "YOUR_FOLDER_ID",
  "refresh_rate_ms": 100,
  "flush_rate_ms": 300,
  "transport": {
    "TargetIP": "216.239.38.120:443",
    "SNI": "google.com",
    "HostHeader": "www.googleapis.com"
  }
}
```

**服务端配置 (`server_config.json`)：**
```json
{
  "storage_type": "google",
  "google_folder_id": "YOUR_FOLDER_ID",
  "refresh_rate_ms": 100,
  "flush_rate_ms": 300
}
```

### 4. 运行

**服务端：**
```bash
./bin/server -c server_config.json -gc credentials.json
```

**客户端：**
```bash
./bin/client -c client_config.json -gc credentials.json
```

***

## 性能与配额

**重要提示**：Google Drive 有严格的 API 速率限制（配额）。
- 使用很小的值（如 `refresh_rate_ms: 100`）会迅速耗尽 API 配额。
- 为避免连接被限制或封禁，建议始终保持这些值**高于 100ms**。
- 对于高负载或多并发用户，建议设置为**200ms 或更高**。

***

## 使用与认证

### 1. 首次认证
项目使用 OAuth2 "三脚架" 流程。只需在本地机器上执行一次：

**中文：**
1. 运行客户端：`./bin/client -c client_config.json -gc credentials.json`
2. 终端会出现一个链接。**复制并在浏览器中打开**。
3. 登录 Google 账号并授权权限。
4. 你将被重定向到一个以 `http://localhost` 开头的地址（页面无法加载也没关系）。
5. **复制浏览器地址栏的完整 URL**，粘贴回终端。
6. 程序会在 `credentials.json` 旁边创建 `.token` 文件。认证完成。

### 2. 部署到服务端
获取 `.token` 文件后，无需再次登录。

**中文：**
在远程上游机器运行服务端：
1. 将 `credentials.json` **和** `.token` 文件复制到服务端。
2. **关键**：确保 `server_config.json` 中的 `google_folder_id` 与客户端自动创建并保存到本地配置中的 ID **完全相同**。
3. 运行：`./bin/server -c server_config.json -gc credentials.json`
4. 服务端将自动使用现有令牌并立即启动。

***

这个翻译保持了**双语并列的原始结构**，并将所有英文内容完整翻译为简洁准确的中文。如果你需要纯中文版本、特定部分的详细解释，或将配置示例中的字段进一步注释，请告诉我！[1][2]

来源
[1] NullLatency/FlowDriver: Bypass restrictive networks by ... - GitHub https://github.com/NullLatency/FlowDriver
[2] client_config.json.example - NullLatency/FlowDriver - GitHub https://github.com/NullLatency/FlowDriver/blob/master/client_config.json.example
