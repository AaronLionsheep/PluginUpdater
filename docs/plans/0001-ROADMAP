# Plugin Updater for Indigo — Roadmap

Nov 23, 2026 · @Aaron

## Goal and principles

Plugin Updater lets Indigo users check, approve, install and roll back plugin updates from any remote client, without sitting at the server Mac. The roadmap ships read-only value first and adds install powers only once each safety layer is in place.

- **Manual approval by default.** Automatic updates are opt-in, per plugin, and limited to patch releases.
- **Opening a link never installs.** Every install is a POST with a CSRF token, an `Origin` check and, optionally, a TOTP code.
- **Every install is reversible.** The previous version is kept outside the Plugins folder, so rollback is one action.
- **One install path.** Email links, the web UI, Indigo actions and schedules all feed the same job queue.
- **Allowlisted sources only.** Plugin Store and GitHub Releases over HTTPS, with checksums where available.

## Phase 0 — Foundations and spikes

Phase 0 proves the Indigo assumptions the whole design rests on, with throwaway scripts on a test server. Each spike either confirms the plan or changes it before real code exists.

- [ ] Swap an enabled plugin's bundle on disk, then call `indigo.server.getPlugin(id).restart()`, and confirm the new version loads.
- [ ] Install a brand-new plugin by copying it into `Plugins/` and find out whether Indigo enables it or needs a manual step.
- [ ] Confirm backups stored outside the Plugins folder are ignored, and no duplicate-ID warnings appear.
- [ ] Confirm `getInstallFolderPath()` and the disabled-plugins folder name on your Indigo version.
- [ ] Restart the updater plugin from inside itself and confirm it comes back cleanly.
- [ ] Serve a GET and a POST through `/message/<plugin id>/<action>/` via the reflector, and note the reply format.
- [ ] Run the CSRF test page against the handler to learn whether the browser attaches Indigo credentials cross-site.
- [ ] Check whether `indigo.schedule` can be created or only duplicated, and which timing fields a plugin can change.
- [ ] Call the Plugin Store version API and the GitHub Releases API for a few real plugins and save sample responses.

## Phase 1 — Inventory and update checks (MVP)

Phase 1 ships a read-only plugin that knows what is installed and what is newer. It cannot change anything, so it is safe to release early and get feedback.

- [ ] Inventory: scan `Plugins/` and `Plugins (Disabled)/`, read each `Info.plist`, and tag running plugins with `getPluginList()`.
- [ ] Sources: map each bundle ID to a Plugin Store entry or GitHub repo, with a user-editable override list.
- [ ] Version parsing: normalize tags into tuples, skip unparseable tags, and exclude drafts and prereleases by default.
- [ ] Classify each update as latest patch, latest minor or latest major.
- [ ] Cache release lists and respect the GitHub rate limit of 60 requests per hour unauthenticated.
- [ ] Check timer in `runConcurrentThread`, with the interval set in plugin preferences.
- [ ] "Check for updates" action, so native Indigo schedules can drive checks too.
- [ ] Expose a pending-update count as a device state for control pages.
- [ ] Log a summary of pending updates to the Indigo event log after each check.

## Phase 2 — Notifications

Phase 2 makes the plugin tell you about updates instead of waiting to be asked. It still installs nothing.

- [ ] "Updates available" event in `Events.xml`, fired once per check as a digest, not once per plugin.
- [ ] Event data on Indigo 2025.1+: a rendered `subject` and `body`, plus a structured `updates` list for scripts and Pushover.
- [ ] Fallback for older Indigo: write the subject and body to variables before firing the trigger.
- [ ] Plain-text email template using `string.Template`, overridable in plugin preferences.
- [ ] Only notify when something changed since the last digest, so the same updates don't repeat daily.
- [ ] Document a sample trigger that sends the digest with Indigo's Send Email action.

## Phase 3 — Installer, backups and rollback

Phase 3 adds the ability to change plugins, driven only from Indigo actions so it can be tested before anything is exposed remotely.

