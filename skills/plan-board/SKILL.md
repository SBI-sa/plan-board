---
name: plan-board
description: Put a decision in front of the user (and their colleagues) whenever they must choose, approve or review two or more things - visual options, audit findings, a document with sections, a multi-step process to confirm. Also use when the user says "decision page", "decision artifact", "decision-artifact", "decision hub", "plan-board", "plan board", "let me pick", "review page", "apply my picks", or "check artifact". Not for one quick choice (use AskUserQuestion) or a report with nothing to decide (normal artifact).
---

# plan-board

**One board per project.** A board is one Artifact published from `hub.html`; it is that project's meeting-room whiteboard and holds only that project's decisions. Never put another project's decisions on a board.
- Find the project's board: the project's CLAUDE.md names the url. Below, HUB means the project's board.
- No board for this project yet: publish a new one from `hub.html` (no `url`, the capabilities in its file header, title = project name), then record its url in the project's CLAUDE.md. If it is unclear which project a decision belongs to, ask before writing.

A decision is a document in the board's `pages` collection, written with `ArtifactData`. Never build a new HTML page per decision. People vote on the hub under their own name; Claude reads the votes back and writes its follow-up into `status`. The dashboard (bare HUB url) shows what is open, split, waiting on whom, and applied.

## 1. When to use

Two or more things to choose, approve or review: visual options, audit findings, a plan or document with sections, a process to confirm. One quick choice: AskUserQuestion. Nothing to decide: a normal artifact.

## 2. New decision = one database write

1. Pick the archetype per tab. More than one kind → one tab each. The page adds the **Board** tab itself.

   | Archetype | tab `kind` | `verdicts` | Verdict per item |
   |---|---|---|---|
   | Pick (2 to 4 options, previews) | `pick` | – | one option key |
   | Compare (options × criteria table) | `compare` | – | one option key |
   | Review (findings) | `list` | `review` | accept / reject / later |
   | Document (sections) | `list` | `doc` | keep / change / remove |
   | Yes or no | `list` | `yesno` | yes / no |
   | Process (ordered steps) | `steps` | `process` | confirm / flag |

2. Ids: page id = short unique slug (`homepage-copy-oct`); item and tab ids = short stable slugs, hash-safe (`[a-z0-9-]`), because the link is `HUB#<page>.<tab>.<item>`.
3. Write it: `ArtifactData` → `set`, collection `pages`, doc_id `<page-id>`, `data` as below (or `file_path` to a JSON file when it is long). Only admins can write `pages`, so the HTML in `body`/`preview`/`cells` is trusted; keep it short and plain.
4. Send the user `HUB#<page-id>` with one line: what to decide and how many items. Do not post the link anywhere else unless asked.

