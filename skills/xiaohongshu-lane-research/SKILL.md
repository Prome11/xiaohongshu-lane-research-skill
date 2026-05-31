---
name: xiaohongshu-lane-research
description: "Use when the user asks to research a Xiaohongshu lane/赛道, demand or content opportunity, market pattern, viral-note mechanism, representative keyword samples, ranking baseline, or comment-grounded public evidence. Requires the Codex Chrome Extension on the user's normal Chrome profile; stop if the Chrome gate cannot pass."
---

# Xiaohongshu Lane Research / 小红书赛道研究

Use this skill for Xiaohongshu lane/赛道, market, content-opportunity, or demand research from the user's own logged-in Chrome session. Keywords are search entry terms for collecting evidence; they are not the research goal.

Core principle: after the pre-collection confirmation gate passes, collect page evidence first, analyze later.

Live Xiaohongshu collection has exactly one supported execution surface: the Codex Chrome Extension connected to the user's own Chrome profile and account.

## Dependency Contract

This skill is portable, but the live browser environment is not portable with it.

The user running the skill must have:
- Codex Desktop with the Chrome plugin available
- the Codex Chrome Extension installed, enabled, and reachable from Codex
- a normal Chrome profile with the user's own Xiaohongshu session
- user attention available for login, captcha, safety checks, or account prompts
- `getnote` configured and authorized with the user's own account only when the user wants video-note body/caption extraction

The skill must not carry, copy, request, or assume another user's Chrome profile, cookies, local storage, account session, passwords, private messages, notifications, or Get笔记 account state.

If the Chrome Extension gate cannot prove a controllable Xiaohongshu tab in the user's own Chrome profile, stop live collection and record the exact stop reason. Do not switch to a separate browser, temporary profile, desktop-control layer, debug-port browser, or any other execution surface.

Get笔记 is an optional enhancement layer, not a live-collection gate. If Get笔记 is missing, unauthenticated, unauthorized, or declined by the user, continue collecting public page evidence when the Chrome gate passes. Record video body/copy extraction as `unavailable` or `not_requested`, and downgrade analysis so it does not claim video transcript, full script, or hidden video content.

## When To Use

Use this when the task asks for:
- Xiaohongshu lane/赛道 research, market/opportunity research, or demand/content analysis that needs public Xiaohongshu evidence
- keyword-driven retrieval of representative public notes, where the keyword is only the search entry term
- 爆款笔记, top posts, representative notes, or ranking samples
- note details, titles, authors, URLs, counts, body text, tags, and post type
- ordered comment records with first-level comments and opened second-level replies
- evidence that can later support content, demand, product, market, or lane analysis

Do not use this as a generic Xiaohongshu page-operation manual or as a workflow whose output is only term wording, term volume, or SEO-style term evaluation. One-off public page reading, copying, or UI operation without a lane/demand/market/opportunity research goal should follow the general Xiaohongshu operation guidance instead.

## 采集前确认项

Before live collection, make sure the user's request states every required field:

```text
目标：
赛道入口词/检索关键词：
每个入口词采集几篇笔记：（建议每个入口词 5 篇）
样本类型配比：（默认短视频笔记约 70%，图文笔记约 30%；例如 5 篇里优先 4 视频 + 1 图文，10 篇里 7 视频 + 3 图文）
内容提取范围：（短视频可选择用已授权的 Get笔记提取文案/caption/可用正文；没有 Get笔记时只记录标题、可见正文/caption、评论和链接；图文笔记只记录标题、可见正文/caption、评论，不做图片 OCR/视觉理解）
评论区采集范围：（建议每篇笔记前 30 条评论线，包含一级评论和已展开的二级回复；如需要更深可设 50 条）
排序/时间口径：
输出用途：
```

This is a hard gate. If any required field is missing or only implied, do not open, claim, navigate, search, filter, or inspect Xiaohongshu. Ask one short clarification that includes the missing fields and recommended defaults.

Do not silently apply defaults. Defaults are only suggestions to include in the clarification question. You may apply them only after the user explicitly accepts them, such as `按默认`, `你来定`, or an equivalent approval.

