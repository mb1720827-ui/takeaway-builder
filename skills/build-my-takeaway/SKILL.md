---
name: build-my-takeaway
description: Use when someone wants their own "save videos to learn later" tool - a personal inbox they share TikTok, YouTube, Instagram or article links into from their phone, where an AI agent watches each one and turns it into notes, collections and a review queue. Builds and deploys their own copy of Takeaway step by step, customised to their interests.
---

# Build my own Takeaway

You're helping someone set up their own copy of [Takeaway](https://github.com/mb1720827-ui/takeaway): they share a video or link from their phone in two taps, and a worker on their computer uses Claude Code to read the captions, look at the frames and write learning notes into their dashboard.

Work through the steps in order. Do the technical work yourself, and ask the person only for things only they can do (signing in to accounts, pasting secrets into their own files, tapping on their phone). Confirm each step works before moving on.

## Step 0: Ask three questions (one message)

1. **Phone:** Android or iPhone?
2. **Interests:** what kinds of videos do they save? This becomes their starting collections. The defaults are AI Tips, AI Innovation, Software Architecture, Inspiration and Life Tips.
3. **Supabase:** do they already have a Supabase account and a project they want to use, or should they create a new free project?

## Step 1: Check the computer

The worker runs on their computer, which must be on for saves to be processed. Check each tool and install what's missing:

| Tool | Check | Install |
|---|---|---|
| Node.js 20+ | `node -v` | nodejs.org |
| Git | `git --version` | git-scm.com |
| Claude Code | `claude --version` | `npm install -g @anthropic-ai/claude-code`, then run `claude` once to sign in |
| yt-dlp | `yt-dlp --version` | Windows: `winget install yt-dlp.yt-dlp` · macOS: `brew install yt-dlp` · Linux: `pipx install yt-dlp` |
| ffmpeg | `ffmpeg -version` | Windows: `winget install Gyan.FFmpeg` · macOS: `brew install ffmpeg` · Linux: `sudo apt install ffmpeg` |

Optional: `pip install whisper-ctranslate2` adds speech-to-text for videos without captions.

## Step 2: Get the code

```bash
git clone https://github.com/mb1720827-ui/takeaway my-takeaway
cd my-takeaway
npm install
```

If they want their own GitHub repository, create one (`gh repo create my-takeaway --private --source . --push`) after this step.

## Step 3: Customise it

- **Collections:** edit `DEFAULT_COLLECTIONS` in `src/lib/notes.ts` to match their interests from Step 0.
- **Focus:** if their saves aren't about software (for example cooking, fitness or design), adjust the wording in `worker/prompt.md` (for example "explain concepts simply, for a home cook") and the explanation fields in `src/lib/notes.ts`.
- Run `npm test` afterwards.

## Step 4: Database (Supabase)

1. Ask them to create a free project at supabase.com, or pick an existing one. Note that the free plan allows 2 active projects.
2. Run `supabase/migrations/20261124100000_takeaway.sql`. Either they paste it into the Supabase SQL Editor, or you run `npx supabase db query --remote=<project-ref> --file supabase/migrations/20261124100000_takeaway.sql` after `npx supabase login`.
3. Ask them for the **project URL** and the **publishable (or anon) key**: Project Settings › API. **Never ask for or use the secret/service-role key.** Nothing here needs it.
4. Create `.env.local` from `.env.example` and fill in `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY`.

## Step 5: Deploy the app

1. Push the code to their GitHub. They then import it on vercel.com, or you deploy with `npx vercel` if they're signed in.
2. Set the two `NEXT_PUBLIC_SUPABASE_*` variables in Vercel. If Vercel warns that `NEXT_PUBLIC_` values are exposed, that's fine for these two (choose "Config").
3. If the build complains about a missing `public` folder, the framework preset is wrong. `vercel.json` already pins `"framework": "nextjs"`, so redeploy.
4. Get their exact live URL (for example `my-takeaway-abc.vercel.app`) and add `https://<url>/**` to Supabase › Authentication › URL Configuration › Redirect URLs. Use the **exact** domain, never a wildcard such as `*.vercel.app`: other people's apps live on that domain too.

