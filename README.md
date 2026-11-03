# libretv-notice

LibreTV TV 客户端的远程公告。App 启动时拉取本仓库的 `notice.json`，拉到就显示在首页顶部的滚动公告栏。

## 怎么改

1. 打开 `notice.json`，点右上角铅笔图标
2. 改 `items` 里任意一条的 `text`，保存（Commit changes）
3. App 最多 30 分钟后自动更新，重开 App 会立即拉取

## 字段

| 字段 | 说明 |
| --- | --- |
| `enabled` | `false` 时 App 不显示任何公告 |
| `items[].text` | 公告正文，纯文本，最多 200 字 |
| `items[].level` | `info` 蓝 / `warn` 黄 / `danger` 红 |
| `items[].until` | 可选，过期日期 `2026-12-31`，过期后这条不再显示 |

`notice.json` 格式写错时 App 会直接忽略本次更新，继续用上一条缓存，不会白屏也不会报错。