Examples that are not enough to start live collection:
- `帮我分析一下，这个关键词在小红书的爆款情况。`
- `搜一下这个词的小红书爆款。`
- `看看这个关键词怎么样。`

For these, ask for confirmation first because note count, comment range, sorting/time baseline, and output use are still missing.

Default suggestions:
- notes per search entry term: 5
- sample type mix: video-first; target about 70% video notes and 30% image/text notes unless the user specifies otherwise
- content extraction: video notes should use Get笔记 copy/body/caption extraction only when Get笔记 is available, authorized, and within user scope; without Get笔记, keep the video sample but mark video body/copy extraction as unavailable and analyze only visible page/comment evidence
- comment range per accepted note: 30 comment lines
- upper normal comment range: 50 comment lines
- sorting/time baseline when the user does not specify: `最多点赞 + 半年内`
- output split: raw evidence in `raw/*.md`, final HTML report in `analysis/*_analysis_report.html`, optional working notes in `_working/`

Comment lines mean page-order records. A top-level comment counts as one line; an opened second-level reply counts as one nested line under its top-level comment.

## Chrome Extension Gate

Before any live collection, load and follow `chrome:Chrome`. Use only the Chrome Extension backend.

Gate requirements:
- initialize the Chrome Extension backend with the Chrome skill's guarded setup cell
- call `browser.user.openTabs()` and choose the most recent tab whose visible title or URL is Xiaohongshu
- call `browser.user.claimTab(tabInfo)` with the exact tab object returned by `openTabs()`
- verify `await tab.url()` has host `xiaohongshu.com` or `www.xiaohongshu.com`
- verify `await tab.title()` or a fresh `tab.playwright.domSnapshot()` shows the expected Xiaohongshu page
- record `collection_tool: Codex Chrome Extension` in each run note or raw file before collecting evidence

If no Xiaohongshu tab exists:
- create a new tab through the same Chrome Extension backend only
- navigate that tab to `https://www.xiaohongshu.com`
- let the user handle login, captcha, account selection, or page attention
- record `chrome-plugin needs user attention` and stop until the user says the page is ready

Hard blocks:
- if the Chrome Extension backend cannot connect, stop and record `chrome-plugin unavailable`
- if the claimed or opened tab is not Xiaohongshu, stop and record `browser-target blocked`
- do not substitute any desktop-control layer, system UI inspection path, separate browser service, debug-port browser, temporary profile, managed browser, guest profile, or tool-launched profile
- do not read cookies, local storage, passwords, private messages, notifications, account backend pages, or non-public account data

## Collection Pattern

Use the cheapest reliable page signal:
- `tab.url()` and `tab.title()` for page identity and detail URLs
- one `tab.playwright.domSnapshot()` after navigation, filter changes, opening a note, expanding replies, or comment scrolling
- read-only `tab.playwright.evaluate()` only for rendered page text, links, counts, and public DOM structure
- Playwright locators only after a fresh enough snapshot proves the target and selector
- controlled clicks, fills, and scrolls only inside the claimed Chrome tab

Do not repeatedly dump whole-page text. Prefer one orientation snapshot, then scoped extraction from search results, detail content, or the comment container.

## Sample Mix And Optional Content Extraction

The old collection path often over-weighted title and comments because visible note body/caption was hard to extract. For video-led lane research, use Get笔记 when it is available and authorized by the user. When it is not available, keep the evidence boundary explicit instead of blocking the whole collection.

Default sample mix:
- Accepted samples should be video-led: about 70% video notes and 30% image/text notes.
- For a 5-note target, prefer 4 video notes + 1 image/text note.
- For a 10-note target, prefer 7 video notes + 3 image/text notes.
- If the ranked page cannot support that ratio without skipping clearly stronger public evidence, record the actual ratio and the reason under `scope notes`; do not pretend the default was met.

