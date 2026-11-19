# FenSelect development plan

**Status · Approved direction, November 19, 2026.** The intended experience is one click on a movie or episode. FenSelect chooses an eligible file, starts playback, and can try the next distinct file when the first one fails or the viewer quickly stops and clicks again. The initial target is a Raspberry Pi or a similarly constrained Kodi device using TorBox and the viewer's existing external scraper. The first implementation milestone is a movie and an episode working end to end.

## What the viewer controls

| Setting | Initial behavior |
| --- | --- |
| Playback profile | Default to **Balanced**. Offer **High Quality** and **Data Saver**. A profile supplies soft preferences. Existing explicit size caps, format exclusions, and device limits take precedence. Switching profiles preserves those settings. |
| File and device filters | Reuse existing settings for 3D, resolution, codecs, HDR/Dolby Vision, audio formats, size, and line speed. Default known 3D files to excluded. Respect explicit device capability overrides when detection is inconclusive. |
| Source preference | Prefer a good Blu-ray encode over a comparable WEB-DL and HDR on a compatible display. Keep existing provider/folder pins and saved autoplay format priorities meaningful. Use a context-menu action for the existing manual list. |
| Language | Use the viewer's configured default audio language and subtitle language, starting from Kodi's preferences. Add **Ask on first play**, **Prefer Sub**, or **Prefer Dub** independently for Movies, TV, and Anime. Default each to Ask. |

Use the existing settings UI. Give the viewer one profile selector and the three language choices. Keep size tuning inside the profile until real comparisons reveal a decision people need to override beyond the existing caps. The [integration map](upstream.md) identifies the controls already present.

## Ranking by file size and quality

**File size per minute is the main quality signal among otherwise eligible, comparable streams.** A good candidate for the profile should routinely beat an implausibly small encode with more attractive filename tags. A 40 GB file might be excellent for a three-hour feature, excessive for an episode, or merely the sum of ten episodes.

1. **Establish eligibility.** Match the movie or episode and check playability, hard user exclusions, explicit caps, and session skips. Keep confirmed language matches ahead of uncertain ones when the user requests a particular track. Preserve explicit folder/cloud pins among eligible candidates.
2. **Normalize size.** Compute decimal MB/min as selected-file bytes divided by 1,000,000 and runtime in minutes. Upstream metadata duration used in size filtering is in seconds. Convert provider-specific units once. Carry raw bytes when possible and retain whether the size is exact, estimated per episode, aggregate only, or unknown.
3. **Choose a size band.** Use the profile, movie/episode context, source class, and resolution. A film grain restoration and a low-motion cartoon warrant different expectations. Treat age, genre, and animation as modest adjustments. Use a wider band or reduced confidence when the content classification, codec efficiency, or runtime is uncertain.
4. **Score distance from that band.** Penalize substantially undersized releases sharply for High Quality. Give Balanced diminishing returns above its preferred range. Data Saver favors the smallest release that clears a plausible quality floor and penalizes excess. Saturate the size benefit so extra audio tracks or inflated files cannot win indefinitely. An exact size carries more weight than a pack-derived estimate. Missing size gets a neutral size contribution unless the user explicitly excludes unknowns.
5. **Apply soft preferences and ties.** Add smaller contributions for source class, supported HDR, and ordered user format preferences. A similarly sized Blu-ray encode should normally beat WEB-DL under Balanced or High Quality. A suitable WEB-DL can beat a severely undersized Blu-ray. For equal scores use the saved upstream sort, then a stable file identity. Keep the original manual ordering intact.

Start with a small, deterministic weighted sum of bounded features in plain Python. Piecewise size features express the asymmetric costs above. Use the same feature definitions for desktop training and on-device inference. Initial weights are hand-tuned. Compare the three profiles with the examples below before setting numeric defaults. Record the chosen bands, units, and evidence with the first implementation.

### How to use TRaSH Guides

Radarr and Sonarr express quality definitions in MB/min, distinguish source and resolution, and treat animation more permissively. For example, their standard 1080p WEB-DL minimums at the time of review are 12.5 MB/min for Radarr and 15 MB/min for Sonarr. These are calibration references for movies and episodes. The guides' highest-quality and effectively unlimited upper settings serve library acquisition. FenSelect needs its own finite preferred bands for streaming and explicit connection limits. [Radarr size definitions](https://trash-guides.info/Radarr/Radarr-Quality-Settings-File-Size/) · [Sonarr size definitions](https://trash-guides.info/Sonarr/Sonarr-Quality-Settings-File-Size/)

