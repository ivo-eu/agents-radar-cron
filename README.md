# agents-radar-cron

独立调度器仓库。唯一职责:每天定点远程触发 [`ivo-eu/agents-radar`](https://github.com/ivo-eu/agents-radar) 的日报。

## 为什么需要它

`ivo-eu/agents-radar` 是 fork。GitHub 对 fork 的 scheduled(cron)事件做低优先级派发,经常被静默丢弃 —— 实测 2026-06-15 之后该 fork 的日报定时 **0 次触发**,而上游非 fork 仓库同样的 cron 天天准点。

本仓库是**独立非 fork** 仓库,cron 派发可靠。它每天用 `workflow_dispatch`(显式事件,不受 fork cron 限流)去触发 fork 的日报。

完整诊断见 `agents-radar/定时爬取诊断与交接.md`。

## 配置

- Secret `DISPATCH_TOKEN`:有 `actions:write` 权限、能触发 `ivo-eu/agents-radar` 的 token。
- 定时:`.github/workflows/dispatch-daily-digest.yml`,`17 23 * * *` UTC = 次日 07:17 北京时间。
- 手动触发:Actions → Dispatch agents-radar daily digest → Run workflow。

## 验证是否生效

```bash
# 看本调度器最近运行
gh run list -R ivo-eu/agents-radar-cron -L 5
# 看它是否成功踢动了 fork 的日报
gh api "repos/ivo-eu/agents-radar/actions/workflows/weekly-digest.yml/runs?per_page=5" \
  --jq '.workflow_runs[] | "\(.created_at)\t\(.event)\t\(.status)/\(.conclusion)"'
```