Extraction rule by post type:
- Video notes: Get笔记 extraction is preferred when available, authorized, and requested. Use it to extract the public copy/script/caption/body layer that supports content-mechanism analysis.
- If Get笔记 is unavailable, unauthenticated, unauthorized, declined, or fails after a recorded attempt, the video note can still be an accepted sample. Record title, visible body/caption if present, public counts, URL, and comments; mark `body/copy extraction status: unavailable`, `insufficient`, `failed`, or `not_requested`; do not make transcript/script/body-level claims for that sample.
- Image/text notes: do not do image OCR or visual interpretation by default. Record only the title, visible body/caption, public counts, and comments. Image URLs may be preserved as links, but they are not content evidence.
- If the user explicitly asks to analyze image contents in a future run, that is a separate OCR/vision layer and must be recorded separately from Get笔记 extraction.

Preferred Get笔记 route for video notes:
0. Confirm Get笔记 is available and authorized under the user's own account, or that the user explicitly wants this extraction attempt.
1. From the claimed normal Chrome Xiaohongshu tab, open the accepted note.
2. Use the Xiaohongshu share action to get a real share link when possible.
3. Run Get笔记 only on that link:

```bash
env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy -u ALL_PROXY -u all_proxy getnote save "<share_or_note_url>" --title "<short title>" --tag "XHS正文" -o json
```

4. Record the returned `note_id`, `note_type`, source URL, and whether the input was `share_link` or `address_url_fallback`.
5. Fetch and record both layers:

```bash
env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy -u ALL_PROXY -u all_proxy getnote note <note_id> --field content
env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy -u ALL_PROXY -u all_proxy getnote note <note_id> --field web_content
```

Evidence meaning:
- `web_content` is the visible web body/caption plus image links when available.
- `content` is Get笔记's extracted/AI-organized content layer based on what Get笔记 can access. Label it as `Get笔记 content`, not raw Xiaohongshu text.
- Image URLs in `web_content` prove that images exist, not what the images say. Image-internal text or meaning is unverified unless a separate OCR/vision step is performed and recorded.
- For video notes, `content` and `web_content` are the primary body/copy evidence layer when they are substantive.
- For image/text notes, Get笔记 is not required by default and must not become a substitute for image understanding.
- If `content` and `web_content` disagree, preserve both and put the conflict under `scope notes`.
- If Get笔记 returns only a title/link, or the content says it is not a real body extraction, mark `body extraction: insufficient` and do not use it as full copy evidence.

Validation note:
- Get笔记 may return useful web body/caption content and image links for some Xiaohongshu notes, but availability varies by note, account state, and network conditions.
- A successful Get笔记 extraction proves only that the tool returned accessible public note content for that run. It does not prove Get笔记 can read text inside images or recover unavailable private data.
- In this skill, use Get笔记 as the preferred extraction layer for video-note copy/body work when available and authorized; keep image/text notes lightweight unless the user separately asks for OCR/vision.

## Search Entry For Lane Evidence

For lane evidence collection, use keyword/search entry terms with the same stable note-search entry that prior successful runs used:

```text
https://www.xiaohongshu.com/search_result/?keyword=<encoded keyword>&source=web_search_result_notes&type=51
```

Run this navigation only inside the already claimed user Chrome tab. After navigation, verify:
- the URL contains `search_result`
- the search entry term is present
- `source=web_search_result_notes`
- `type=51`
- the page title or DOM shows a Xiaohongshu search result page

Do not treat `source=web_explore_feed` as equivalent for ranking/filter collection. If a search entered from the home or explore feed lands on `source=web_explore_feed`, switch the claimed tab to the stable note-search entry above before applying ranking filters.

For the default `最多点赞 + 半年内` gate:
- open the filter panel with the current visible filter icon selector, preferably `.filter-icon`, after a fresh page snapshot proves the search page
- confirm the expanded filter panel exposes `排序依据` and `发布时间`
- select `最多点赞` first, then immediately re-read the live page state before doing anything else
- after selecting `最多点赞`, verify whether the filter panel is still open; if it closed, reopen it and re-confirm `排序依据` / `发布时间` before selecting `半年内`
- select `半年内` only after the second fresh page-state check proves the correct option is visible and the click target is current
- verify active filter chips from the expanded panel, not just the collapsed `已筛选` label
- record the strict visible ranking order after the filter is applied

Do not chain coordinate clicks inside the filter panel. A click on `最多点赞` may close, shift, or re-render the panel; a second click using the old coordinate can hit a search result card and open a detail page. Treat every filter-option click as a state transition: click one option, wait, re-snapshot or re-evaluate, then choose the next target from the current DOM.

