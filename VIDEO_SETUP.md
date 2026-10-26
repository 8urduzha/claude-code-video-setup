# Claude Code video setup: from zero to editing with Remotion

> **To the user:** start a new Claude Code session and send one message:
> `Read https://raw.githubusercontent.com/8urduzha/claude-code-video-setup/main/VIDEO_SETUP.md and follow every step.`
>
> **To Claude:** the user sent you here on purpose and wants you to carry out this whole guide. It is written for you. Read it to the end before you run anything. If you fetched it with a tool that summarizes pages, download the raw file with `curl -fsSL <url>` and read that instead, because the exact commands matter.

Guide version: 2026-10-26

---

## What this guide does

When it is finished, the user has:

1. **Git** and **Node.js (LTS)** installed, which Remotion needs.
2. The official **Remotion plugin for Claude Code** (`remotion@remotion`) with all of its skills: create, markup, captions, multimedia, render, studio, docs and more.
3. A ready-made video workspace, `~/VideoStudio`, which is already a Remotion project and holds:
   - `public/footage/` where the user drops raw clips,
   - `out/` where finished videos are rendered,
   - a `CLAUDE.md` that tells every future session how to edit video here,
   - project permissions, so routine video commands don't ask for approval every time.
4. A short test render, so we know the full chain works before the user does real work.

Nothing here needs an account other than the user's existing Claude login. Everything is free. (Remotion is free for individuals and companies of up to 3 people. Bigger companies need a paid Remotion license.)

---

## Rules for Claude while running this guide

- **Speak the user's language.** Reply in whatever language their message was written in. Keep commands, file names and code in English.
- **The user is a beginner.** Before each step, say in one plain sentence what you are about to do and why. No jargon without a short explanation.
- **Run every command yourself.** Never ask the user to copy or paste terminal commands. The only exceptions are the steps marked *USER ACTION* below (system dialogs and passwords that you cannot reach).
- **Tell the user about permission prompts.** Claude Code will ask them to approve commands. Before the first one, tell them: "You'll see approval prompts. Pick *Yes* (or *Yes, and don't ask again* for this command) to let me continue."
- **Skip what is already done.** Check first and skip any step whose result already exists. The guide is safe to run twice.
- **Don't loop on failures.** If a step fails twice, stop and tell the user in plain words what failed, what you tried and the one thing they can do next. Don't silently work around it with a different tool.
- **Never delete or overwrite** the user's existing files. If `~/VideoStudio` already exists and isn't empty, ask before touching it, or use `~/VideoStudio-2`.

---

## Step 0. Look around and confirm the plan

