## 系统架构图(AI生成，非100%完整)

```mermaid
graph TD
    subgraph 微信生态
        User[微信用户]
        WXServer[微信公众号服务器]
    end

    subgraph 服务1: 消息接收与预处理器
        Receiver[app.py /wechatrevice]
        Note1[验证签名 / 接收XML<br/>立即返回success<br/>守护线程异步转发]
    end

    subgraph 服务2: 核心推理引擎
        API[app.py /inference]
        Agent[functionSetInference.py<br/>多Agent链]
        Note2[Summarizer → Statistician → Sensor →<br/>Selecter → Responder]
    end

    subgraph 服务3: 定时调度与批量推送器
        Scheduler[index.py main_handler]
        Note3[收集日志 → 合并去重 → 并发推理 →<br/>微信客服发送 → 扣配额/归档]
    end

    subgraph 腾讯云COS 对象存储
        COS1[用户对话模板]
        COS2[原始消息日志]
        COS3[用户文本备份]
        COS4[聊天记录]
        COS5[待处理消息DF]
        COS6[语音文件]
        COS7[用户配额]
    end

    User <-->|消息/语音/事件| WXServer
    WXServer -->|POST XML 带签名| Receiver
    Receiver -->|立即返回success| WXServer
    Receiver -.->|异步写日志/存语音| COS2
    Receiver -.->|异步写日志/存语音| COS6
    Scheduler -.->|读取日志| COS2
    Scheduler -->|并发请求推理| API
    API --> Agent
    Agent -.->|读写历史/状态| COS1
    Scheduler -.->|读写数据帧| COS5
    Scheduler -.->|读写配额| COS7
    Scheduler -->|调用微信客服接口| WXServer
    Scheduler -.->|更新聊天记录| COS4
    Scheduler -.->|备份/清理| COS3