Do not reuse bare screen coordinates across pages, after navigation, after a filter option click, after returning from a detail page, or after any `已筛选` state change. If coordinate clicks are necessary, first read the target element's current bounding box from the live page and verify the panel is open before each individual filter-option click. If clicking the outer `div.filter` does not open the panel, click `.filter-icon` before falling back to measured coordinates.

If a filter attempt opens a note detail page before the ranking gate is complete, mark that page as `opened before sorting gate`, do not count it as an accepted sample, go back to the stable search URL, and restart the sorting gate from a fresh page snapshot.

If the page shows `安全验证`, captcha-like image selection, `请选择最符合描述的两张图片`, or another account-attention challenge, stop live collection, record `chrome-plugin needs user attention`, and wait for the user to complete it. Do not solve the challenge, route around it, switch browsers, switch profiles, use a debug-port browser, or continue by treating pre-challenge results as the final filtered order.

## Run Structure

Use one folder per run:

```text
runs/YYYY-MM-DD-topic/
  raw/
    entry_term_a.md
    entry_term_b.md
  analysis/
    topic_analysis_report.html
  _working/
```

`raw/*.md` holds evidence. `analysis/*_analysis_report.html` holds the final interpretation/report. `_working/` holds temporary extracts and notes produced during plugin-backed collection.

## Lane Evidence And Viral Note Mode

For each search entry term, record:
- lane/opportunity goal
- search entry term
- target note count
- target sample type mix and actual video/image split
- sorting/time baseline
- page ranking order from the current Chrome Extension tab
- insertion segments such as `相关搜索`
- title/card text
- author/counts when available
- first-pass route: accepted sample, scope note, or follow-up candidate

For each accepted note, record:
- sample number
- ranking position and segment
- title
- author
- full address-bar URL
- public counts
- post type: video or image
- public content summary from the page
- Get笔记 note_id: present only when extraction succeeds; `unavailable`, `not_requested`, or `failed` otherwise
- Get笔记 source type: `share_link`, `address_url_fallback`, `not_required`, `not_requested`, `failed`, or `unavailable`
- Get笔记 `web_content` summary: required only when video extraction succeeds; otherwise record why it is missing
- Get笔记 `content` summary: required only when video extraction succeeds; otherwise record why it is missing
- body/copy extraction status: complete, partial, insufficient, unavailable, failed, not_requested, or not_required
- image-internal content status: not_checked, image_links_only, OCR_verified, vision_verified, unavailable, or not_required
- tags
- comment range recorded
- lane signal
- demand signal
- scope notes

Use `scope note` for high-ranked cards that explain the lane boundary. Scope notes preserve ranking context without counting as accepted samples.

## Comment Mode

Use this mode for an opened note or a note selected during lane evidence collection.

Default target:
- record the first 30 comment lines per accepted note when available
- include first-level comments and opened second-level replies in page order
- preserve usernames when present
- if fewer than 30 lines are available, record all available lines and state the actual range
- if the user asks for deeper comment evidence, collect up to 50 lines by default before pausing

Collection order:
1. Confirm the note URL and comment area from `tab.url()`, `tab.title()`, and one fresh `tab.playwright.domSnapshot()`.
2. Locate the comment container from the snapshot.
3. Open only reply controls that affect the requested first-N range, such as `展开更多回复` or `展开 X 条回复`.
4. Scroll in short controlled increments inside the claimed tab when more comments are needed.
5. Re-snapshot after each expansion or scroll that changes the comment area.
6. Extract rendered comment text from the scoped comment container.
7. Clean repeated UI metadata.
8. Record comments in page order.

Do not expand all replies by default. Expand only what is needed to preserve the requested first-N order or a user-specified range.

## Context-Safe Limits

Checkpoint triggers:
- after the Chrome Extension gate is recorded
- after the sorting gate is recorded
- after every opened detail page, including rejected cards and `scope note` cards
- after every accepted representative note
- after each comment collection batch for a note
- before switching search entry terms
- before any planned pause, model switch, or new session