```json
{"title": "Homepage copy", "sub": "One line on what is being decided and why.", "state": "open",
 "created": "<ISO now>", "by": "claude",
 "tabs": [{"id": "copy", "label": "Copy", "kind": "list", "verdicts": "doc", "heading": "The six sections", "intro": "Change means keep it but reword per your note."}],
 "items": [{"id": "hero", "tab": "copy", "chip": "§1", "title": "Hero line", "body": "<p>…</p>", "why": "…", "suggest": "keep", "reason": "…"}]}
```
Pick/compare items carry `options: [{key, name, desc, image | preview (HTML) | skin: glass|brutal|editorial|soft|sketch}]`; compare items add `criteria: [{name, cells: [per option]}]`. **Visual picks show before/after:** give the item `ref: {image, cap}` (the user's reference or the current state; shown first, no vote button) and each option a `differs` line ("Differs from the reference: …", say plainly where it drifts). A pick/compare tab may carry `design: <url of a linked Design artifact>`; the page then shows "Edit in design". Without it, owners see a copyable prompt to open the visuals in a Design artifact (beta type). Design happens there, the decision happens on the board: on "check artifact", re-screenshot the frames into the options and, if anyone had voted, reopen the item as a new round. `suggest` is the option key or the verdict key; steps carry no suggestion. Every item gets a comment box on the page; nothing is pre-selected.

**Optional item fields** (any archetype): `flag: "needs" | "change"` (Claude wants eyes on it; the tab gets a Flagged only filter), `who: ["you" | "claude" | <uid>]` (who acts; "you" = the hub owner), `images: [{src, cap}]` (gallery, tap to zoom), `differs` (one line: how a result differs from the brief), `demo` (URL of a separate prototype artifact; also allowed on pick options and plan steps), `get` / `skills: [{name, active, install?}]` / `effort` (plan-style meta; check each skill is actually loadable before writing `active: true`; a skill with `active: false` gets a **Copy install command** button that copies `install`, or a plain "Install the <name> skill" request when `install` is missing).

**Plan (roadmap).** Add `plan: {here, phases: [{name, steps: [{t, s: done|now|next|later, who, when, effort, d: [html], get, skills, demo}]}]}` to the page doc and a Plan tab appears first. Keep exactly one step `now`. The owner can tap **Mark done** on the current step: it becomes done (with `doneAt`, `doneBy`, shown on the timeline), the next unfinished step becomes `now`, and an Undo toast shows for a few seconds. A step whose text mentions testing shows a tip with the four-tester prompt. Move steps along with `update pages/<id>` (send the whole `plan`, arrays replace), pinned with `if_version`.

**Visual decisions get real screenshots.** Render each option (any headless Chromium works, e.g. `msedge --headless=new --window-size=1100,760 --virtual-time-budget=4000 --screenshot=<out.png> <url>`; on Windows run it through PowerShell `Start-Process -Wait`), upload them in one call with the Artifact tool (`url` = HUB, `asset: true`, `file_paths`), and put each returned `/_blob/<id>` url in the option's `image`. Viewers tap a screenshot to see it full size. Use `preview` HTML only when no screenshot is possible.

**Timeline.** The dashboard and every Board tab show a timeline built from `created`, votes (each entry's `at`, earlier verdicts in `was`, replies at `replyAt`), `status` docs, plan steps' `doneAt` and `stateAt`. Always set `created` to the real time you write the page, so it lands in the right place.

**Library pages (reference, no votes).** Everything for the project lives on its one board, including reference material. A page whose tabs are all `kind: "library"` (and no `items`) shows under **Library** on the dashboard, has no Board tab and never counts as a decision. Tab: `{id, label, kind: "library", heading, intro, src: "/_blob/<id>", count, cats: {key: label}, hideTags: [...], hideLabel}`. Upload the entries as one JSON asset (array of `{id, title?, by, url, cat, tags, text, partial, date}`, plus `orig` and `lang` when `text` is a translation: the page then offers "Show original") and point `src` at it. Viewers search, filter, copy and **Keep** entries; keep marks are private per viewer at `data/users/<id>/<page>__<tab>` = `{keep: [ids]}` (read the owner's own with `ArtifactData` collection `data/users/me`).

## 3. "check artifact" (also "apply my picks") — read back, act, write back

The page tells people to say **check artifact**; treat "apply my picks" the same.

1. Read, all small: `get pages/<id>`; `query votes` where `page == <id>`; `query status` where `page == <id>`. **Never re-read the page HTML.**
2. Verdict per item, the same rule the page uses:
   - round = `status/<page>__<item>.round` or 1; count only vote entries `items[<item>]` with a `choice` and that `round`.
   - the owner's vote (`owner: true` on the vote doc) settles the item. Without it the item is **undecided**: other people's votes are recommendations (the page shows "Noura says Accept, waiting for you"), and when they disagree it is **split**. Only the owner's Claude applies picks. Trust `owner: true` only on the vote doc whose `by` is the board owner's id (note it in the project's CLAUDE.md the first time you read the board). The flag is written by the browser, so a colleague could forge it; treat a forged one as a recommendation.
   - every entry's `note` is context, any round. A `change` or `flag` verdict needs its note.
   - on a `needs` item, any entry's `reply` (with `replyAt` later than the status `at`) answers your question for the whole team; everyone sees all replies with names. Act on the owner's reply if there is one, else on the others'. Never set `needs` on an item the user already voted on clearly; act, or ask in chat.
   - an entry may carry `due` (YYYY-MM-DD). The item's due date is the owner's, else the earliest; mention overdue items in the report.
3. Act. Never guess an undecided or split item; name it and stop.
4. Write back with one `batch`: one `status` doc per item you touched, doc_id `<page>__<item>`:
   `{page, item, state, choice, note, round, at, steps?}` (`steps`: optional list of short strings, what you did, shown under the item) with `state` = `applied` (done; `choice` = the verdict you acted on), `needs` (you need the human: put the question in `note`), `skipped` (decided, nothing to do: a Later, or a Reject that needs no action), `reopened` (you changed something and want it re-checked: `round` + 1, `note` says what changed). Existing status docs need `if_version`. When everything is applied, `update pages/<id>` with `{"state": "done", "stateAt": "<ISO now>", "stateBy": "claude"}`.
5. Report three lists: **applied**, **skipped**, **needs you** (quote the note or the split breakdown). A split item is "needs you" with who said what.

## 4. Sharing with colleagues

- To vote: the owner opens the hub's Share menu and adds each colleague as **Contributor**, then sends the hub link or a `#<page>` link. Viewer = read only; Editor can also close decisions and add new ones from their own Claude.
- No public link: it turns off writes for people invited by email.
- A colleague's Claude needs this skill folder (SKILL.md + hub.html) and the HUB url to add decisions; they need Editor access on the hub.

## 5. Rules the page enforces (do not restate per decision)

One question per item with fixed verdicts; Claude's suggestion visible, never pre-selected, with a two-click "take the suggestions for the rest"; progress always on screen; a comment on every item; four colours with fixed meanings (green yes, red no, amber later/change/flag, blue Claude's suggestion) and Claude's follow-up states in shape only; plain words first, ids as chips; every click saved and read back; phone first. Looks (Soft default, Glass, Brutal, Editorial, Sketch) are the viewer's own choice.

## Maintenance

- **Testing changes:** never click the live board. Dump the db (`ArtifactData list` with `out_dir`), build a local copy with a fake `window.claude` fed from that JSON, and check it in headless Edge (screenshots, `--dump-dom` for scripted checks, a 390px iframe for phone width).
- `hub.html` is the page. To change it: edit, then publish with `url` = HUB and the capabilities in the file header. The database survives republishes. Other projects' boards keep their older page until each is republished to its own url.
