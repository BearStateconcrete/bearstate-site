---
name: website-edit-publish
description: Edit and publish changes to the Bear State Concrete website. Use this skill whenever Hugo wants to change text, copy, images, layout, or any content on bearstateconcrete.com. Triggers include: "update the website", "change the site", "edit the homepage", "update the copy", "fix the wording on the site", or any request to modify site content and push it live. Also use when Hugo asks to publish, deploy, or push site changes to GitHub/Vercel.
---

# Website Edit & Publish — Bear State Concrete

## Overview

The site is a single static HTML file deployed via GitHub → Vercel auto-deploy.

- **Local file:** `C:\KNYTEC\Sites\bearstate-site\index.html`
- **Repo:** `https://github.com/BearStateconcrete/bearstate-site`
- **Branch:** `main`
- **Live URL:** `https://bearstateconcrete.com`
- **Vercel:** Auto-deploys ~60 seconds after push to `main`

---

## Step 1 — Read the file

Before making any edits, read the current file:

```
Desktop Commander:read_file
path: C:\KNYTEC\Sites\bearstate-site\index.html
```

Scan for every instance of the text/section Hugo wants to change. The file is ~1071 lines — use `view_range` if needed to zero in on a section.

---

## Step 2 — Make edits

Use `Desktop Commander:edit_block` for all edits. Rules:
- One focused edit per block — don't bundle unrelated changes
- `old_str` must be unique in the file and match exactly (copy from read output)
- For text scattered across multiple sections, make a separate edit_block call per instance
- Always re-read the file after edits if making further changes to the same area

```
Desktop Commander:edit_block
file_path: C:\KNYTEC\Sites\bearstate-site\index.html
old_string: <exact text to replace>
new_string: <replacement text>
```

---

## Step 3 — Commit and push

Git is available via **cmd shell only** (not PowerShell — git is not in the PowerShell PATH).

Write the commit message to a temp file first to avoid shell quoting issues:

```
Desktop Commander:write_file
path: C:\KNYTEC\Sites\bearstate-site\commitmsg.txt
content: Your commit message here
```

Then commit and push:

```
Desktop Commander:start_process
shell: cmd
timeout_ms: 30000
command: cd C:\KNYTEC\Sites\bearstate-site && git add -A && git commit -F commitmsg.txt && git push && del commitmsg.txt
```

---

## Known issues & fixes

**Git not recognized in PowerShell:**
Always use `shell: cmd` for git commands. PowerShell does not have git in its PATH on this machine.

**`&&` not valid in PowerShell:**
Use semicolons (`;`) for chaining in PowerShell, or just use cmd.

**"Author identity unknown" error:**
Git user config was missing. Already fixed — set globally to:
- `user.email = hugo@bearstateconcrete.com`
- `user.name = Hugo Aguilar`
Should not recur. If it does, run:
```
git config --global user.email "hugo@bearstateconcrete.com"
git config --global user.name "Hugo Aguilar"
```

**Commit message pathspec errors:**
If the commit message contains spaces and quotes get stripped, write message to `commitmsg.txt` and use `git commit -F commitmsg.txt` instead of `-m`.

---

## Site structure reference

Key sections and what they control:

| Section | ID | Notes |
|---|---|---|
| Nav | `.nav` | Logo, links, phone, CTA button |
| Hero V1 | `#hero-v1` | Active hero — full bleed dark photo |
| Hero V2 | `#hero-v2` | Hidden (`hero-hidden`) — editorial split |
| Hero V3 | `#hero-v3` | Hidden (`hero-hidden`) — typographic |
| Marquee | `.marquee` | Scrolling service ticker |
| Services | `#services` | 6 service cards |
| Recent Work | `#work` | 5 project cards with photos |
| Process | `#process` | 4-step process |
| About | `#about` | Company story + stats |
| Service Area | `.area-section` | City list + region descriptions |
| Contact/Quote | `#quote` | Form + contact info |
| Footer | `.footer` | Brand copy, links, license |

**Hero variants:** Only V1 is visible. To switch heroes, remove `hero-hidden` class from the desired variant and add it to V1.

**Project photos** live in `C:\KNYTEC\Sites\bearstate-site\public\projects\`

---

## After pushing

Vercel deploys automatically in ~60 seconds. Verify at `https://bearstateconcrete.com`.
