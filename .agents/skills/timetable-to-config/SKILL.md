---
name: timetable-to-config
description: >-
  Expert guide and workflow for converting music festival, idol live, and tokutenkai timetable images or raw text into LivePulse JSON configuration files. Enforces schema standards (v2.0 / flat list), parallel stage/table mapping, mandatory `isStarred: false` default, and local hierarchical file saving (`events/YYYY/MM/DD/<event_name>.json`).
---

# Timetable to LivePulse Config Generator Skill

This skill provides step-by-step instructions for analyzing timetable images, social media announcements (X/Twitter, Instagram, Weibo), posters, or raw timetable text and generating clean, valid **LivePulse JSON configuration files**, automatically saved to local directories organized by **Year / Month / Day (`events/YYYY/MM/DD/`)**.

---

## 📁 Core Rule 1: Local File Output Hierarchically by Date (年月日分级存储)

> [!IMPORTANT]
> **All generated JSON configuration files MUST be automatically written to local files organized by Year / Month / Day hierarchy:**
> `events/YYYY/MM/DD/<YYYY-MM-DD>_<eventName>.json`
>
> - **Folder Structure**: `events/<YYYY>/<MM>/<DD>/` (e.g. `events/2026/09/05/`)
> - **Filename**: `<YYYY-MM-DD>_<eventName>.json` (e.g. `2026-09-05_ワンコインショーケース.json`)
> - When generating configs, always invoke `write_to_file` to persist the file locally and return the clickable file link in markdown.

---

## ⚠️ Core Rule 2: Default Unselected (`isStarred: false`)

> [!IMPORTANT]
> **All generated event items (`lives`, `tokutenkais`, `otherEvents`) MUST have `"isStarred": false` by default.**
> When a user imports a festival or event timetable, no events should be pre-selected. This allows the user to browse the timetable and manually pick the specific performances and meet & greets they intend to attend.

---

## 📋 Supported JSON Configuration Schemas

LivePulse supports two standard formats. Always produce valid JSON using either of these schemas:

### Format 1: Structured Festival Template (`v2.0` - Recommended)

Use this format when converting a single music festival, idol joint showcase, or multi-stage event.

```json
{
  "version": "2.0",
  "festival": {
    "name": "ワンコインショーケース",
    "venue": "Spotify O-WEST",
    "date": "2026-09-05",
    "openTime": "12:00",
    "startTime": "12:30",
    "endTime": "15:25",
    "description": "ワンコインショーケース @ Spotify O-WEST (開場12:00 / 開演12:30)"
  },
  "lives": [
    {
      "id": "onecoin_live_1",
      "groupName": "Mirror,Mirror",
      "stage": "Spotify O-WEST 主舞台",
      "startTime": "12:30",
      "endTime": "12:55",
      "description": "Live 舞台演出 (25分钟)",
      "isStarred": false
    },
    {
      "id": "onecoin_live_2",
      "groupName": "AKANECLUB.",
      "stage": "Spotify O-WEST 主舞台",
      "startTime": "12:55",
      "endTime": "13:20",
      "description": "Live 舞台演出 (25分钟)",
      "isStarred": false
    }
  ],
  "tokutenkais": [
    {
      "id": "onecoin_tokuten_1",
      "groupName": "Mirror,Mirror",
      "venue": "Spotify O-WEST",
      "tableArea": "1号卓",
      "startTime": "13:55",
      "endTime": "15:25",
      "description": "終演後物販・特典会",
      "isStarred": false
    },
    {
      "id": "onecoin_tokuten_2",
      "groupName": "AKANECLUB.",
      "venue": "Spotify O-WEST",
      "tableArea": "2号卓",
      "startTime": "13:55",
      "endTime": "15:25",
      "description": "終演後物販・特典会",
      "isStarred": false
    }
  ],
  "otherEvents": []
}
```

### Format 2: Flat Schedules Array (`livepulse_schedules_v4`)

Use this format for full backups or multi-day cross-event schedule lists.

```json
[
  {
    "id": "evt_1725000001",
    "groupName": "Starry☆Sky",
    "title": "Starry☆Sky",
    "type": "live",
    "category": "live",
    "parentEvent": "SUMMER IDOL FES 2026",
    "venue": "瓦肆 VAS 主舞台",
    "tableArea": "",
    "date": "2026-08-29",
    "startTime": "13:30",
    "endTime": "14:00",
    "description": "Live Stage Performance",
    "isStarred": false
  },
  {
    "id": "evt_1725000002",
    "groupName": "Starry☆Sky",
    "title": "Starry☆Sky",
    "type": "tokuten",
    "category": "tokuten",
    "parentEvent": "SUMMER IDOL FES 2026",
    "venue": "瓦肆 VAS 特典区",
    "tableArea": "3号桌",
    "date": "2026-08-29",
    "startTime": "14:20",
    "endTime": "15:30",
    "description": "特典会 (拍立得/交流)",
    "isStarred": false
  }
]
```

---

## 🔍 Extraction & Processing Workflow

When given an image or raw text, follow this 5-step workflow:

### Step 1: Extract Event Metadata
- **Date**: Extract and format strictly as `YYYY-MM-DD` (e.g. `2026-09-05`). If only `9.5` or `9/5` is given, infer the upcoming year.
- **Festival / Event Name**: Extract official title (e.g. `ワンコインショーケース`, `TOKYO IDOL FESTIVAL 2026`).
- **Venue(s)**: Extract main venue or club names (e.g. `Spotify O-WEST`, `MAO Livehouse`, `瓦肆 VAS`).
- **Doors / Start Time**: Extract `OPEN` and `START` times if available.