Session caps:
- stop after 2 opened detail pages total unless the user explicitly asked for a deeper live run
- stop after 1 accepted note when 30-50 comment lines were collected
- stop after 2 consecutive no-comment or scope-note pages; write the low-yield pattern and next candidate strategy
- do not open more than 2 detail pages after a checkpoint without updating durable state

When a cap is hit, finish the current raw entry, update project state files when working inside a project, and pause live collection with the next exact candidate or action.

## Raw File Shape

Use this shape for lane research runs:

```markdown
# 赛道研究证据

- 目标:
- 赛道入口词/检索关键词:
- 每个入口词采集几篇笔记:
- 样本类型配比:
- 内容提取范围:
- 评论区采集范围:
- 排序/时间口径:
- 输出用途:
- collection_tool: Codex Chrome Extension

## Chrome 插件闸门

## 排序闸门审查

## 检索筛查记录

## 样本类型配比

- 目标配比:
- 实际配比:
- 偏离原因:

## Note 1

### 视频文案/Get笔记提取 或 图文轻量记录

## Reviewer Pass
```

Use this shape for comments:

```markdown
## 评论区记录

- collection_tool: Codex Chrome Extension
- 页面显示评论数:
- 目标范围:
- 实际记录:

1. 用户名：评论正文
   - 回复 用户名：回复正文
2. 用户名：评论正文
```

For image-style or text-empty comments, write `[仅图片/无可见文字]`.

## Cleanup Rules

Preserve:
- usernames
- comment text
- mentions such as `@someone`
- emoji text when copied
- top-level and reply order

Clean repeated UI metadata:
- dates
- locations
- like counts
- `赞`
- `回复`
- empty image placeholders

When copied text combines several comments into one line, split by the repeated username, comment, metadata rhythm.

## Analysis And Report Layer

Only after raw evidence is complete enough for the user's target, write analysis separately from raw evidence. The default final deliverable is an HTML report under `analysis/*_analysis_report.html`.

Do not stop at a loose Markdown judgment unless the user explicitly asks for a brief/checkpoint only.

The report must answer the business question, not merely restate the collection:
- why posts became popular
- title and hook formulas
- video copy/body/caption patterns and lane mechanisms
- copy/comment fit: for video notes, explain how the copy structure created the comment demand and why that specific video became popular
- lightweight image/text-note support from title, visible body/caption, and comments
- comment-demand patterns
- emotional value
- practical value
- product/service/transaction signals when supported by evidence
- caveats and scope boundaries
- adjacent lane/search-entry expansion suggestions

Required report structure:
1. Header with lane name, research question, data boundary, and source limitations.
2. Badge/stat cards for entry terms, ranking excerpt size, accepted samples, video/image split, sampled comment lines, full-comment coverage, visible total comments, and blockers/gaps when relevant.
3. `先给结论`: 3-5 evidence-backed findings that state the lane mechanism in plain language.
4. `发布标题`: reusable title/hook formulas. Each formula needs a slot pattern, why it works, 1-2 template examples, and source examples with URLs/counts.
5. `短视频文案内容机制`: when Get笔记 extraction succeeds, use it to summarize copy/script/caption structure, proof style, and offer/service/product clues. For video samples without Get笔记 extraction, state that the video body/copy layer is unavailable and analyze only title, visible caption/body, public counts, and comments. Do not pretend to know the transcript or full script.
6. `图文笔记轻量观察`: use only title, visible body/caption, public counts, and comments from image/text notes. Do not analyze what the images say unless a separate OCR/vision layer exists. If the run has zero image/text accepted samples, state `本轮图文样本 0` in the data boundary or this section instead of silently omitting it.
7. `评论区共性`: comment behavior signals with counts or clear qualitative weight. Categories must fit the lane; do not reuse e-commerce/service categories blindly.
8. `入口词/样本拆解`: explain each search entry term or sample cluster as `标题入口 × 视频文案/图文正文 × 评论区验证 × 为什么火`, plus image-content status when images matter.
9. `代表样本拆解`: table with title, URL, post type, public counts, video-copy structure or image/text lightweight observation, comment uptake, why this note became popular, Get笔记 extraction status, image-internal content status, and comment validation.
10. `可借鉴选题/下一轮补证`: concrete content angles and the exact evidence gap to fill next.
11. Footer listing source files, Get笔记 note IDs when used, and the collection tool.

