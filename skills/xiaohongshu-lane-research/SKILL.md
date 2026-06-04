---
name: xiaohongshu-lane-research
description: "Use when the user asks to research a Xiaohongshu lane/赛道, demand or content opportunity, market pattern, viral-note mechanism, representative keyword samples, ranking baseline, or comment-grounded public evidence. Requires the Codex Chrome Extension on the user's normal Chrome profile. After the user confirms the research target, continue until the confirmed target is complete unless the user stops, a hard browser/account block occurs, or the evidence source is exhausted."
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
- `getnote` configured and authorized with the user's own account when the confirmed scope includes video-note body/caption extraction

The skill must not carry, copy, request, or assume another user's Chrome profile, cookies, local storage, account session, passwords, private messages, notifications, or Get笔记 account state.

If the Chrome Extension gate cannot prove a controllable Xiaohongshu tab in the user's own Chrome profile, stop live collection and record the exact stop reason. Do not switch to a separate browser, temporary profile, desktop-control layer, debug-port browser, or any other execution surface.

Get笔记 is an optional dependency layer, not a live-collection gate. But the Get笔记 decision/status is mandatory for video-led lane research:
- If the confirmed content scope uses the default or asks for video body/caption extraction, run a bounded Get笔记 availability/extraction check for accepted video notes.
- If Get笔记 is missing, unauthenticated, unauthorized, declined by the user, or fails after a recorded attempt, continue collecting public page evidence when the Chrome gate passes.
- Record video body/copy extraction as `complete`, `partial`, `insufficient`, `unavailable`, `failed`, or `declined`.
- Use `not_requested` only when the user explicitly excludes Get笔记/video-copy extraction before collection, or when the sample is not a video note and Get笔记 is not required.
- Never mark an unchecked video-note extraction layer as `not_requested`; use `not_checked` only as a temporary in-progress state, and do not pass reviewer status with it.

Do not log into, authorize, repair, or configure Get笔记 on behalf of the user. If authorization or setup is missing, ask the user to handle it or continue without the extraction layer, then record the exact status as `unavailable`, `failed`, or `declined`.

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
内容提取范围：（默认：短视频用已授权的 Get笔记提取文案/caption/可用正文；如果 Get笔记不可用/未授权/失败/用户拒绝，则只记录标题、可见正文/caption、评论和链接并标明状态；图文笔记只记录标题、可见正文/caption、评论，不做图片 OCR/视觉理解）
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
- content extraction: video notes should use Get笔记 copy/body/caption extraction when Get笔记 is available, authorized, and within user scope; the agent must either attempt the extraction, record a concrete unavailable/failed status, or record the user's explicit decline. Without Get笔记, keep the video sample but mark video body/copy extraction as unavailable/failed/declined and analyze only visible page/comment evidence.
- comment range per accepted note: 30 comment lines
- upper normal comment range: 50 comment lines
- sorting/time baseline when the user does not specify: `最多点赞 + 半年内`
- output split: raw evidence in `raw/*.md`, final HTML report in `analysis/*_analysis_report.html`, optional working notes in `_working/`

Comment lines mean page-order records. A top-level comment counts as one line; an opened second-level reply counts as one nested line under its top-level comment.

## Completion Policy

After the user confirms the research target and collection scope, default to completing the full target. Do not stop after one or two notes, after one comment batch, after a checkpoint, or after a small partial sample just to ask whether to continue.

Continue working until one of these stop conditions is true:
- the confirmed target is complete: all search entry terms, accepted sample counts, comment ranges, Get笔记/video-copy extraction attempts or explicit unavailable/declined statuses, raw evidence files, requested analysis/report output, and reviewer pass are complete
- the user explicitly asks to stop, pause, reduce scope, or change direction
- a hard browser/account block occurs, such as Chrome Extension unavailable, wrong browser target, login required, captcha, safety verification, account attention, or inaccessible Xiaohongshu page
- the evidence source is exhausted under the confirmed search entry terms and filters, and the remaining gap is recorded with the exact missing count and next candidate strategy

