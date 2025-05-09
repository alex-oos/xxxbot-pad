# WX849 回调模式使用指南

本指南介绍如何设置和配置回调模式，使原始框架能将微信消息转发给 DOW 框架处理。此模式避免了两个框架同时轮询 API 造成的冲突，提高了稳定性和效率。

## 工作原理

回调模式的工作流程如下：

1. 原始框架接收微信消息
2. 原始框架通过回调脚本将消息转发给 DOW 框架
3. DOW 框架处理消息并响应
4. DOW 框架通过 API 接口发送回复

这种模式的优势：

- 避免两个框架同时轮询 API
- 降低服务器负载
- 减少消息处理延迟
- 提高整体稳定性
- 避免消息处理冲突

## 安装步骤

### 1. 启动 DOW 框架

首先启动 DOW 框架，它会自动启动 HTTP 服务器接收消息回调：

```bash
python app.py
```

在日志中会显示类似以下信息：

```
[WX849] HTTP服务器已启动，回调URL: http://127.0.0.1:8088/wx849/callback
[WX849] 请在原始框架配置文件中添加以下回调设置:
  "callback_url": "http://127.0.0.1:8088/wx849/callback",
  "callback_key": "xxxx-xxxx-xxxx-xxxx"
```

记下`callback_url`和`callback_key`，后续需要使用。

### 2. 安装回调发送脚本

将`wx849_callback_sender.py`脚本复制到原始框架的根目录下。

### 3. 创建回调配置文件

在原始框架根目录下创建`wx849_callback_config.json`文件，内容如下：

```json
{
  "callback_url": "http://127.0.0.1:8088/wx849/callback",
  "callback_key": "在此填入DOW框架生成的密钥"
}
```

将`callback_key`的值替换为 DOW 框架日志中显示的密钥。

### 4. 修改原始框架配置

打开原始框架的配置文件（通常是`config.json`），添加以下设置：

```json
"callback_path": "python wx849_callback_sender.py"
```

如果你的 Python 路径不同，需要相应调整命令。

### 5. 重启原始框架

完成上述配置后，重启原始框架使配置生效。

## 测试回调

你可以通过手动运行回调脚本来测试配置是否正确：

```bash
python wx849_callback_sender.py '{"MsgId":"12345678901234567","FromUserName":"wxid_test123","ToUserName":"filehelper","Content":"这是一条测试消息","Type":1,"CreateTime":1617960123}'
```

如果配置正确，你应该能在 DOW 框架的日志中看到收到了测试消息。

## 故障排除

如果回调不工作，请检查以下几点：

1. **检查 DOW 框架是否正常运行**：确保 DOW 框架已启动并显示 HTTP 服务器已启动的消息。

2. **检查配置文件**：确保`wx849_callback_config.json`中的 URL 和密钥正确。

3. **检查网络连接**：确保 DOW 框架和原始框架之间的网络连接正常。如果它们在不同的服务器上，确保防火墙允许相应端口的访问。

4. **检查日志**：查看`logs/wx849_callback_*.log`文件中的错误信息。

5. **手动测试**：使用上述测试命令手动测试回调脚本。

## 自定义配置

你可以通过编辑配置文件来自定义回调行为：

- **修改回调 URL**：如果 DOW 框架运行在不同的服务器或端口上，修改`callback_url`参数。
- **修改监听地址**：编辑 DOW 框架的配置文件，修改`wx849_callback_host`和`wx849_callback_port`参数。

## 安全注意事项

- 回调密钥用于验证请求的合法性，防止未授权访问，请妥善保管。
- 如果 DOW 框架和原始框架运行在不同的服务器上，建议使用 HTTPS 和防火墙规则增强安全性。
- 定期检查日志，监控异常活动。

## 常见问题

**Q: 为什么选择回调模式而不是原来的轮询模式？**

A: 回调模式避免了两个框架同时轮询 API 造成的冲突，显著提高了稳定性和效率。

**Q: 回调模式对性能有什么影响？**

A: 回调模式通常比轮询模式更高效，减少了不必要的 API 请求，降低了处理延迟。

**Q: 如何知道回调是否正常工作？**

A: 查看 DOW 框架日志中是否显示收到消息的记录，或者发送测试消息并检查响应。

**Q: 如果原始框架和 DOW 框架不在同一台服务器上，该如何配置？**

A: 需要将回调 URL 更改为 DOW 框架的公网 IP 或域名，并确保相应的端口开放和可访问。
