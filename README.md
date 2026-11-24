# Takeaway Builder

**Get your own "save videos to learn later" tool by asking your AI agent for it.**

[Takeaway](https://github.com/mb1720827-ui/takeaway) turns the TikToks, YouTube videos and articles you save into notes you can actually review. You tap Share on your phone, and an AI on your computer reads the captions, looks at the frames (including on-screen code) and writes a breakdown with key points, steps, concepts and things to try, sorted into collections, with a review queue.

This repository teaches your AI coding agent to build and deploy **your own copy**, customised to what you save.

## Use it with one prompt

### Claude Code

```
/plugin marketplace add mb1720827-ui/takeaway-builder
/plugin install takeaway-builder@takeaway-builder
```

Then say:

> **Build me my own Takeaway.**

### Any other agent (Codex, Cursor, Gemini CLI…)

> **Read https://github.com/mb1720827-ui/takeaway-builder/blob/main/AGENTS.md and build me my own Takeaway.**

## What the agent does

1. Asks three questions: Android or iPhone, what you save, and which Supabase project to use.
2. Checks and installs the tools (Node.js, Claude Code, yt-dlp, ffmpeg).
3. Clones Takeaway and customises your collections and the note style.
4. Sets up the database (Supabase) and deploys the app (Vercel).
5. Walks you through creating your account and inbox token.
6. Adds a **Save to Takeaway** button to your phone's share menu (HTTP Shortcuts on Android, Shortcuts on iPhone).
7. Optionally sets up phone notifications (ntfy).
8. Starts the worker and tests it end to end with one of your videos.

Secrets stay with you: the guide tells the agent never to ask for passwords or keys in chat, and never to use admin keys.

## What you need

- A computer that's on when you want saves processed (it runs the AI worker)
- Claude Code with a Claude plan. Processing uses your plan, so there's no separate API bill.
- Free accounts on GitHub, Supabase and Vercel

## Note

Takeaway downloads videos for **personal study** and deletes them after processing. Don't run it as a public service without checking each platform's terms.

## License

MIT
