# Xiaohongshu Lane Research Skill

A Codex Skill for public Xiaohongshu lane research: keyword entry terms, ranked public-note samples, comment evidence, optional Get笔记 extraction, and evidence-bounded analysis reports.

This repository contains only the reusable Skill. It does not include browser sessions, credentials, private data, historical research runs, or collected Xiaohongshu samples.

## What This Is

`xiaohongshu-lane-research` helps Codex collect and analyze public Xiaohongshu evidence for:

- lane / 赛道 research
- keyword-based representative samples
- viral-note and content-mechanism analysis
- public comment demand signals
- evidence-bounded content or market opportunity research

It is not a generic Xiaohongshu scraper, a private-data tool, or a Douyin automation workflow.

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

Clone or download this repository, then copy the Skill folder into your Codex skills directory:

```bash
git clone https://github.com/Prome11/xiaohongshu-lane-research-skill.git
cd xiaohongshu-lane-research-skill
mkdir -p ~/.codex/skills
cp -R skills/xiaohongshu-lane-research ~/.codex/skills/
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
