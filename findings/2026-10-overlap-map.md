# Open-PR overlap map — `yuliskov/SmartTube` (2026-10-15)

*Published 2026-10-15. Part of the [smarttube-lab](../README.md) findings series.*

English. Every number below is reproducible with the commands in the Method section; the
diff-level claims were verified against the actual PR diffs on 2026-10-15.

---

## What this is

A triage map of the 49 open pull requests against `yuliskov/SmartTube` as of 2026-10-15, produced by
running similarity detection over the queue and then **reading the diffs of every flagged group**.
Its single purpose: let a contributor know whether the thing they are building is already being
built by someone else — and let a reviewer know which groups cannot both be merged.

## Headline numbers

- **49 open PRs** (50 on 2026-10-14), **732 open issues**, one maintainer, ~97 commits/30 days.
- The similarity scan returns **5 groups covering 11 PRs**.
- **3 of those 11 are currently `MERGEABLE`** (`#5791`, `#6265`, `#5839`); the other **8 are
  `CONFLICTING`** on GitHub (re-verified 2026-10-15).
- After diff review: **one** group is a genuine duplicate pair (2 PRs, ~4% of the queue);
  **one** group is complementary; **three** groups are not duplicates at all — they collide on
  shared files.
- So the visible pathology in this queue is **not** people solving the same problem twice. It is
  **file-level collision**: many independent features editing the same hot UI files
  (`VideoMenuPresenter`, `GeneralSettingsPresenter`, `MainUIData`, the 46-locale `strings.xml`)
  and therefore conflicting with each other on the way in.

## The groups, after reading the diffs

### 1. Voice-over — the genuine duplicate pair (only one can ever land)
`#5817` *Add optional Yandex voice-over translation (EN→RU)* — mikhkz, 2026-05-25, 33 files
`#6265` *Add TTS voiceover to the TV player* — in0vik, 2026-09-24, 36 files

Both add **the same new file paths** with different designs:
`common/.../controllers/VoiceTranslateController.java`, `common/.../vot/VotClient.java`,
`common/.../vot/TranslationAudioPlayer.java`,
`smarttubetv/.../playback/actions/VoiceTranslateAction.java`.
Merging either one turns the other permanently conflicting. `#6265` is newer, targets RU/EN/KK
with a YouTube-dubbed-track fallback, and ships six unit-test files plus `docs/ANDROID_TV_TESTING.md`;
`#5817` is the older EN→RU-only implementation with its own VOT protocol stack.

### 2. Subtitles — complementary, but merge order matters
`#5839` *Add dual subtitle playback with translation support* — whyzgeek, 2026-05-30
`#6272` *Significant improvements to Subtitles* — botetta, 2026-09-27

`#5839` renders a second, translated subtitle line (ExoPlayer-fork `SubtitlePainter` extension +
`DualSubtitleCueMarkers`; `tlang` first, Google-Translate fallback). `#6272` adds SRV3 styling
support — colours, positions, bold/italics, font sizes — across the decoder, `SubtitleManager` and
`TrackSelectorManager`. The features **stack**, but they edit the same files
(`SubtitlePainter.java`, `SubtitleManager.java`, `PlaybackFragment.java`, `EmbedPlayerView.java`,
`lb_playback_fragment.xml`, `SubtitleSettingsPresenter.java`), so whichever merges second needs a rebase.

### 3. Playlists — not duplicates
`#5791` *Start mix — radio-style infinite playback* (bluegate-studio, 2026-05-19, **MERGEABLE**)
`#5790` *Shuffle playlists & channels via context menu* (bluegate-studio, 2026-05-19, conflicting)
`#6162` *Play playlist in reverse* (shiv1810, 2026-09-03, conflicting)

Three different features that all add an entry to the same context menus
(`VideoMenuPresenter`, `GeneralSettingsPresenter`, `MainUIData`).

### 4. Player error handling — same method, two retry families
`#6122` *cap format-error reloads per video* (draft since 2026-08-28, 0 comments, conflicting)
`#6188` *bot-block detection with bounded retries + client switch* (2026-09-07, 2 comments, conflicting)

Both patch `runFormatErrorAction()` in `ErrorFixerController.java` and both introduce a per-video
counter. In current `master` the file has gained `MAX_SABR_ERROR_FIX_ATTEMPTS = 3` for the SABR path
(commits of Oct 14–15, after both PRs) while `runFormatErrorAction()` still reloads unconditionally
in all three branches. Two independent counters in one method would need reconciling.

### 5. Settings: counter + settings search — a metric false positive
`#5437` *New videos counter* (asphalt123, 2026-02-28, 51 files, conflicting)
`#5495` *Search in Settings* (asphalt123, 2026-03-10, 60 files, conflicting)

Different features by the same author. They score as similar because **both rewrite ~46 locale
`strings.xml` files**; in a diff of that size the locale churn dominates. Listed here so nobody
acts on it.

## What this map is not

- It is not a review, and it is not a request. Groups 1 and 4 are the only ones where two PRs
  compete for the same outcome; everything here is public information already visible in the PR list.
- It does not claim any PR should be closed. Group 2 stacks, group 3 is three features, group 5 is
  noise.
- It is not a substitute for reading the diffs — the whole point is that the similarity score alone
  got three of five groups wrong.

## Method (reproduce)

```bash
# 1. similarity scan over the open PR queue
python3 find_dupes.py --repo yuliskov/SmartTube --limit 100 --workers 4 --out map.json

# 2. read the diffs of every flagged group (this is the part the tool cannot do)
for n in <numbers from the groups>; do gh pr diff $n --repo yuliskov/SmartTube | head -200; done

# 3. count merge state for the grouped PRs
for n in <numbers>; do gh pr view $n --repo yuliskov/SmartTube --json number,mergeable,isDraft; done
```

`find-dupes` is public: `github.com/josejim/find-dupes`. Its own README carries the lesson this map
produced: **the script flags, the human decides** — here the human step changed the headline number
from "22% of the queue is duplicate work" to "one pair, plus a lot of file collision".
