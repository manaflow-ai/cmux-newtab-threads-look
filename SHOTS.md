# Before and after

**Before** is cmux's built-in `threads` New Tab template at manaflow-ai/cmux `99039d5bf89c`: the webview
gallery's New Tab entry (`agent-pane.new-tab`, variants `look-threads-*`, `template-threads`), captured headless in
Chromium on Linux (`webviews/scripts/gallery/capture-states.mjs`).

**After** is this look: `customize/main.js` run in cmux's own app host (the native QuickJS VM,
`cmux-app-host`) with the same chats, and its scene drawn by cmux's web New Tab app page
(`NewTabAppPage` and `SceneRenderer`, the page `tabs.newTabApp` shows), captured the same way.
The install itself was checked in a nightly cmux build: `cmux setup install
github.com/manaflow-ai/cmux-newtab-threads-look --yes` wrote `customize/` and set `tabs.newTabApp`,
`cmux apps info local/customize` named this app, and `cmux setup remove` put everything back.

The fixtures: few = 3 chats (one working, one unread, one asking for input) plus 3 older ones;
six = 6 such chats plus 40 older; 10k = 14 plus 10,000 older; recently closed = few plus three
closed tabs. Ages can differ by a minute: each side stamps its fixtures when it runs.

| State | Before (light) | After (light) | Before (dark) | After (dark) |
| --- | --- | --- | --- | --- |
| empty | ![](screenshots/before-empty-light.png) | ![](screenshots/after-empty-light.png) | ![](screenshots/before-empty-dark.png) | ![](screenshots/after-empty-dark.png) |
| few | ![](screenshots/before-few-light.png) | ![](screenshots/after-few-light.png) | ![](screenshots/before-few-dark.png) | ![](screenshots/after-few-dark.png) |
| six | ![](screenshots/before-six-light.png) | ![](screenshots/after-six-light.png) | ![](screenshots/before-six-dark.png) | ![](screenshots/after-six-dark.png) |
| 10k | ![](screenshots/before-10k-light.png) | ![](screenshots/after-10k-light.png) | ![](screenshots/before-10k-dark.png) | ![](screenshots/after-10k-dark.png) |
| recently-closed | ![](screenshots/before-recently-closed-light.png) | ![](screenshots/after-recently-closed-light.png) | ![](screenshots/before-recently-closed-dark.png) | ![](screenshots/after-recently-closed-dark.png) |

## What does not match yet

What is the same: the field on top, Recently Closed under it, then Chats and Projects with a
filter and a search. Live chats lead (Needs input, then Running, then Unread), followed by a rule
and every other chat by age, dimmer. Each difference below names the API change that would
close it.

- **The field is a plain `TextField` and a Send button, not cmux's composer.** There is no folder
  or machine picker, no agent or model chip, no permission menu, no attachments, no microphone
  and no slash commands. The folder shows as a label above the field. Needs cmux's composer as a
  scene node.
- **The field's width.** The web scene gives a `TextField` no share of its row, so the look sets a
  wide frame that the row shrinks to fit. Needs `TextField` to fill its row, as the native one
  does.
- **Sending.** Return runs the `newTab.submit` action with the text. Today cmux sends a submitted
  prompt right away only when the person typed it into the built-in field, so an app's text may
  wait in the composer. Needs `newTab.submit` from an app's field to count as the person's.
- **All chats.** `agent_chat.list` returns every chat unsorted, so the look sorts them itself and
  shows 60 older rows at a time behind "Show N more". cmux's own list reads pages in order and
  scrolls without a button. Needs a sorted, limited read with a cursor.
- **Agent marks.** A scene `Icon` cannot draw the Claude or Codex mark that cmux's rows show, so
  rows use a state glyph (running, needs input, unread, chat). Needs harness marks as icons.
- **Recently Closed rows** show the name and age only. `closed.list` rows carry a kind (tab or
  workspace) but not what the tab showed (a page, a terminal, a chat), its address or folder, or
  whether it can still reopen, so there is no detail line and no per-content icon. Needs those
  fields on `closed.list`.
- **The filter menu** draws "All" without cmux's chevron. Needs a Menu style with a chevron.
- **Opening a chat** opens it in a new tab with `agent.openSessionTab`, not in the New Tab's own
  pane. Needs an "open here" action.
- **Install replaces `customize/`.** A setup's app is always `local/customize`, so this look
  replaces the person's own customize views and any other look. Needs setup-owned app ids.
