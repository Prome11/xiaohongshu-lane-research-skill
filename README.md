# 小红书账号内容打法研究 Skill for Codex

如果一个 AI 能帮你系统研究一个赛道里的数百篇高赞笔记、优秀创作者、短视频内容和评论区反馈，你还会不知道这个赛道该怎么做爆款内容吗？

这个 Skill 基于 Codex，按照 IP 操盘手的研究方法，系统拆解一个小红书赛道里的优秀创作者和爆款笔记：他们怎么选题、标题怎么写、短视频怎么讲、评论区为什么会有反馈、用户真实需求藏在哪里。

已经被平台流量和用户反馈验证过的内容，才值得优先研究。和网上大多数“洗稿工具”不同，它不是帮你复制别人的内容，而是帮你完成一次面向账号起号和内容创作的赛道内容打法研究。

你得到的不只是零散素材，而是一份可复查、有证据边界的内容打法报告。它会帮你看清：

- 这个赛道的爆款内容通常在讲什么
- 优秀创作者反复使用哪些选题角度
- 高赞笔记的标题钩子和表达套路
- 短视频内容如何开头、铺垫、制造共鸣或信任
- 评论区里用户反复暴露的需求、疑虑和痛点
- 哪些内容结构值得你借鉴，哪些只是个别偶然爆款

这里的“全量研究”不是获取小红书后台全库数据，而是在你指定的入口词、排序口径和公开可访问页面范围内，尽可能系统覆盖这个赛道的高赞/爆款内容、创作者表达和评论区需求。样本量可以按你的研究深度逐步扩大；样本越多，运行时间越长，也越需要你处理登录、验证或 Get笔记授权等用户侧动作。

This repository contains only the reusable Codex Skill. It does not include browser sessions, credentials, private data, historical research runs, or collected Xiaohongshu samples.

## What It Helps You Research

`xiaohongshu-lane-research` helps Codex study what content works in a Xiaohongshu niche:

- content angles and topic selection used by strong creators
- title hooks, promise structures, and emotional triggers
- short-video openings, pacing, proof styles, and trust-building patterns
- visible captions/body text and image/text-note support
- public comment demand signals, objections, questions, and hidden needs
- which conclusions are supported by public samples, and which need more evidence

It is not a generic Xiaohongshu scraper or a private-data tool.
It is not a Douyin automation workflow.

## Requirements

This Skill is meant for Codex Desktop. Live collection requires the user running it to have:

- Codex Desktop
- the Chrome plugin available in Codex
- the Codex Chrome Extension installed and enabled in the user's own Chrome profile
- the user's own logged-in Xiaohongshu session
- user attention available for login, captcha, safety checks, or account prompts
- `getnote` configured and authorized with the user's own account when video-note body/caption extraction is desired

The Skill cannot package or transfer anyone's browser login state. Each user must use their own Chrome profile and their own Xiaohongshu account.

Get笔记 is optional. Without Get笔记 authorization, the Skill can still collect ranked public notes, URLs, titles, visible captions/body text, counts, and public comments. The difference is analytical: Codex must mark video body/copy extraction as unavailable and avoid claims about full video scripts, transcripts, or hidden video content.

## Installation

In Codex, send this message:

```text
Use $skill-installer to install:
https://github.com/Prome11/xiaohongshu-lane-research-skill/tree/main/skills/xiaohongshu-lane-research
```

Restart Codex so the new Skill is discovered.

## Usage

After installation, ask Codex to use the Skill, for example:

```text
Use $xiaohongshu-lane-research to research this Xiaohongshu lane.
目标：
赛道入口词/检索关键词：
每个入口词采集几篇笔记：
样本类型配比：
内容提取范围：
评论区采集范围：
排序/时间口径：
输出用途：
```

The Skill requires the collection setup before opening Xiaohongshu. If required fields are missing, Codex should ask for confirmation first.

Once the research target is confirmed, Codex should keep working until the target is complete. It should not stop after one or two notes just to ask whether to continue. It should only stop when the agreed target is complete, the user explicitly stops or changes scope, a hard browser/account block appears, or the available evidence source is exhausted and the remaining gap is documented.

For live Xiaohongshu collection, Codex should use the visible Xiaohongshu search box for every entry term. It should not navigate directly to constructed `search_result` URLs. After visible search, Codex should run one search-entry filter probe first, change only one visible UI state at a time, confirm active filters from the expanded panel, record the current sorted page order, then collect one note at a time. Do not batch-click filters or scrape candidates before the visible filter state is proven.

## Safety Boundary

The Skill should only collect public visible information from Xiaohongshu pages in the user's own logged-in Chrome session.

It must stop instead of switching tools when:

- the Codex Chrome Extension is unavailable
- the claimed tab is not Xiaohongshu
- login, captcha, or account attention is required
- the page cannot be verified by URL, title, or DOM

The Skill must not inspect cookies, local storage, passwords, private messages, notifications, backend account data, or non-public data.

The Skill must not use someone else's Get笔记 account or saved note state. If Get笔记 is used, it should be authorized by the user running the workflow. Codex should not log into or authorize Get笔记 for the user; if authorization is missing, record the extraction layer as unavailable or not requested.

## Evidence Boundary

Xiaohongshu evidence is public sample evidence. It can show what public notes and comments say, but it does not by itself prove:

- platform-wide market size
- actual transaction volume
- income claims
- private account analytics
- Douyin automation readiness

Keep facts, assumptions, gaps, and next actions separate.

## License

MIT
