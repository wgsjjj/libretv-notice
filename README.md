# libretv-notice

LibreTV TV 客户端的远程公告。App 启动时拉取本仓库的 `notice.json`：

- **普通公告** → 显示在首页顶部的滚动公告栏，多条轮换
- **重要公告** → 打开 App 直接弹窗，图文并排，点「我知道了」或按返回键后不再打扰

## 怎么改

1. 打开 `notice.json`，点右上角铅笔图标
2. 改内容，**顺便把 `updated` 改成当前时间**（格式 `2026-11-05 16:35:00`），保存
3. App 最多 30 分钟后自动更新，重开 App 会立即拉取

`updated` 不只是给人看的：App 会从四个镜像并发拉取，按这个时间挑最新的一份。忘了改也没关系（内容照样生效），但改一下能让新内容更快压过镜像的旧缓存。

`notice.json` 格式写错时 App 会直接忽略本次更新，继续用上一条缓存，不会白屏也不会报错。

## 字段

| 字段 | 说明 |
| --- | --- |
| `enabled` | `false` 时 App 不显示任何公告 |
| `updated` | 本文件的更新时间，`yyyy-MM-dd HH:mm:ss`。App 用它挑最新镜像 |
| `items[].id` | **重要公告必填**，这条的编号，确认过后同一 id 不再弹 |
| `items[].text` | 公告正文，纯文本。普通公告最多 200 字（换行会被压成空格）；重要公告最多 600 字，保留换行 |
| `items[].level` | `info` 蓝 / `warn` 黄 / `danger` 红 |
| `items[].popup` | `true` = 重要公告（启动弹窗）；不写或 `false` = 普通公告，只在滚动栏 |
| `items[].title` | 可选，弹窗顶部标题，留空显示「重要公告」，最多 20 字 |
| `items[].image` | 可选，弹窗右侧配图，放在与 `notice.json` 同一目录，写文件名即可（如 `wechat-qr.png`）；也可以写完整的 `https://` 地址 |
| `items[].until` | 可选，过期日期 `2026-12-31`，过期后这条不再显示 |

## 发一条重要公告

```json
{
  "id": "wechat-2026-11",
  "title": "关注公众号 W.技术站",
  "text": "关注微信公众号《W.技术站》。\n\n后续版本更新都在公众号通知，防止失联。\n\n没有直播源、点播源，可以进公众号获取。",
  "level": "danger",
  "popup": true,
  "image": "wechat-qr.png"
}
```

关键就两条：

1. **`popup: true`** —— 只有标了这个才会弹窗
2. **`id` 要换新的** —— 已经是 `wechat-2026-11`，下次发新公告就换成 `update-2.3` 之类，换了才会重新弹

`id` 不变的话，已经点过「我知道了」的用户永远看不到。这条规则同样适用于想「再推一次」的场合。

多条重要公告同时存在时，App 会合在一个弹窗里显示，点一次确认全部。配图取第一条带 `image` 的。

## 关于图片

- 图片**放在本仓库根目录**，`image` 只写文件名。App 会用「实际拉到公告的那个镜像」去取图，国内不会被 GitHub 裸域名卡住
- 也支持写成完整的图片地址（任意图床），只要能被 `https` 直接访问
- 建议：二维码类图片正方形、宽度 400px 以上，透明底或纯白底。App 弹窗里按 280dp 宽显示，外层会补白底和内边距，方便手机扫
- 图拉不到时弹窗只显示文字，不会出现裂图

当前仓库里的 `wechat-qr.png` 就是公众号二维码，重要公告直接 `"image": "wechat-qr.png"` 引用即可。

## App 从哪拉

四个镜像并发请求，按 `updated` 取最新的一份，所以某个镜像缓存旧了也不影响：

```
https://ghfast.top/https://raw.githubusercontent.com/wgsjjj/libretv-notice/main/notice.json
https://gh-proxy.com/https://raw.githubusercontent.com/wgsjjj/libretv-notice/main/notice.json
https://cdn.jsdelivr.net/gh/wgsjjj/libretv-notice@main/notice.json
https://fastly.jsdelivr.net/gh/wgsjjj/libretv-notice@main/notice.json
```

图片走同一个前缀，把 `notice.json` 换成文件名即可。

## 公告没弹？

1. 「设置 → 最底部」确认 App 版本 ≥ 2.2.7（更早的版本没有图片能力，2.2.5 才开始有弹窗）
2. 看顶部滚动栏有没有这条新公告 —— 没有就是还没拉到，等 30 分钟或重开 App
3. 打开「设置 → 调试 → 日志浮窗」，重启后看这几行：
   - `公告全部镜像拉取失败` → 网络问题
   - `公告已更新：N 条（重要 0）` → 拉到了，但内容是旧的（镜像缓存没刷新）
   - `重要公告已确认：xxx` → 你之前点过确认，账本里有这个 id，换 id 才会重弹