Checkpoints are for durability, not permission to pause. After writing a checkpoint or raw evidence update, immediately continue with the next required candidate, comment batch, search entry term, extraction attempt, or report step unless a stop condition above is true.

If context length, model limits, or tool instability force a handoff, write the current evidence state and the exact next action, mark the run as `进行中`, and make clear that the target is not complete. Do not present a partial run as the final answer.

Get笔记 unavailability is not a stop condition. Continue without the video body/copy layer and mark the extraction status as `unavailable`, `failed`, or `declined`. Use `not_requested` only for an explicit pre-collection visible-page-only/no-Get笔记 scope.

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

## Human-Paced Visible UI Discipline

Use the same visible, slow, one-action-at-a-time path a user would naturally use. Do not collect from stale or unstable page state.

For live collection:
- Prefer claiming an already-open normal Chrome Xiaohongshu tab when one exists. If a new tab must be opened, load Xiaohongshu, wait for the visible home/search page to settle, and verify URL/title/DOM before any search or filter action.
- Enter every search entry term through the visible Xiaohongshu search box. Click the current search input, clear existing text if needed, type the keyword, and submit with the visible search control or Enter. Do not navigate directly to constructed `search_result` URLs for live collection.
- After the visible search submits, wait for result cards and the filter control, then take a fresh snapshot/screenshot before interacting.
- For video-led or video-only research, select the visible `视频` tab before the ranking filters unless the user explicitly asks for all-content ranking. Verify the active tab or video-style result cards before opening the filter panel. If you keep `全部` as the ranking surface, record that as a deliberate scope choice.
- Use visible UI events for clicks. It is acceptable to use DOM/locator reads to identify current targets and bounding boxes after a fresh snapshot, but do not mutate DOM state, dispatch synthetic events, call hidden APIs, or rely on stale selectors. When possible, click the current visible target center from its live bounds.
- One UI transition means one action: navigate, wait, re-read; click tab, wait, re-read; open filter, wait, re-read; click one filter option, wait, re-read. Do not chain search, tab, sort, time, and candidate extraction in one uninterrupted script.
- Use modest human-paced waits after visible transitions, especially after search navigation, tab switches, filter-option clicks, returning from detail pages, and comment scrolling. A fixed wait plus a fresh page-state check is better than racing the UI.
- Do not run multi-keyword or multi-note batch loops until a single keyword's ranking gate has passed and the current sorted DOM order has been recorded.
- After the ranking gate passes, collect one accepted note at a time from the current page order: open the note, record address/title/counts/content boundary, run the Get笔记/share-link layer when required by the confirmed content scope or record its concrete unavailable/failed/declined status, collect comments, return to the ranking page, then re-confirm page state before opening the next note.
- If any step opens the wrong page, returns empty results unexpectedly, changes the active tab, or shows `已筛选` without expanded-chip proof, stop the current candidate path and restore the ranking gate before counting any sample.

## Sample Mix And Get笔记 Boundary

The old collection path often over-weighted title and comments because visible note body/caption was hard to extract. For video-led lane research, use Get笔记 when it is available and authorized by the user. When it is not available, keep the evidence boundary explicit instead of blocking the whole collection. The agent must not skip the Get笔记 check silently and later call the extraction `not_requested`.

Default sample mix:
- Accepted samples should be video-led: about 70% video notes and 30% image/text notes.
- For a 5-note target, prefer 4 video notes + 1 image/text note.
- For a 10-note target, prefer 7 video notes + 3 image/text notes.
- If the ranked page cannot support that ratio without skipping clearly stronger public evidence, record the actual ratio and the reason under `scope notes`; do not pretend the default was met.

