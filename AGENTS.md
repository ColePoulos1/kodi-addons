# Working on this repository

Read the [roadmap](docs/fenselect-plan.md) and [upstream integration map](docs/upstream.md) before changing FenSelect. They identify approved behavior, proposed tuning values, and the first milestone. This checkout currently contains plans and a pinned upstream reference.

## Where work belongs

- Treat `upstream/FenLightPlus` as a reference checkout. Initialize it with the command in the [README](README.md). The pinned commit makes the code map reproducible. Update the pin deliberately and recheck the mapped functions when adopting later upstream changes.
- For the future fork, use `addons/plugin.video.fenselect/`. Use a separate `addons/<addon-id>/` for each additional product. Keep repository generation shared. Record the upstream licensing evidence and retained notices before importing inherited code or artwork.
- Update the relevant milestone and evidence in the roadmap when an implementation changes its status. Use a tested release before adding user-facing installation steps.

## When working on selection

1. Trace candidate creation, filtering, playback handoff, and player feedback for the affected path. Include movie, episode, cached pack, manual selection, and next-episode callers when touching a shared function.
2. Reuse the settings store, filename tags, and resolver. Preserve manual ordering and explicit exclusions. Use one source of truth for the 3D toggle, size caps, codec exclusions, and provider pins.
3. Carry size units and provenance into scoring. Distinguish exact file bytes, estimated episode size, aggregate pack size, and missing data. Read runtime units at the boundary. Keep inference local to Kodi and training on the desktop.
4. Keep private watch history, raw candidate exports, magnet links, provider URLs, signed links, tokens, and Kodi userdata out of commits and test fixtures. Synthetic candidates are sufficient for ranking regressions.
5. Exercise the smallest reproducible comparison or playback sequence that demonstrates the change. Verify hardware-dependent claims on the named Pi and Kodi version. Record what ran and distinguish proposed budgets from measurements.

For documentation changes, inspect the staged paths, run `git diff --cached --check`, and follow the local Markdown links. There is no add-on build or test command at the planning stage.