For single-entry runs, such as one lane entry term with 5 accepted notes, write the report at sample-cluster level instead of pretending there is cross-keyword coverage.

Comment signal rules:
- Count comment lines from the collected first-N records when possible.
- It is acceptable for comment categories to overlap; say so if percentages are not additive.
- Separate automated `问一问` summaries from organic user demand when they would otherwise dominate the signal.
- Use short representative comment snippets, but do not let one vivid comment override the aggregate pattern.

Evidence boundaries:
- Do not infer uncollected video visuals, full transcripts, private account data, or platform-wide market size.
- Do not turn scope notes, insertion blocks, or follow-up candidates into accepted samples.
- Do not let image/text notes dominate the evidence set by default; the normal lane-research mix is about 70% video notes and 30% image/text notes.
- Do not treat image/text notes as failed samples just because they lack Get笔记 extraction; they are supporting samples when title, visible body/caption, and comments are recorded.
- Do not describe routine video-copy extraction as `补采`, `补录`, or an exceptional add-on in future final reports. Video copy/body/caption is a normal evidence layer once this skill is in use.
- Do not treat Get笔记 `content` as raw Xiaohongshu copy; label it as Get笔记-extracted/AI-organized content.
- Do not claim to know what a Xiaohongshu image says just because Get笔记 returned an image URL. Image content requires OCR/vision evidence or must be listed as unknown.
- Do not ignore video body/copy evidence after it is available; title/comment-only analysis is incomplete for video-led final reports when Get笔记 extraction succeeded.
- Do not treat missing Get笔记 as a failed collection. Treat it as a missing video-content layer and clearly downgrade the analysis.
- Do not keep video copy and comments in separate silos when video copy exists. When it does not exist, explicitly say the copy/comment fit cannot be fully judged.
- If a conclusion depends on unopened follow-up candidates, put it under `下一轮补证`.
- Do not claim Douyin validation from Xiaohongshu evidence.

Reference report shape:
- Use data boundary first, then stats, top conclusions, title formulas, video-copy/body mechanisms, copy/comment fit for why each core video became popular, comment behavior signals, representative sample table, and next evidence gaps.

Never let analysis rewrite raw evidence. If a conclusion needs more proof, add it as a gap or next action.

## Reviewer Pass

Before marking a search entry term, note, or comment batch complete:
- confirm `collection_tool: Codex Chrome Extension` appears in the raw file
- confirm the user's lane/opportunity goal, search entry terms, note count, sample type mix, content extraction scope, comment range, sorting/time baseline, and output use are recorded
- recount accepted samples and confirm the count matches the target
- check the actual video/image split and explain any meaningful deviation from the 70/30 default
- check that each accepted sample has a full detail URL, post type, public counts, page content summary, extraction status, lane signal, demand signal, and comment range
- for video notes, confirm every accepted sample has either Get笔记 extraction fields or an explicit reason such as `unavailable`, `not_requested`, `failed`, or `insufficient`; do not require Get笔记 success for the sample to count
- for image/text notes, confirm the sample records title, visible body/caption, comments, and `Get笔记 source type: not_required` unless a separate extraction attempt was made
- verify comment counts and `实际记录` ranges against the written list
- flag duplicate-system, low-comment, AI-assisted, brand/product, service-market, generic psychology, parenting, male-growth, or entertainment caveats before analysis overweights them
- confirm the final HTML report exists when the user asked for analysis/report output
- check the final report includes data boundary, stats, video/image split, conclusions, title formulas, video copy mechanisms, copy/comment fit, lightweight image/text observations, image-content status where relevant, comment signals, sample table, evidence gaps, Get笔记 note IDs when used, and source files
- if image/text accepted sample count is zero, confirm the report says so explicitly rather than hiding the `图文笔记轻量观察` layer
- check project state and handoff files match the raw file and report before pausing

Reviewer status:
- `通过`: the evidence chain is internally consistent and meets the target
- `复核`: sample eligibility, ordering, comments, or state sync is uncertain
- `进行中`: the target still lacks required samples or comments