Radarr groups Blu-ray, WEB, and remux choices into different profiles. Sonarr documents WEB-focused television profiles and alternatives for older shows. Its anime guide also treats dual audio as a configurable preference. Use those distinctions to design comparisons across source classes and languages. A Kodi device can evaluate its own bundled weights offline. There is no need to sync custom-format catalogs at playback time. [Radarr profiles](https://trash-guides.info/Radarr/radarr-setup-quality-profiles/) · [Sonarr profiles](https://trash-guides.info/Sonarr/sonarr-setup-quality-profiles/) · [Sonarr anime profiles](https://trash-guides.info/Sonarr/sonarr-setup-quality-profiles-anime/)

### Worked acceptance examples

Sizes below use decimal GB. These are synthetic comparisons for tuning, with matching tracks and compatible devices unless specified. They express the expected ordering without prescribing a universal bitrate.

| Candidate set | Expected decision |
| --- | --- |
| 120-minute cinematic feature, equal resolution and codec. 1.2 GB Blu-ray encode (10 MB/min) or 7.2 GB WEB-DL (60 MB/min). | Balanced and High Quality pick the WEB-DL. Its size advantage outweighs the Blu-ray label. |
| The same feature, 7.2 GB Blu-ray encode or 7.2 GB WEB-DL with comparable mastering and tracks. | Prefer Blu-ray. Size provides no reason to reverse the source preference. |
| 22-minute, low-motion animated episode. Competent 0.55 GB encode (25 MB/min) or 2.2 GB encode (100 MB/min), otherwise equivalent. | Data Saver picks the smaller file. Balanced should avoid spending four times the data for little expected benefit. |
| One 45-minute episode in a 40 GB pack. Its indexed file is 1.8 GB. | Score 40 MB/min. If only the pack total exists, mark size uncertain. Never score the episode as a 40 GB file. |
| Older film with a known high-detail restoration. Larger file fits the High Quality band and configured line speed. | Keep the larger restoration eligible. Release year alone cannot impose a low size limit. |
| Highest-scoring candidate is excluded 3D, over an explicit cap, or confirmed to lack the required track. | Advance to an eligible candidate. The score never overrides the exclusion. |

## Language on first play

Use reliable original-language metadata and compare normalized language codes. Anime overrides Movies/TV only for identified anime, including anime films. Generic animation or Japanese audio alone is insufficient to classify a title. If original language is missing, honor normal track preferences and skip the prompt.

For a known mismatch, apply the category preference. In Ask mode, offer concrete choices such as **Sub · Japanese audio, English subtitles** and **Dub · English audio**. Sub prefers original audio plus full subtitles in the configured subtitle language, falling back to the user's default language when a language must be chosen. Dub prefers audio in the user's default language and otherwise respects their Kodi subtitle settings. For matching original audio, play directly with Kodi's ordinary subtitle behavior. [Kodi language settings](https://kodi.wiki/view/Settings/Player/Language)

Remember the answer per Kodi profile and movie or series. Provide a title context-menu override and a way to forget it. Reuse the answer on retries and later episodes. Canceling the dialog ends the request. Dual-audio files can fulfill either choice. Prefer verified metadata, allow unknown tags to reach playback, and inspect the actual streams after opening the selected file. Distinguish full-dialogue subtitles from forced-only tracks. On confirmed mismatch, try the next file. When none match, explain the missing track and offer the available alternative once. Do not require a language popup on every Play.

## Playback, retry, and resume

| Event | Action and evidence to retain |
| --- | --- |
| First Play | Rank eligible entries, resolve the best, and open it. Keep the current attempt, alternate candidates, and requested language local to the active Kodi profile. |
| Resolver/startup failure or confirmed playback error | Let the existing resolver advance. Scope transient failures to this attempt or a short cooldown. Treat a service-wide authentication failure as a provider problem. Keep the resume point after a midstream error. |
| User stops | End playback immediately. Remember the last file and stop time. Never start another stream in response to Stop alone. |
| User promptly clicks the same movie or episode | Try the next distinct file. A provisional quick-reclick window is two minutes. Validate the boundary on the Pi and make it easy to tune in one place. |
| User returns later | Prefer the last working eligible file and restore normal bookmark behavior. Changing the language preference or hard filters triggers a fresh eligibility check. |
| All eligible files exhausted | Finish with a concise reason and allow an intentional retry or manual choice. Keep automatic recovery bounded even after rescrapes. |

Key attempts by profile, media ID, and episode as applicable. Prefer torrent hash plus normalized file path or provider file ID scoped to its parent. Fall back to a provider-scoped file identity if a cross-provider identity is unavailable. A signed playback URL can change for the same file. A user rejection skips all known routes to that file. A transport failure may try another working route. Handle late player callbacks and quick double clicks against an attempt ID so an old session cannot reject the current stream. A single exit is weak evidence for long-term quality preferences.

## Implementation order

| Order | Deliverable | Evidence for completion |
| --- | --- | --- |
| 1. Establish the baseline | Record applicable upstream reuse terms. Inspect one sanitized movie and episode candidate set from the actual TorBox/Prowlarr path. Record Pi model, Kodi/OS version, HDR/audio capabilities, and effective connection cap. Trace exact sizes through the current filters and pack shortcut. | Identify which fields carry size, units, confidence, runtime, identity, original language, and format tags. Mark missing fields explicitly. Observe stock autoplay, manual pick, and resume on the target device. |
| 2. Build the smallest playable fork | Give FenSelect its own add-on ID, storage/settings namespace, plugin routes, and update source. Reuse upstream filters, then wire a pure-Python ranker and the three profiles into autoplay. Check widgets, background next-episode, ignored-filter recovery, cached packs, and manual choice. | Side-by-side installation with Fen Light+ preserves separate settings. A movie and episode use size-aware ranking. Worked comparisons pass. No extra resolver or network request per candidate is needed for scoring. |
| 3. Complete one-click behavior | Add language preference and track selection, source identity across clicks, and bounded retry. Reuse Kodi player events and existing resolver/bookmark behavior. | On the Pi, demonstrate a bad first link, quick stop/reclick, later resume, a dual-audio title, missing required track, a second episode without another prompt, and exhaustion. Record time to first playback separately from local ranking. |
| 4. Prepare the first Kodi release | Put independently versioned add-ons under `addons/<addon-id>/`. Build a neutral repository add-on with its own ID, a shared `addons.xml` index and checksum, and versioned ZIPs. Publish the tested feed, with its installer and instructions. | A fresh Kodi profile installs the repository and FenSelect, receives a test version update, and leaves a co-installed upstream add-on alone. Validate the index can describe a second distinct add-on using a disposable fixture. [Kodi repository format](https://kodi.wiki/view/Add-on_repositories) |
| 5. Learn from real choices | Capture optional local candidate comparisons. Build a small desktop chooser, fit pairwise weights offline, and evaluate on unseen titles. | Learned weights improve held-out first-choice agreement over the hand-set baseline while passing every eligibility and playback test. Keep the baseline if improvement is inconclusive. |

### First implementation session

Initialize the pinned source and read the [integration map](upstream.md). Start with the same title returned via the TorBox cloud path and the configured external scraper. Compare exact file bytes with the displayed size, locate where a pack estimate is produced, and check whether current size filtering discards the file before a ranker could see it. Retrieve only the metadata needed for the comparison. Record findings before choosing thresholds or adding a settings screen.

### Performance and learning

Measure ranking on the slowest supported device using a fixed candidate set, both cold and warm. Report candidate count, added latency (median and p95), and incremental memory separately from scrape, resolve, and time to first frame. An initial engineering target is under 50 ms p95 to rank 200 already-parsed candidates on the test Pi. Treat this as a proposed budget until measured. Keep scoring dependency-free, parse each candidate once, and resolve lazily.

For training, save the context and normalized features for all candidates in a decision, the active profile, and the viewer's comparison. Present **A**, **B**, **either**, or **neither**. Fit a small linear pairwise model on the desktop. Group the train/test split by movie or series to avoid nearly identical episodes in both sets. TorBox's torrent list can supply representative sizes. Prowlarr query/grab history can help find searches. Neither records the viewer's full choice set or confirms a good viewing experience. Synthetic comparisons test policy and edge cases. [TorBox API](https://api-docs.torbox.app/) · [Prowlarr history events](https://github.com/Prowlarr/Prowlarr/blob/develop/src/NzbDrone.Core/History/History.cs)

Track first-choice agreement by profile and media type, first-click sustained playback, technical failure rate, and quick-reclick rate. Compare first-click success with the same eligible candidate sets where possible. Keep technical reliability labels separate from aesthetic preferences. Keep watch data local by default and exports opt-in. Deliver validated weights with a matching feature version and a fallback to bundled defaults.

## Decisions still open

Exact preset bands, the first target Pi/Kodi combination, reliable anime metadata in this setup, and the final repository add-on ID require implementation evidence or maintainer input. Confirm user-facing settings with a few genuinely ambiguous pairs, particularly an older restoration, an animated feature, and a WEB-DL versus Blu-ray tradeoff. These decisions can be made independently of building the desktop trainer.