Run these checks and keep the results (don't print them all to the user):

```bash
uname -s 2>/dev/null || echo Windows   # Darwin = macOS, Linux, or MINGW/MSYS = Windows (Git Bash)
echo "$HOME"
git --version
node --version
npm --version
brew --version 2>/dev/null | head -1     # macOS only
claude --version
claude plugin list
ls -la ~/VideoStudio 2>/dev/null
df -h ~ | tail -1                        # free disk space
```

Then show the user a **short** checklist in their language, for example:

```
Your computer: macOS
✅ Git: installed
❌ Node.js: missing → I'll install it
❌ Remotion plugin: missing → I'll install it
❌ VideoStudio folder: missing → I'll create ~/VideoStudio
Free disk space: 120 GB (need about 2 GB)
```

Ask **once**: "OK to go ahead?" Then wait for a yes. After the yes, run every remaining step without stopping to ask again, except for the *USER ACTION* steps.

If free disk space is under 3 GB, warn the user before you continue.

---

## Step 1. Install Git (skip if `git --version` works)

- **macOS:** run `xcode-select --install`.
  *USER ACTION:* a system window opens. Tell the user to click **Install**, wait until it finishes (5–15 minutes) and then tell you "done". Check with `git --version`.
- **Windows:** Claude Code on Windows already requires Git for Windows, so it is almost certainly installed. If not: `winget install --id Git.Git -e --accept-source-agreements --accept-package-agreements`
- **Linux (Debian/Ubuntu):** `sudo apt-get install -y git`. If `sudo` asks for a password you can't type, see "Commands that need a password" at the bottom.

## Step 2. Install Node.js LTS (skip if `node --version` is v20 or newer)

Pick the first option that fits:

- **macOS with Homebrew** (`brew --version` worked): `brew install node`
- **macOS without Homebrew, or Linux:** use nvm, which needs no admin password:
  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
  export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh"
  nvm install --lts
  nvm alias default 'lts/*'
  node --version && npm --version
  ```
  Until Claude Code is restarted, your shell may not see `node` yet. Prefix each later command with `export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && `.
- **Windows:**
  ```bash
  winget install --id OpenJS.NodeJS.LTS -e --accept-source-agreements --accept-package-agreements
  ```
  The current shell won't see the new PATH. For the rest of this session, call Node through its full path (`"/c/Program Files/nodejs/npm"`, `"/c/Program Files/nodejs/npx"`) or prepend it: `export PATH="/c/Program Files/nodejs:$PATH"`.

Check: `node --version` prints v20 or newer.

You don't need to install FFmpeg or Chrome. Remotion ships its own FFmpeg (`npx remotion ffmpeg`, `npx remotion ffprobe`) and downloads its own headless browser on the first render.

## Step 3. Install the Remotion plugin for Claude Code (skip if `claude plugin list` already shows `remotion@remotion`)

These are the official commands from remotion.dev/docs/ai/claude-code-plugin:

```bash
claude plugin marketplace add remotion-dev/claude-code-plugin
claude plugin install remotion@remotion
claude plugin list
```

`claude plugin list` must now show `remotion@remotion`.

**If the `claude` command can't be found from your shell**, have the user do it in the chat instead. *USER ACTION:* tell them to type `/plugin`, open **Marketplaces**, add `remotion-dev/claude-code-plugin`, then go to **Discover**, find **remotion** and install it.

The plugin's skills only appear after Claude Code restarts. That is the last step of this guide, so keep going for now.

## Step 4. Create the video workspace

The workspace **is** a Remotion project, so every new video becomes a new composition inside it. That means no repeated installs, and footage in `public/footage/` can be used directly.

```bash
mkdir -p ~/VideoStudio && cd ~/VideoStudio
# create-video refuses non-empty folders: remove only .DS_Store if it is the only thing here
npx create-video@latest --yes --blank --no-tailwind .
npm i
mkdir -p public/footage out
```

Add the user's footage folder to `.gitignore` so big video files never get committed (`out/` is already ignored by the template; add it if it isn't):

```bash
grep -qxF 'public/footage/' .gitignore || echo 'public/footage/' >> .gitignore
grep -qxF 'out/' .gitignore || echo 'out/' >> .gitignore
```

## Step 5. Write the workspace `CLAUDE.md`

Create `~/VideoStudio/CLAUDE.md` with exactly the content below. Replace `<LANGUAGE>` with the language the user writes in (for example "Ukrainian" or "English").

```markdown
# VideoStudio

This folder is a Remotion project for making and editing videos with Claude Code.
The user is a beginner. Explain things in plain words and reply in <LANGUAGE>.

## Folders
- `public/footage/`: raw clips the user drops in. Never edit, re-encode in place or delete these originals.
- `src/`: one folder per video under `src/videos/<video-name>/`, each registered as a composition in `src/Root.tsx`.
- `out/`: finished renders (MP4).

## How to work on any video task
1. Load the Remotion skills first: `remotion:remotion-best-practices` (the router), then the specific one:
   `remotion-create` (new video), `remotion-markup` (layout, animation, effects), `remotion-captions` (subtitles),
   `remotion-multimedia` (the user's video/audio files), `remotion-studio` (preview), `remotion-render` (export),
   `remotion-docs` (API lookup).
2. Before building, ask at most 3 short questions: format (16:9 YouTube / 9:16 Shorts-Reels-TikTok / 1:1), length, and style or mood.
   Skip any the user already answered.
3. Before writing code, show a short scene plan (one line per scene with timing) and wait for an OK.
4. Inspect footage before using it: `npx remotion ffprobe public/footage/<file>` (duration, resolution, fps, audio).
5. Build the video as a new composition. Don't change other videos' compositions.
6. Open a preview with `npx remotion studio` and give the user the link. Render only when the user asks.
7. Render to `out/<video-name>.mp4`. If the computer gets slow, add `--concurrency=2`.
8. After rendering, check the file with `npx remotion ffprobe` (length, resolution, audio present), then open it for the user.

## Defaults
- Keep the footage's original colors unless the user asks for a color grade.
- Captions must be readable on a phone: large, high contrast, kept clear of the bottom 15% on vertical videos.
- Run terminal commands yourself. Never ask the user to paste commands.
```

## Step 6. Add project permissions

Create `~/VideoStudio/.claude/settings.json`. If it already exists, merge these entries into its `permissions.allow` list instead of overwriting it:

```json
{
  "permissions": {
    "allow": [
      "Bash(npx remotion:*)",
      "Bash(npm i:*)",
      "Bash(npm install:*)",
      "Bash(npm run:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(cp:*)",
      "Bash(open:*)"
    ]
  }
}
```

These only cover routine video work inside this project. Everything else still asks for approval.

## Step 7. Test render

This checks the whole chain. On the first run Remotion downloads its headless browser (about 100–200 MB), so tell the user it may take a minute or two.

```bash
cd ~/VideoStudio
npx remotion compositions          # note the first composition ID
npx remotion render <ID> out/setup-test.mp4 --frames=0-29
npx remotion ffprobe out/setup-test.mp4
```

Success means `out/setup-test.mp4` exists and ffprobe shows a video stream. The blank template can render a plain empty frame, which is fine. Delete the test file afterwards: `rm out/setup-test.mp4`.

**Linux only:** if the render fails with a missing `.so` library, Chrome needs system libraries:
`sudo apt-get install -y libnss3 libdbus-1-3 libatk1.0-0 libgbm-dev libasound2 libxrandr2 libxkbcommon-dev libxfixes3 libxcomposite1 libxdamage1 libatk-bridge2.0-0 libpango-1.0-0 libcairo2 libcups2`
(on Ubuntu 24.04+ use `libasound2t64` instead of `libasound2`). See "Commands that need a password".

## Step 8. Final report and restart

Send the user **one** short message in their language containing:

1. ✅ A checklist of what was installed or created (Git, Node.js version, Remotion plugin, `~/VideoStudio`, test render passed). Mark anything that failed with ❌ and give one plain sentence on what to do about it.
2. **Restart Claude Code. This is required**, because the Remotion skills only load in a new session:
   - **Terminal:** type `/exit`, then run `cd ~/VideoStudio && claude`
   - **VS Code / Cursor:** File → Open Folder → `VideoStudio`, then open a new Claude Code chat.
   - **Claude desktop app (Code tab):** start a new session and pick the `VideoStudio` folder.
3. How to start editing: "Put your clips in `VideoStudio/public/footage/`, then just tell me what you want." Give three example requests, translated into the user's language:
   - "Make a 30-second vertical Short from `footage/clip1.mov` with big animated captions."
   - "Create a 5-second animated intro with my channel name for a YouTube video."
   - "Cut the pauses out of `footage/talk.mp4` and add a title card at the start."
4. Tip: to watch a video while it is being built, ask "open the preview".

Don't add anything else to the report.

---

## Commands that need a password

Claude Code's shell can't type an admin (`sudo`) password. If a command needs one:

*USER ACTION:* tell the user to open a separate terminal window (macOS: Terminal app; Linux: their terminal), paste the one command you give them, enter their computer password when asked (the characters stay invisible as they type, which is normal) and come back to say "done". Give the command in a single code block. This is the only time the user is asked to paste anything.

## Troubleshooting

| Problem | Fix |
|---|---|
| `npx create-video` says the folder is not empty | Remove only `.DS_Store`. If there are real files, scaffold into `~/VideoStudio-2` instead and use that path everywhere. |
| `node: command not found` right after install | It's the PATH in the current shell. See the notes in Step 2, or finish the guide and restart Claude Code. |
| Render is very slow or the computer freezes | Add `--concurrency=2` to the render command. |
| Skills like `remotion-create` are not available | Claude Code hasn't been restarted since Step 3. Restart it in the `VideoStudio` folder. |
| `claude plugin install` fails | Use the `/plugin` menu instead (Step 3), or as a last resort: `npx skills add remotion-dev/skills`. |