Extraction rule by post type:
- Video notes: Get笔记 extraction is the default body/caption route when available, authorized, and within the confirmed scope. Use it to extract the public copy/script/caption/body layer that supports content-mechanism analysis.
- If Get笔记 is unavailable, unauthenticated, unauthorized, declined, or fails after a recorded attempt, the video note can still be an accepted sample. Record title, visible body/caption if present, public counts, URL, and comments; mark `body/copy extraction status: unavailable`, `insufficient`, `failed`, or `declined`; do not make transcript/script/body-level claims for that sample.
- `not_requested` is valid only if the user explicitly set the content scope to visible-page-only/no-Get笔记 before collection. It is not valid merely because the agent skipped the check or wanted to move faster.
- Image/text notes: do not do image OCR or visual interpretation by default. Record only the title, visible body/caption, public counts, and comments. Image URLs may be preserved as links, but they are not content evidence.
- If the user explicitly asks to analyze image contents in a future run, that is a separate OCR/vision layer and must be recorded separately from Get笔记 extraction.

Preferred Get笔记 route for video notes:
0. Confirm Get笔记 is available and authorized under the user's own account for the confirmed scope, or record a concrete `unavailable`, `failed`, or `declined` status before treating the video sample as complete. Do not use unchecked `not_requested` for video samples.
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
- For Xiaohongshu video notes, substantive `web_content` is the primary Get笔记 video body/copy evidence layer. It may include the public caption/body and the video oral transcript/ASR text that Get笔记 extracted from the video.
- `content` is Get笔记's AI-organized summary/structured extraction based on what Get笔记 can access. Label it as `Get笔记 content`, not raw Xiaohongshu text and not the primary transcript.
- Image URLs in `web_content` prove that images exist, not what the images say. Image-internal text or meaning is unverified unless a separate OCR/vision step is performed and recorded.
- For video notes, mark `body/copy extraction status: complete` when `web_content` contains substantive oral transcript/body text. Use `partial` only when the returned text is clearly truncated, mostly tags, or otherwise incomplete but still useful. Use `insufficient` when Get笔记 returns only a title/link/tags without usable body or transcript text.
- For image/text notes, Get笔记 is not required by default and must not become a substitute for image understanding.
- If `content` and `web_content` disagree, preserve both and put the conflict under `scope notes`.
- If Get笔记 returns only a title/link, or the content says it is not a real body extraction, mark `body extraction: insufficient` and do not use it as full copy evidence.

Validation note:
- Get笔记 may return useful web body/caption content and image links for some Xiaohongshu notes, but availability varies by note, account state, and network conditions.
- A successful Get笔记 extraction proves only that the tool returned accessible public note content for that run. It does not prove Get笔记 can read text inside images or recover unavailable private data.
- In this skill, use Get笔记 as the preferred extraction layer for video-note copy/body work when available and authorized; keep image/text notes lightweight unless the user separately asks for OCR/vision.

## Visible Search Entry For Lane Evidence

For lane evidence collection, search entry terms must be entered through Xiaohongshu's visible search UI in the already claimed user Chrome tab.

For each entry term:
- Start from a verified Xiaohongshu home, explore, or search page in the user's normal Chrome tab.
- Use the visible search box. Click it, clear any old term, type the current entry term, and submit with the visible search control or Enter.
- Wait for the visible search results page to settle.
- Verify the search box, page title, or DOM shows the requested entry term.
- Verify visible result cards and the filter control exist before applying ranking filters.
- Record the resulting address-bar URL as evidence only after the visible search has completed.

Do not construct or navigate directly to `search_result` URLs for live collection. Do not replace the visible search route with hidden APIs, backend endpoints, browser history manipulation, address-bar URL templates, or prebuilt `source=web_search_result_notes&type=51` links.

Before full collection, run a single-keyword filter probe:
- Use the first confirmed search entry term.
- Establish the intended ranking surface (`视频` for video-led/video-only research unless explicitly scoped otherwise; `全部` only when all-content ranking is intended).
- Apply the ranking/time filters using the human-paced visible UI discipline above.
- Verify active chips from the expanded filter panel and record the first visible sorted page order.
- Only after this probe passes may you continue to collect the target number of notes or repeat the route for additional entry terms.
- If the probe triggers safety verification, empty results, or an unstable page, record the exact failing action and do not proceed to batch collection. After the user clears verification, restart the probe from a fresh search-page state; do not reuse pre-verification candidates.

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