- [ ] Job queue with a worker thread, so action callbacks return immediately.
- [ ] Download to a temp folder over HTTPS and verify the checksum or release digest where available.
- [ ] Unzip with `zipfile`, then validate the bundle ID, `ServerApiVersion` and minimum Indigo version from `Info.plist`.
- [ ] Strip `com.apple.quarantine` with `xattr -dr`, and flag bundles that contain native code (Mach-O files).
- [ ] Atomic swap: stage on the same volume, move the old bundle to the backup folder, rename the new one in, restart the plugin.
- [ ] Keep N previous versions per plugin, with N set in preferences (default 1).
- [ ] Actions: update to latest patch, minor or major; update to a specific version (cascading menus fed from the cache); roll back.
- [ ] Warn on major releases and downgrades in action dialogs.
- [ ] "Update completed" and "Update failed" events with event data.
- [ ] Self-update last: swap its own bundle, then restart itself without waiting.

## Phase 4 — Remote web UI and email confirmation

Phase 4 brings install powers to remote clients through the Indigo reflector, with the CSRF defences in place from the first release.

- [ ] Hidden HTTP-handler action that routes on the first path segment: pages, JSON API and job status.
- [ ] Self-contained, mobile-first page: phone layout by default, centered column from about 768px, card layout from about 1024px.
- [ ] Screens: plugin list, plugin detail with version picker and rollback, confirm and progress.
- [ ] CSRF token on every POST, `Origin` check, and JSON-only API endpoints.
- [ ] One-time confirmation tokens: `secrets.token_urlsafe(32)`, bound to plugin, version and checksum, expiring in 48–72 hours.
- [ ] Review links in the digest email, opening a confirm page (GET) that installs only on POST.
- [ ] Return 410 Gone for used, expired or superseded tokens.
- [ ] Progress polling from `/job/<id>`, and a result email when the job finishes.
- [ ] Pick the visual direction from the canvas sketches (currently the one-mode grey industrial style).

## Phase 5 — Hardening and optional extras

Phase 5 adds features that are useful but not needed for a safe first release.

- [ ] Optional TOTP: enrollment shown only in the server-side config dialog, ±1 time step, no code reuse, lockout after failures, recovery codes.
- [ ] Enforce TOTP on every install path when enabled, including the setting that turns it off.
- [ ] Opt-in automatic updates, per plugin, limited to patch releases.
- [ ] Managed schedule: let the user pick an existing schedule for the plugin to enable, disable and run.
- [ ] Show `codesign` status for bundles with native code.
- [ ] Activity screen: history of checks, installs and rollbacks.
- [ ] Optional dimmed palette when the phone is in dark mode.

## Open questions and risks

| Question or risk | Why it matters | Settled by |
| --- | --- | --- |
| Can a newly copied plugin be enabled without the server UI? | Decides whether first installs work remotely or only updates do | Phase 0 spike |
| Does the browser attach Indigo credentials to cross-site POSTs? | Sets how urgent the CSRF defences are, though they ship either way | Phase 0 CSRF test |
| Is the Plugin Store version API still supported? | Without it, GitHub becomes the only source and the rate limit matters more | Phase 0 API check; ask Indigo |
| Can plugins create schedules? | Decides how the managed-schedule feature works | Phase 0 spike |
| Many releases lack checksums | Verification falls back to HTTPS and the allowlist only | Phase 3 design |
| A compromised source could ship malicious code | The plugin runs arbitrary code as the Indigo user | Allowlist, manual approval, native-code warnings |
| Native code blocked by Gatekeeper on a headless server | A plugin fails to start with no visible prompt | Phase 3 quarantine handling and test |
| Self-update interrupts its own HTTP reply or job | Update could fail halfway | Phase 3 self-update ordering |

## Milestones

&#91;embedded content: roadmap · 6 phases, each ending in a gate or release\]

Each phase ends in a release you can use on its own; the version numbers are suggestions.
