```json
{
  "ClientAddr": "10.0.3.36:58748",
  "ClientHost": "10.0.3.36",
  "ClientPort": "58748",
  "ClientUsername": "-",
  "DownstreamContentSize": 15406,
  "DownstreamStatus": 200,
  "Duration": 179024,
  "GzipRatio": 0,
  "OriginContentSize": 0,
  "OriginDuration": 0,
  "OriginStatus": 0,
  "Overhead": 179024,
  "RequestAddr": "10.0.1.7:8080",
  "RequestContentSize": 0,
  "RequestCount": 10,
  "RequestHost": "10.0.1.7",
  "RequestMethod": "GET",
  "RequestPath": "/dashboard/favicon.ico",
  "RequestPort": "8080",
  "RequestProtocol": "HTTP/1.1",
  "RequestScheme": "http",
  "RetryAttempts": 0,
  "RouterName": "dashboard@internal",
  "StartLocal": "2026-04-17T14:40:19.888831613+08:00",
  "StartUTC": "2026-04-17T06:40:19.888831613Z",
  "downstream_Accept-Ranges": "bytes",
  "downstream_Content-Length": "15406",
  "downstream_Content-Security-Policy": "frame-src 'self' https://traefik.io https://*.traefik.io;",
  "downstream_Content-Type": "image/x-icon",
  "entryPointName": "traefik",
  "level": "info",
  "msg": "",
  "request_Accept": "image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8",
  "request_Accept-Encoding": "gzip, deflate",
  "request_Accept-Language": "zh-CN,zh;q=0.9",
  "request_Referer": "http://10.0.1.7:8080/dashboard/",
  "request_User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36",
  "request_X-Forwarded-Host": "10.0.1.7:8080",
  "request_X-Forwarded-Port": "8080",
  "request_X-Forwarded-Prefix": "/dashboard/",
  "request_X-Forwarded-Proto": "http",
  "request_X-Forwarded-Server": "kubernetes-7",
  "request_X-Real-Ip": "10.0.3.36",
  "time": "2026-04-17T14:40:19+08:00"
}
 ```

### 一、客户端信息（Client 开头）

- **ClientAddr**: `10.0.3.36:58748`
  客户端完整地址（IP:端口）
- **ClientHost**: `10.0.3.36`
  客户端 IP（仅主机）
- **ClientPort**: `58748`
  客户端发起请求的端口
- **ClientUsername**: `-`
  客户端认证用户名（无则为 `-`）

### 二、请求信息（Request 开头）
- **RequestAddr**: `10.0.1.7:8080`
  请求目标地址（Host:端口）
- **RequestHost**: `10.0.1.7`
  请求主机名（不含端口）
- **RequestPort**: `8080`
  请求端口
- **RequestMethod**: `GET`
  HTTP 请求方法（GET/POST/PUT/DELETE 等）
- **RequestPath**: `/dashboard/favicon.ico`
  请求路径（URI）
- **RequestProtocol**: `HTTP/1.1`
  HTTP 协议版本
- **RequestScheme**: `http`
  协议（http / https）
- **RequestContentSize**: `0`
  请求体大小（字节），GET 通常为 0
- **RequestCount**: `10`
  Traefik 启动后累计请求数

### 三、响应/下游信息（Downstream 开头）
- **DownstreamStatus**: `200`
  返回给客户端的 HTTP 状态码
- **DownstreamContentSize**: `15406`
  返回给客户端的响应体大小（字节）
- **downstream_*** 开头（响应头）
  - `downstream_Accept-Ranges`: `bytes`
  - `downstream_Content-Length`: `15406`
  - `downstream_Content-Type`: `image/x-icon`
  - 其他：返回给客户端的 HTTP 响应头

### 四、上游/源服务（Origin 开头）
- **OriginStatus**: `0`
  源服务（上游）返回的状态码（0 表示未转发到上游，Traefik 内部处理）
- **OriginContentSize**: `0`
  源服务返回内容大小
- **OriginDuration**: `0`
  源服务处理耗时（纳秒）

### 五、时间与耗时
- **StartUTC**: `2026-04-17T06:40:19.888831613Z`
  请求开始时间（UTC）
- **StartLocal**: `2026-04-17T14:40:19.888831613+08:00`
  请求开始时间（本地时区）
- **Duration**: `179024`
  **总耗时（纳秒）**：179,024 ns ≈ **0.179 ms**
- **Overhead**: `179024`
  Traefik 自身处理开销（纳秒），此处等于总耗时（无上游）
- **time**: `2026-04-17T14:40:19+08:00`
  日志记录时间（本地）

### 六、Traefik 内部信息
- **RouterName**: `dashboard@internal`
  匹配的路由器名称（内部 dashboard 路由）
- **entryPointName**: `traefik`
  入口点名称
- **RetryAttempts**: `0`
  重试次数
- **GzipRatio**: `0`
  Gzip 压缩比例（0 表示未压缩）

### 七、请求头（request_* 开头）
- **request_Accept**: 客户端接受的响应类型
- **request_Accept-Encoding**: 支持的编码（gzip）
- **request_Accept-Language**: 语言偏好
- **request_Referer**: 来源页面
- **request_User-Agent**: 客户端浏览器/设备信息
- **request_X-Forwarded-***: 代理转发头（IP/Host/Port/Proto 等）
- **request_X-Real-Ip**: 真实客户端 IP

### 八、日志元信息
- **level**: `info`
  日志级别
- **msg**: `""`
  日志消息（空）

---

### 本条日志总结
- 客户端 `10.0.3.36` 通过 HTTP/1.1 GET 请求 Traefik 内部 Dashboard 的 `/dashboard/favicon.ico`
- Traefik 直接内部处理（**无上游转发**，Origin 全 0）
- 总耗时 **0.179ms**，返回 200 + 15KB 图标
- 属于正常的内部访问日志


 

 ```json
 {
  "ClientHost": "10.0.3.36",
  "DownstreamStatus": 404,
  "Duration": 126189,
  "RequestAddr": "10.0.1.7",
  "RequestHost": "10.0.1.7",
  "RequestMethod": "GET",
  "RequestPath": "/",
  "RetryAttempts": 0,
  "StartLocal": "2026-04-17T15:01:38.041376818+08:00",
  "entryPointName": "web",
  "level": "info",
  "msg": "",
  "request_User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36",
  "time": "2026-04-17T15:01:38+08:00"
}
```