If a filter attempt opens a note detail page before the ranking gate is complete, mark that page as `opened before sorting gate`, do not count it as an accepted sample, go back to the search results page through normal browser back or by re-running the visible search-box route, and restart the sorting gate from a fresh page snapshot.

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
- Get笔记 note_id: present only when extraction succeeds; `unavailable`, `failed`, `declined`, or explicit-scope `not_requested` otherwise
- Get笔记 source type: `share_link`, `address_url_fallback`, `not_required`, `failed`, `unavailable`, `declined`, or explicit-scope `not_requested`
- Get笔记 `web_content` summary: required only when video extraction succeeds; otherwise record why it is missing
- Get笔记 `content` summary: required only when video extraction succeeds; otherwise record why it is missing
- body/copy extraction status: complete, partial, insufficient, unavailable, failed, declined, explicit-scope not_requested, or not_required
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
- if the user asks for deeper comment evidence, collect up to 50 lines by default per accepted note before moving to the next required sample

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

## Checkpoint Policy

Checkpoint triggers:
- after the Chrome Extension gate is recorded
- after the sorting gate is recorded
- after every opened detail page, including rejected cards and `scope note` cards
- after every accepted representative note
- after each comment collection batch for a note
- before switching search entry terms
- before any unavoidable tool/model handoff

Checkpoint requirements:
- update the raw evidence file or working state with what has been collected so far
- record accepted sample count, rejected/scope-note candidates, actual comment ranges, extraction status, and remaining target gap
- after the checkpoint, continue toward the confirmed target unless a stop condition in `Completion Policy` is true

Low-yield handling:
- after 2 consecutive no-comment, irrelevant, duplicate, or scope-note pages, write the low-yield pattern and switch to the next candidate strategy
- do not stop solely because of low yield unless the confirmed evidence source is exhausted

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
- video copy/body/caption patterns and lane mechanisms when extraction exists; otherwise state the missing video-content layer
- copy/comment fit: for video notes with extracted copy, explain how the copy structure created the comment demand and why that specific video became popular; without extracted copy, limit the judgment to title, visible caption/body, public counts, and comments
- lightweight image/text-note support from title, visible body/caption, and comments
- comment-demand patterns
- emotional value
- practical value
- product/service/transaction signals when supported by evidence
- caveats and scope boundaries
- adjacent lane/search-entry expansion suggestions

Fixed report standard:
The final HTML report must follow the fixed lane-research report framework. Do not substitute a generic report order, rename the main sections, or scatter the required lens across unrelated headings.

