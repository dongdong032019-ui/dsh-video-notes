# dsh-video-notes

Structured notes taken while reverse-engineering a Chinese walkthrough titled *“把 DeepSeek Harness 打造成满配桌面端”* (7 minutes, 1080p). The video is a screen recording with burned-in subtitles; everything here was reconstructed by frame sampling + subtitle extraction, then **verified against vendor docs and the live APIs** — where the video and reality disagreed, reality wins and the difference is recorded.

```
docs/01-video-breakdown.md    chapter timeline, the six-step recipe, the traps
docs/02-capability-matrix.md  Agnes / AnySearch / Agents Anywhere, checked against the APIs
```

## What the video actually claims

Ship a “fully equipped” DSH desktop in six moves:

1. install DSH Desktop
2. fill gaps from its plugin market
3. pair a phone through Agents Anywhere for remote control
4. switch the model to get native vision (no plugin needed)
5. wire up Agnes for free image and paid-per-second video
6. replace the built-in web search with AnySearch

Four gaps — see, draw, search, remote — each closed with the cheapest working option. That framing is the useful part; the specific menu labels are not, because the video was recorded on a different DSH build line.

## Corrections the video needs

| Video says | Reality |
|---|---|
| “image and video are both free” | image is ¥0 during the promotion; **video bills per second** (720P ¥0.15/s … 2K ¥0.35/s) |
| the agent calls `api.agnes-ai.com` | China-region traffic goes to `api.agnes-ai.cn`; the `.com` host returns `401` for these keys |
| “install the 7 essential plugins” | that list came from a search-result HTML page generated inside the demo, not from any official catalogue |
| a single V4.1 model covers vision | true, but the same account may not be entitled to every image/video model — `model_not_found` / `insufficient_user_quota` are common; list `/v1/models` first |

## Method (reproducible)

Frames sampled at 1 s with a mean-abs-difference dedupe (257 kept), bottom subtitle band cropped into 2 s strips for reading, dark chapter slides isolated by brightness/saturation thresholds. No ASR needed: the subtitles are burned in.

## License

MIT
