# Kodi Add-ons

This repository will distribute Kodi add-ons through one installable repository. The first planned add-on, **FenSelect**, aims to make Play choose a stream you want to keep watching. It will rank the available files on the Kodi device, honor your language choice, and remember which file to try if you click Play again.

**Status · Planning.** There is no Kodi installer or release yet. Start with the [FenSelect roadmap](docs/fenselect-plan.md) for the approved playback behavior, implementation sequence, and acceptance checks.

| If you want to… | Start here |
| --- | --- |
| Understand why file size matters and how the three profiles will behave | [Ranking and worked examples](docs/fenselect-plan.md#ranking-by-file-size-and-quality) |
| Find the existing filters, settings, and retry loop | [Fen Light+ integration map](docs/upstream.md) |
| Pick up the next development task | [Milestones](docs/fenselect-plan.md#implementation-order) and [contributor instructions](AGENTS.md) |

Fen Light+ is the planned base. This project credits Tikipeter for Fen and Fen Light and thejason40 for Fen Light+. Git records the inspected revision as an optional submodule. To inspect it locally, run this from the repository root.

```sh
git submodule update --init --depth 1 upstream/FenLightPlus
```

Future add-ons can share a Kodi repository installer and update index while keeping independent IDs, versions, and release schedules. Installation instructions will accompany the first tested release. [Kodi repository format](https://kodi.wiki/view/Add-on_repositories)