Required title and section order:
1. H1: `视频文案、评论反馈与爆款机制反推`.
2. Header/lead block with lane name, research question, data boundary, source limitations, collection tool, and reviewer status.
3. Badge/stat cards for entry terms, ranking excerpt size, accepted samples, video/image split, sampled comment lines, full-comment coverage, visible total comments, and blockers/gaps when relevant.
4. H2 `先给结论`: 3-5 evidence-backed findings that state the lane mechanism in plain language.
5. H2 `一、发布标题：可直接套用的爆款公式`: reusable title/hook formulas. Each formula needs a slot pattern, why it works, 1-2 template examples, and source examples with URLs/counts.
6. H2 `二、短视频文案内容机制：文案怎么讲，评论怎么接`: when Get笔记 extraction succeeds, use it to summarize copy/script/caption structure, proof style, and offer/service/product clues, then explain how comments validated or rejected the copy. For video samples without Get笔记 extraction, keep this same section name and state that the video body/copy layer is unavailable, then analyze only title, visible caption/body, public counts, and comments. Do not pretend to know the transcript or full script.
7. H2 `三、图文笔记轻量观察：本轮图文样本 N`: use only title, visible body/caption, public counts, and comments from image/text notes. Do not analyze what images say unless a separate OCR/vision layer exists. If the run has zero image/text accepted samples, write `本轮图文样本 0` instead of omitting the section.
8. H2 `四、评论区共性：用户到底在用评论做什么`: comment behavior signals with counts or clear qualitative weight. Categories must fit the lane; do not reuse e-commerce/service categories blindly.
9. H2 `五、入口词看爆款为什么爆：标题 × 视频文案 × 评论区验证`: explain each search entry term or sample cluster as `标题入口 × 视频文案/图文正文 × 评论区验证 × 为什么火`, plus image-content status when images matter.
10. H2 `六、代表样本拆解：文案 × 评论 × 爆火原因`: table with title, URL, post type, public counts, video-copy structure or image/text lightweight observation, comment uptake, why this note became popular, Get笔记 extraction status, image-internal content status, and comment validation.
11. H2 `七、可借鉴选题与下一轮补证`: concrete content angles and the exact evidence gap to fill next.
12. Footer listing source files, Get笔记 note IDs when used, and the collection tool.

The report's core analytical lens is always `文案怎么讲 × 评论怎么接 × 为什么火`. If video-copy evidence is missing, preserve that lens but downgrade the body-copy part explicitly instead of changing the report structure.

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
- Use the fixed report framework above: `视频文案、评论反馈与爆款机制反推`, then `先给结论`, `一、发布标题`, `二、短视频文案内容机制：文案怎么讲，评论怎么接`, `三、图文笔记轻量观察`, `四、评论区共性`, `五、入口词看爆款为什么爆`, `六、代表样本拆解`, and `七、可借鉴选题与下一轮补证`.
- A report that merely includes similar information under different generic headings does not pass reviewer status.

Never let analysis rewrite raw evidence. If a conclusion needs more proof, add it as a gap or next action.

## Reviewer Pass

Before marking a search entry term, note, or comment batch complete:
- confirm `collection_tool: Codex Chrome Extension` appears in the raw file
- confirm the user's lane/opportunity goal, search entry terms, note count, sample type mix, content extraction scope, comment range, sorting/time baseline, and output use are recorded
- recount accepted samples and confirm the count matches the target
- check the actual video/image split and explain any meaningful deviation from the 70/30 default
- check that each accepted sample has a full detail URL, post type, public counts, page content summary, extraction status, lane signal, demand signal, and comment range
- for video notes, confirm every accepted sample has either Get笔记 extraction fields, a recorded extraction attempt failure, a concrete unavailable/unauthorized status, or the user's explicit pre-collection decline. `not_requested` is valid only for explicit visible-page-only/no-Get笔记 scope; `not_checked` or skipped extraction cannot pass reviewer status.
- for image/text notes, confirm the sample records title, visible body/caption, comments, and `Get笔记 source type: not_required` unless a separate extraction attempt was made
- verify comment counts and `实际记录` ranges against the written list
- flag duplicate-system, low-comment, AI-assisted, brand/product, service-market, generic psychology, parenting, male-growth, or entertainment caveats before analysis overweights them
- confirm the final HTML report exists when the user asked for analysis/report output
- check the final report follows the fixed lane-research report framework and includes data boundary, stats, video/image split, conclusions, title formulas, available video copy mechanisms, copy/comment fit when supported, lightweight image/text observations, image-content status where relevant, comment signals, sample table, evidence gaps, Get笔记 note IDs when used, and source files
- if image/text accepted sample count is zero, confirm the report says so explicitly rather than hiding the `图文笔记轻量观察` layer
- check project state and handoff files match the raw file and report before final delivery or unavoidable handoff

Reviewer status:
- `通过`: the evidence chain is internally consistent and meets the target
- `复核`: sample eligibility, ordering, comments, or state sync is uncertain
- `进行中`: the target still lacks required samples or comments
