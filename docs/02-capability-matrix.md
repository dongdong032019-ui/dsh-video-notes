# 能力规格卡（按官方文档 + 实测核对）

> 所有条目以 2026-10 的官方文档为准，括号内为实测结论。密钥一律用占位符。

## 1. Agnes（图像 / 视频）

- 中国站 base：`https://api.agnes-ai.cn`（`.com` 对同类 key 返回 401）
- 鉴权：`Authorization: Bearer <key>`

### 1.1 图像

```
POST /v1/images/generations
{ "model": "agnes-image-2.5-flash", "prompt": "...", "size": "1024x1024",
  "extra_body": { "response_format": "url" } }
```

- `response_format` **必须**放在 `extra_body` 内，放顶层报错
- 图生图/多图合成：`extra_body.image = ["https://…", "data:image/png;base64,…"]`，不要传 `tags:["img2img"]`
- 返回 `data[0].url` 或 `data[0].b64_json`；建议超时 60–360 s
- 计费：当前促销 ¥0（1K–4K 输出、第 4 张起的参考图均为 0）

### 1.2 视频（异步，按秒计费）

```
POST /v1/videos
{ "model": "agnes-video-2.5-flash", "prompt": "...", "mode": "text",
  "seconds": "5", "size": "720P", "aspect_ratio": "16:9" }
→ video_id
GET  /agnesapi?video_id=<id>&model_name=agnes-video-2.5-flash
→ status=completed → url
```

- `mode`：`text` | `keyframe`（首/尾帧）| `reference`（图/音/视频参考）
- `seconds`：字符串 "4"–"12"；`n` 固定 1
- `size`：720P / 1080P / 1K / 2K；`aspect_ratio`：21:9 16:9 4:3 1:1 3:4 9:16
- 价格：720P ¥0.15/s、1080P 与 1K ¥0.25/s、2K ¥0.35/s；输入视频时长并入计费
- 查询必须带 `model_name`（非 text 模式尤其），轮询 1–2 s 且要处理 429

### 1.3 账号分组

`GET /v1/models` 才是真相。常见差异：`agnes-image-2.0-flash` 可能不在分组内（`model_not_found`），`agnes-video-2.5` 可能额度为 0（403 `insufficient_user_quota`），此时改用 `-flash` 变体。

## 2. AnySearch（搜索 provider）

- 插件：`@anysearch/anysearch-dsh`；安装进 DSH profile 即可接管 `web_search` / `web_fetch`
- 无 key 走匿名额度；配 key 后 **1000 次/天**、20 QPS（开发者/学生认证可翻倍）
- key 存放：`$DSH_HOME/.credentials.yaml` → `ANYSEARCH_API_KEY: "as_sk_…"`（插件每次请求重新解析，改完不必重启）
- 追加工具：`anysearch_capabilities`（垂类目录）、`anysearch_search`、`anysearch_batch_search`
- base：`https://api.anysearch.com`；`/v1/domains` 带正确 key 返回 `{"code":0}`，带假 key 返回 401 —— 可用来验证 key 有效性；没有 `/v1/usage` 类余额接口

## 3. Agents Anywhere（手机远控）

- Windows 客户端 + 手机 App + Cloud 账号，四段链路：
  `手机 App → AA Cloud → 本机 Connector → DSH Bridge → DSH 会话`
- 手机显示“设备在线 / 0 个 Agent 在线”= Connector 起来了但没找到 DSH 桥接端点
- 端点文件两边路径不同（见 `dsh-mobile-bridge-fix`），且端口随 DSH 重启变化
- DSH 侧需要 `@agents-anywhere/dsh-bridge-next`；peer 要求 `@deepseek-ai/dsh-typert-protocol >=0.2.0-rc.1 <0.3.0-0`
- 桌面端与 DSH 内建远控都会管理 Connector，**只用一边**

## 4. 计费与额度速查

| 项目 | 口径 |
|---|---|
| Agnes 图像 | 当前 ¥0 |
| Agnes 视频 | ¥0.15–0.35 / 秒 |
| AnySearch | 1000 次 / 天（认证×2），20 QPS |
| DeepSeek 对话 | 按 token，峰谷价不同 |

## 5. 一条通用心法

外部能力接进 agent，优先走“**官方插件 / provider 适配层**”，其次才是“让 agent 自己写脚本调 API”。前者能被宿主统一管理（凭据、热加载、卸载），后者只是临时胶水。