### Step 2: Classify Event Item Types
Categorize each timeline entry into one of 3 types:

1. **`live` (Stage Performance)**:
   - Music sets, idol live performances, stage shows.
   - Extract `groupName` (strip trailing parentheticals like `(Live)` or `(演出)`).
   - Extract stage name into `stage` / `venue` (e.g. `Main Stage`, `WEST`, `EAST`).

2. **`tokuten` (Meet & Greet / 特典会 / 物贩)**:
   - Keywords: `特典会`, `物販`, `チェキ会`, `サイン会`, `Fan Sign`, `Meet & Greet`, `Parallel Tokuten`, `並行特典会`, `終演後物販`.
   - Extract `tableArea` / `booth` (e.g. `3号卓`, `Booth B`, `ロビーA`, `5号桌`).
   - **Joint/Post-Live Tokutenkai Handling**: If a single time slot is labelled `終演後物販・特典会` (all artists participating after the live), expand it into individual `tokuten` items for each performing artist sharing that time slot.

3. **`other` (Special Announcements / Intermissions / Special Sessions)**:
   - ⚠️ **Important**: `OPEN / START`（開場 / 開演）属于活动的基础元数据（录入在 `festival.openTime` 与 `festival.startTime`），**不作为独立的 `otherEvents` 条目生成**。
   - 适用于 `otherEvents` 的类型：`休憩 / Break`（中场休息）、`転換 / 换场`（特别转场/舞台准备）、`全体集合 / エンディング`（全员大合影/谢幕仪式）、`DJ Time / 前置MC`（非演出团体DJ/开场白）、`抽選会`（抽奖活动）等。
   - 若时间表中无特殊中场或仪式环节，`otherEvents` 数组应保持为空 `[]`。

### Step 3: Time Normalization
- All times must be strictly formatted in 24-hour `HH:mm` format with leading zeros:
  - `9:30` ➔ `09:30`
  - `12:30〜12:55` ➔ `startTime: "12:30"`, `endTime: "12:55"`
  - `1:30 PM` ➔ `13:30`

### Step 4: Ensure Unselected Default
- Check every element in `lives`, `tokutenkais`, and `otherEvents` to verify `"isStarred": false`.

### Step 5: Save to Local Directory (按年月日分级)
- Automatically persist the generated JSON to the hierarchical path:
  `events/<YYYY>/<MM>/<DD>/<YYYY-MM-DD>_<sanitized_event_name>.json`
- Example: `events/2026/09/05/2026-09-05_ワンコインショーケース.json`
- In chat output, provide the clickable link `[YYYY-MM-DD_eventName.json](file:///absolute/path/to/file)` along with the full JSON code block.

---

## 💡 Practical Examples

### Example A: Japanese Idol Showcase Text

**Input Text**:
```text
ワンコインショーケース
○日程 9.5（土）
○会場 Spotify O-WEST
○时间 开场12:00/开演12:30
○タイムテーブル
OPEN12:00/START12:30
12:30〜12:55 Mirror,Mirror
12:55〜13:20 AKANECLUB.
13:20〜13:45 かすみ草とステラ
13:55〜15:25 終演後物販・特典会
```

**Output JSON**:
```json
{
  "version": "2.0",
  "festival": {
    "name": "ワンコインショーケース",
    "venue": "Spotify O-WEST",
    "date": "2026-09-05",
    "openTime": "12:00",
    "startTime": "12:30",
    "endTime": "15:25",
    "description": "ワンコインショーケース @ Spotify O-WEST"
  },
  "lives": [
    {
      "id": "live_1",
      "groupName": "Mirror,Mirror",
      "stage": "Spotify O-WEST",
      "startTime": "12:30",
      "endTime": "12:55",
      "description": "Live 舞台演出",
      "isStarred": false
    },
    {
      "id": "live_2",
      "groupName": "AKANECLUB.",
      "stage": "Spotify O-WEST",
      "startTime": "12:55",
      "endTime": "13:20",
      "description": "Live 舞台演出",
      "isStarred": false
    },
    {
      "id": "live_3",
      "groupName": "かすみ草とステラ",
      "stage": "Spotify O-WEST",
      "startTime": "13:20",
      "endTime": "13:45",
      "description": "Live 舞台演出",
      "isStarred": false
    }
  ],
  "tokutenkais": [
    {
      "id": "tokuten_1",
      "groupName": "Mirror,Mirror",
      "venue": "Spotify O-WEST",
      "tableArea": "",
      "startTime": "13:55",
      "endTime": "15:25",
      "description": "終演後物販・特典会",
      "isStarred": false
    },
    {
      "id": "tokuten_2",
      "groupName": "AKANECLUB.",
      "venue": "Spotify O-WEST",
      "tableArea": "",
      "startTime": "13:55",
      "endTime": "15:25",
      "description": "終演後物贩・特典会",
      "isStarred": false
    },
    {
      "id": "tokuten_3",
      "groupName": "かすみ草とステラ",
      "venue": "Spotify O-WEST",
      "tableArea": "",
      "startTime": "13:55",
      "endTime": "15:25",
      "description": "終演後物贩・特典会",
      "isStarred": false
    }
  ],
  "otherEvents": []
}
```

---

## 📥 How to Import into LivePulse

Generated JSON configuration files can be loaded into LivePulse using either method:
1. **Tab 1 (Festivals & Events)**: Click `📝 模版与智能解析` (Templates & Parser) ➔ Paste text or load JSON file.
2. **Tab 3 (Settings & Data Center)**: Drag and drop the `.json` file into the `📥 导入日程数据` (Import Schedules) dropzone. Choose **合并追加 (Merge)** to add to existing events, or **完全覆盖 (Overwrite)** to replace.