## Step 6: Account and inbox token

1. They open the live app, choose **Create account**, then go to **Settings** and **Create token**. They must copy it: it's shown once.
2. Add the worker settings to `.env.local` on their computer: `TAKEAWAY_EMAIL`, `TAKEAWAY_PASSWORD` (the same account) and `TAKEAWAY_APP_URL`. They should paste the password themselves; don't ask them to send it to you in chat.

## Step 7: The share button on their phone

**Android (HTTP Shortcuts, free on the Play Store):**
1. Create a variable `shared` with "Allow receiving value from Share dialog" turned on.
2. Create a shortcut "Save to Takeaway": `POST https://<their-url>/api/inbox`, header `Authorization: Bearer <token>`, body custom text, `text/plain`, containing only the `shared` variable.
3. Under response handling, show the response as a toast.
4. Test it: in TikTok, Share → HTTP Shortcuts → Save to Takeaway. They should see "Saved to Takeaway".

**iPhone (Shortcuts app):**
1. Create a new shortcut "Save to Takeaway". In its details, turn on **Show in Share Sheet** and accept URLs and text.
2. Add **Get Contents of URL**: URL `https://<their-url>/api/inbox`, method `POST`, header `Authorization` = `Bearer <token>`, request body JSON with a key `url` set to **Shortcut Input**.
3. Add **Show Notification** with the contents of the URL.
4. Test it: Share → Save to Takeaway.

## Step 8: Notifications (optional)

They install **ntfy** on their phone and subscribe to a long, random topic name (generate one for them, for example `takeaway-` plus 20 random characters), then set `NTFY_TOPIC` to the same name in `.env.local`. Anyone who knows the topic name can read the notifications, so keep it secret.

## Step 9: Start the worker and verify

```bash
npm run worker -- --once   # processes anything queued, then stops
```

Share one short video from the phone, run the command above and watch the log. "Ready in …s: <title> → <collection>" means it worked. Then open the dashboard and Review mode with them.

To keep it running, use `npm run worker`. To start it automatically:
- **Windows:** run `powershell -ExecutionPolicy Bypass -File worker\install-autostart.ps1` once. It creates a hidden Task Scheduler task that starts at login, restarts the worker if it stops and logs to `worker.log`. Before starting the task (`Start-ScheduledTask -TaskName "Takeaway worker"`), stop any worker already running in a terminal.
- **macOS/Linux:** a crontab line `@reboot cd /path/to/my-takeaway && npm run worker >> worker.log 2>&1`, or a launchd/systemd service.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "Claude Code isn't installed" | Install it, or set `CLAUDE_BIN` to its full path in `.env.local`. |
| A TikTok or YouTube link fails to download | Update yt-dlp (`yt-dlp -U`, or `winget upgrade yt-dlp.yt-dlp`). Platforms change often. |
| Instagram links fail | Instagram often blocks downloads without a login. This isn't supported yet. |
| Phone gets 401 | Wrong or old token. Create a new one in Settings and update the shortcut. |
| Items stay "queued" | The worker isn't running, or `.env.local` has the wrong email or password. |
| Notes feel too long or too technical | Edit `worker/prompt.md`, then retry an item from the dashboard. |

## Rules for you, the agent

- Never commit `.env.local`, and never put a secret key in a `NEXT_PUBLIC_` variable.
- Don't ask the person to paste passwords, tokens or keys into the chat. Have them put secrets straight into `.env.local`, the Vercel dashboard or their phone.
- Explain each step in one or two plain sentences before doing it, and wait for confirmation before anything that creates accounts, deploys or costs money.
- This tool downloads videos for personal study. Mention that it shouldn't be offered as a public service without checking each platform's terms.
