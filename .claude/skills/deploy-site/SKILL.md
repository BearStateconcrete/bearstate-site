---
name: deploy-site
description: Edit and publish this website. Use whenever the user asks to change their site, update copy, swap photos, add a service, edit hours, change phone numbers, fix typos, or publish/push/deploy changes. After editing files, commits and pushes to GitHub, which auto-deploys to Vercel.
---

# Deploy Site Skill

This skill is how you (Claude on the client's KNYTEC workstation) help the client update their website.

## What this site is

The client owns a single-page website hosted on Vercel. The source code is in this folder. Every push to GitHub auto-deploys the live site within ~30 seconds.

- **Main file:** `index.html` — the entire site is here (HTML + inline CSS).
- **Photos:** `public/projects/` — replace files here to update gallery images.
- **Logo:** `public/logo-horizontal.png` and `public/logo-circle.jpg`.
- **Live site:** Whatever domain is configured. Check the Vercel project for the URL.

## When to trigger this skill

ANY request that touches the site. Examples:
- "Update my hours"
- "Change the phone number to 714-555-9999"
- "Add a new service"
- "Replace the patio photo"
- "Fix the typo in the headline"
- "Publish this"
- "Push the site"
- "Deploy"
- "Make it live"

If the user says something is wrong with the site, treat it as a request to fix and deploy.

## The Workflow

Always follow this order. Do not skip steps.

### Step 1 — Understand the request

Re-read what the user asked. Ask ONE clarifying question only if essential. Otherwise proceed.

### Step 2 — Edit the file

Use the Edit tool on `index.html` (or the relevant file). Make the smallest change that satisfies the request. Do not refactor surrounding code.

For photo swaps: replace the file at `public/projects/<name>.jpg` with the new image. Keep the same filename so the HTML still references it.

### Step 3 — Show the user what changed

Briefly describe the change in plain language. One sentence. Example: "Updated the phone number from 714-299-1630 to 714-555-9999 in the hero section and footer."

### Step 4 — Deploy

Run the deploy sequence. The git binary is at `C:\Program Files\Git\bin\git.exe`. Use `Start-Process` to capture output reliably:

```powershell
$git = "C:\Program Files\Git\bin\git.exe"
$log = "$env:TEMP\deploy.log"
$err = "$env:TEMP\deploy.err"
Set-Location <repo-folder-path>

Start-Process -FilePath $git -ArgumentList "add","." -RedirectStandardOutput $log -RedirectStandardError $err -Wait -NoNewWindow

Start-Process -FilePath $git -ArgumentList "-c","user.email=client@knytec.com","-c","user.name=KNYTEC Client","commit","-m","<commit-message>" -RedirectStandardOutput $log -RedirectStandardError $err -Wait -NoNewWindow
Get-Content $log
Get-Content $err

Start-Process -FilePath $git -ArgumentList "push" -RedirectStandardOutput $log -RedirectStandardError $err -Wait -NoNewWindow
Get-Content $log
Get-Content $err
```

**Commit message:** A short imperative sentence describing the change. Example: `"Update phone number to 714-555-9999"` or `"Replace patio finish photo"`.

If `git status` shows nothing changed before the commit, tell the user "No changes detected — your file may already be up to date" and stop. Do not force an empty commit.

### Step 5 — Confirm deployment

After `git push` succeeds, tell the user:

> Changes pushed. Your site will update in about 30 seconds at `<live-url>`. Refresh in a minute to see the change.

Do NOT poll Vercel. The git push triggers Vercel's auto-deploy via the GitHub integration — it works on its own. The client doesn't need a live deployment status check.

## What NOT to do

- **Never run `vercel --prod` or any Vercel CLI command.** The client's machine doesn't have Vercel CLI credentials. Git push is the only deploy mechanism.
- **Never create new repos or new Vercel projects.** Both are managed by KNYTEC at the org level.
- **Never edit files outside this folder.** Your filesystem access is scoped to this folder.
- **Never commit secrets, tokens, or API keys.** If the user asks you to add an API key, stop and tell them: "API keys should be added through KNYTEC support — they live in Vercel environment variables, not in the code. Email support@knytec.com."
- **Don't push back on small content changes.** If the user wants their phone number changed, change it. Don't second-guess wording requests.
- **Push back on structural changes.** If the user wants to add a new section, rebuild the layout, or change the design system, tell them: "This is a bigger change — let me email KNYTEC so we can plan it together. For now I can [smaller alternative]." Then offer the smaller alternative.

## Troubleshooting

**`git push` fails with auth error**

The PAT may have expired. Tell the user:

> Looks like the GitHub credential expired. Email support@knytec.com and we'll get it refreshed — usually takes a few minutes.

Do not attempt to regenerate credentials yourself.

**`git push` says "Everything up-to-date"**

The change wasn't committed. Re-check `git status`. If still empty, the file was likely never modified.

**Vercel hasn't updated after 2 minutes**

Tell the user to hard-refresh their browser (Ctrl+Shift+R on Windows). If still not updated after 5 minutes, escalate: "Something on the hosting side is slow. I'll email KNYTEC to check on it." Then stop.

## Brand voice for copy edits

When writing or editing copy on the site:
- Plain English. No corporate language.
- Short sentences.
- Direct.
- Active voice.
- No "synergy", "leverage", "unlock", "transform" or similar.

If the user asks you to write new marketing copy, match the existing tone in `index.html`.

## Examples

**Example 1 — Phone number change**

User: "Change my phone to 714-555-9999"

You:
1. Edit `index.html`, replace all instances of the old phone with `714-555-9999` (including `tel:` links).
2. Say: "Updated phone number to 714-555-9999 across the hero, services, and footer."
3. Run deploy sequence with commit message `"Update phone number to 714-555-9999"`.
4. Say: "Pushed. Live in about 30 seconds."

**Example 2 — Photo swap**

User: "Replace the firepit photo with this new one" + attaches new image

You:
1. Save the attachment as `public/projects/firepit-bench.jpg` (same filename, overwrites).
2. Say: "Swapped the firepit photo with your new one."
3. Run deploy sequence with commit message `"Update firepit photo"`.
4. Say: "Pushed. Live in about 30 seconds."

**Example 3 — Out of scope**

User: "I want to add a blog section with five posts and a newsletter signup"

You:
1. Don't start editing.
2. Say: "That's a bigger build — let me loop in KNYTEC so we plan it right. I'm sending support@knytec.com a note now. In the meantime, want me to add a placeholder 'Blog coming soon' line to your nav?"
3. Either email support OR (if no email tool available) tell the user to email support@knytec.com themselves.
