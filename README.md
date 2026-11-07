<h1 align="center">Cockpit</h1>

<p align="center">
  Run many Claude Code sessions at once on macOS, each in its own git worktree.
</p>

<p align="center">
  <a href="https://github.com/kaimunlau/cockpit-releases/releases/latest"><img alt="Latest release" src="https://img.shields.io/github/v/release/kaimunlau/cockpit-releases?label=download&color=7B83EB"></a>
  <img alt="macOS" src="https://img.shields.io/badge/platform-macOS-lightgrey">
</p>

![The board](docs/screenshots/board.png)

- **One board** for every session: working, blocked on you, or waiting.
- **What changed**: files, diff and the PR each session opened.
- **Your own `claude`**: plugins, skills, hooks, MCP servers and permission prompts work as in a terminal. Cockpit never writes to `~/.claude/`.

**Contents:** [Installation](#installation) · [Features](#features) · [Keyboard shortcuts](#keyboard-shortcuts) · [Updating](#updating) · [License](#license)

---

## Installation

### Requirements

| Requirement | Needed for |
|---|---|
| macOS (Apple Silicon or Intel) | Running Cockpit |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (`claude`), signed in | Every session |
| `git` | Worktrees, diffs, branch status |
| [GitHub CLI](https://cli.github.com) (`gh`), signed in with `gh auth login` | PR badges and the PR tab (optional) |

> [!NOTE]
> These must be on your login shell's `PATH`. Cockpit reads it from `$SHELL -l -i`.

### Download and install

1. Download the latest `Cockpit_<version>_universal.dmg` from [cockpit-releases](https://github.com/kaimunlau/cockpit-releases/releases/latest). One app for Apple Silicon and Intel.
2. Open the DMG and drag Cockpit to Applications.
3. Open Cockpit. macOS blocks it the first time: the app is ad-hoc signed, not signed with an Apple Developer ID.
4. Go to **System Settings → Privacy & Security → Security**, click **Open Anyway**, confirm, and open Cockpit again.

> [!TIP]
> You do this once per Mac. After an update, macOS may ask again for access to folders like Documents or Desktop.

### First steps

**1. Add a repo.** Click **Add repo**, pick a local git repo, then click **Add repo** in the dialog. Cockpit preselects the repo's default branch; keep it.

![Add repo on the empty board](docs/screenshots/first-add-repo.png)

**2. Start a session.** Click **New session** (1), write a **First prompt** (2), click **Create session** (3). Leave the other fields as they are.

![New session form](docs/screenshots/first-new-session.png)

**3. Open it.** Click the new card.

![The new session's card on the board](docs/screenshots/first-open-session.png)

---

## Features

### Board

![The board](docs/screenshots/board.png)

One row per repo, one column per state:

| Column | Meaning |
|---|---|
| **Working** | Claude is running. The card counts running subagents, shells and monitors. |
| **Blocked** | Claude needs you, usually a permission prompt. The card glows and you get one macOS notification; click it to open the session. |
| **Your turn** | Claude is done. Ended sessions show dimmed here. |

Cards show the branch, time in state, lines changed and the PR badge. The badge turns red on failed CI, requested changes or conflicts; click it to open the PR.

- Rows are in alphabetical order. To set your own order, set **Repo order** to **Manual** in Settings, then drag a repo row.
- **+** starts a session in that repo.
- **…** → **Default branch…** changes the repo's default branch. **…** → **Remove repo** removes a repo with no sessions.
- Search by title, PR link or PR number.

### Starting a session

![New session form](docs/screenshots/new-session.png)

- The defaults work: write a first prompt and create. A blank branch name and title are generated from the prompt.
- Type `/` for slash commands. Add files by dropping, pasting or **Add files…**.
- **Base branch** starts on the branch you last used in that repo, else on its default branch. The list has the local branches, then the branches only `origin` has.
- Each session gets its own worktree in `<repo parent>/.cockpit-worktrees/`, branched from a fresh `origin/<base>`.
- The chips in the prompt box set the session's permission mode (Auto by default), model and effort. The model starts on **Default model** from Settings. Effort **Default** (the sliders icon) leaves it to your Claude Code settings. Sessions never use `--dangerously-skip-permissions`.
- Claude can propose a session through Cockpit's `new_session` tool. It opens this form prefilled; nothing starts until you click **Create session**.

### Session view

![Session view](docs/screenshots/session.png)

| Area | What it shows |
|---|---|
| **Terminal** | The real `claude` process. |
| **Terminal / Chat** | The two icons switch the center between the terminal and the chat of the same process (see below). Each session remembers its choice. |
| **Header** | PR badge, branch, lines changed, uncommitted files, ahead/behind, context use. Click the title to rename. |
| **Sidebar** | Every session and what it is running. **Collapse sidebar** shrinks it to a rail. |
| **Side pane** | Files, Diff, PR, Notes, Workflow, and Artifacts (claude.ai pages the session linked, when there are any). The button at the top right hides it. |

Repos have notes too: click the pencil next to the repo name in the sidebar.

### Chat view

![Chat view](docs/screenshots/chat.png)

Click the chat icon in the session header (its tooltip reads **Chat (Beta)**) to read the session as a chat instead of a terminal. It is the same `claude` process: switching back and forth never restarts or touches it.

> [!NOTE]
> Chat is in beta, so every chat icon carries a **Beta** pill, and the toggle in Settings a **Beta** tag. The terminal is still the full view: anything the chat can't show, the terminal does.

- Your prompts sit on the right, with their images. Claude's replies are formatted text, with their images. An image file shows only if it is inside the session's worktree. An image on GitHub shows too; one from any other site stays a link. An image that can't load shows as its text, a link when it has a web address.
- Each tool call is one line with its command or file and a ✓ or ✕. Click it to see its input and result. Thinking is a collapsed **Thinking** line.
- Two or more tool calls in a row fold into one steps line, along with any thinking between them. Each tool call counts as a step. While Claude works, the line shows the step Claude is on, such as **◐ Bash cargo test · 3 steps**, and each new step replaces it. Once the run is over, it reads like **6 steps Bash ×5 · Edit ✓**. If any steps failed, it ends in ✕ and how many failed instead, also while Claude works. Click it to see every step.
- File paths are links: click one to open it in the Files tab, at its line. <kbd>⌘</kbd>-click a **Read**, **Write** or edit line (**Edit**, **MultiEdit**, **NotebookEdit**) to open its file.
- Every code block has a copy button in its top-right corner: Claude's code, a tool call's input and result, and command output. It shows while the pointer is over the block or the button has keyboard focus. Click it to copy the block's text. It reads **Copied** for 2 seconds, or **Copy failed** if the clipboard refuses.
- A subagent's work sits under the **Agent** line that started it. Click it to open the subagent's thread.
- Subagents, background shells and monitors that are still running are listed above the composer, each with how long it has run. The list stays there while you scroll, and each line goes away when its work ends.
- `!` commands and slash commands such as `/model` show with their output. `/compact` and interruptions are divider lines.
- New replies appear within about 2 seconds, a block at a time. The chat follows them while you are at the bottom, and stays put while you scroll up to read.
- Text that arrives while the chat is open moves in a little: a reply fades up line by line and your prompt fades in. Set **Chat text** to **Type** to have replies type themselves out instead, or to **None** for no motion. What the chat already held when you opened it shows at once, and nothing moves while macOS **Reduce motion** is on.
- After `/clear` the chat starts empty. After a resume it shows the resumed conversation.
- In a folder Claude Code hasn't opened before, it first asks in the terminal whether to trust the folder. If a session stays **Starting** for a few seconds, the chat says Claude Code may be waiting for an answer, with **Open in terminal**.
- When Claude needs an answer, it shows as a card above the composer (see **Answer cards**). Anything the chat can't show reads **Shown in the terminal**, with **Open in terminal**.

To open every new session in Chat, set **New sessions open in** to **Chat** in Settings.

### Answer cards

![Answer cards](docs/screenshots/answer-cards.png)

When a session is **Blocked**, its chat shows what Claude is waiting on as cards above the composer. Answer there, or in the terminal as before: whichever comes first counts.

| Claude asks | The card shows | You can |
|---|---|---|
| To use a tool (run a command, edit a file…) | The command or file path | **Allow**, or **Deny** with an optional message for Claude |
| A question (`AskUserQuestion`) | Every question with its options, and **Other…** for your own words | Pick the answers, then **Submit** |
| To approve a plan | The plan, with its images from the worktree or GitHub | **Send feedback** so Claude revises it, or **Approve in terminal** |
| Anything else (an MCP server's form, an error) | **Answer in terminal** | **Open in terminal** |

- A subagent's card names the subagent, with ⧉.
- After a click, the button spins until Claude moves on. Nothing is typed into the terminal.
- A Deny with no message tells Claude "Denied in Cockpit."
- If you answer in the terminal first, the card says **Answered in the terminal** and fades.
- A card left unanswered for about 10 minutes turns into **Answer in terminal**: Claude Code's own question is still open there.
- Cards never offer "allow for the session" or a mode change, and plans are approved only in the terminal. Use the terminal for those.

### Composer

![Composer](docs/screenshots/composer.png)

The box under the chat sends your next prompt to Claude, as if you had typed it in the terminal.

- Opening a session in Chat puts the cursor in the composer, so you can start typing right away.
- Press <kbd>↩</kbd> or click **Send**. <kbd>⇧</kbd> <kbd>↩</kbd> starts a new line. Type `/` to pick a slash command.
- Paste, drop or click **Add files…** to attach screenshots and files, as in the first prompt.
- **Send** sends at once when Claude is ready for a prompt: on your turn, and also when Claude has finished but its background shells or subagents still run (a CI watcher, say), unless a macro is waiting on them to type its next command.
- While Claude works or waits on a question, <kbd>↩</kbd> queues your message instead, as the line above the box says (its ◐ turns while Claude works). The button reads **Queue**, and the message shows above the box as **Queued**. When Claude's turn ends, it goes in by itself. <kbd>↩</kbd> again adds the new text to it as a new paragraph, and the button reads **Add to queue**. Files you add go with it. One message is queued per session, and it lasts until Cockpit quits.
- While the session shows **Starting…** or **Session ended**, a line says why you can't send. You can keep typing; the text waits for you.
- On the queued message, **Edit** puts it back in the box, **Remove** drops it, and **Send now** types it into Claude Code at once: Claude takes it at its next step, in the same turn, and it shows in the chat from there. **Send now** is unavailable while Claude waits on a question, while a macro runs, while the session starts or while text you typed in the terminal waits in Claude's input. Hover it to see why.
- A queued message is held, not sent, when you interrupted Claude (<kbd>esc</kbd>), when you typed text in the terminal during the turn (**Send anyway** sends it), when the session ended (resume it first) or when sending failed. Keys that type nothing, such as <kbd>⇧</kbd> <kbd>⇥</kbd>, don't hold it. The bubble then says why (**Held · you interrupted Claude, so this wasn't sent**, say) and has a **Send** button.
- While a message is held and Claude is ready, <kbd>↩</kbd> or **Send** under the box adds your new text to it and sends both.
- Each session keeps its own draft until you send it, even when you switch views or sessions.
- Chips at the bottom left show the permission mode, model and effort, and change them (see [Run settings](#run-settings)).
- If Claude Code's input in the terminal holds something, Send waits: text you typed there, or a picker you opened. Keys that leave the input empty don't hold it, such as <kbd>esc</kbd>, <kbd>⇧</kbd> <kbd>⇥</kbd>, or text you typed and erased. The run chips are locked too. A live view of the terminal's last lines shows what you left there. Click **Back to terminal** to finish it there, or **Send anyway** to send the composer's text.
- A picker opened by a command from the composer, such as `/model`, holds Send the same way until you close it in the terminal. A command that leaves the input empty, such as `/cost`, doesn't.

![Composer with a queued message](docs/screenshots/composer-queued.png)

### Run settings

![Run settings](docs/screenshots/run-chips.png)

The chips at the bottom left of the composer show what the terminal's status line shows: the permission mode (**Plan**, **Auto**, **Accept edits** or **Manual**, each in its own colour), the model, the effort level and **thinking off** when thinking is off.

- Click the mode, model or effort chip and pick a value. Cockpit restarts Claude with it, in the same conversation. **Restarting Claude…** shows until it is your turn again.
- The chips change only on your turn, or once Claude has finished and only background work still runs. Otherwise they are locked, and hovering one says why.
- If background shells, subagents or monitors still run, the menu asks first and lists them. **Restart** stops them and restarts Claude; **Cancel** keeps them running.
- A <kbd>⇧</kbd> <kbd>⇥</kbd> in the terminal changes the mode too. The mode chip catches up the next time Claude acts: on your turn, that is once you send your next prompt.
- If Claude can't start again, the session ends and the composer says why. **Resume** starts it with the values you picked.

> [!NOTE]
> In Plan mode, Claude Code currently runs Sonnet when Haiku is picked, so the model chip shows Sonnet. The model menu still ticks Haiku and says "Plan mode runs Sonnet" under it, in the composer and in the New session form. In other modes Haiku runs without an effort level, and the effort menu says so.

### Reply to a selection

![Reply to a selection](docs/screenshots/chat-reply.png)

Answer one part of a reply without retyping it.

- Select any text in the chat and click **Reply**. Type your reply and press <kbd>↩</kbd> or click **Add**. <kbd>⇧</kbd> <kbd>↩</kbd> starts a new line, and <kbd>esc</kbd> or a click elsewhere cancels.
- Each reply waits above the composer as a chip: the text you selected, then your reply. Hover a chip to read it in full, or click **×** to remove it.
- Add as many as you like, then write the rest of your prompt and press **Send**. Claude gets each selection quoted, with your reply under it, before the text you typed. A slash command you typed goes first instead, so Claude Code still runs it, with the replies after it.
- While the reply box is open, the chat stops following new messages, so the text you selected stays put.
- Replies stay in the session's draft until you send it, like the rest of the draft.

### Files

![Files tab](docs/screenshots/files.png)

- Read-only file tree. `M` marks changed files, `A` added ones.
- Syntax highlighting and line numbers. Markdown is rendered, with its images from the worktree or GitHub. Images from other sites stay links. An image that can't load shows as its text, a link when it has a web address. **Source** shows the raw text.
- <kbd>⌘</kbd>-click a file path in the terminal, like `src/App.tsx:120`, or click one in the chat, to open it here. The tree opens to the file, and the viewer scrolls to that line and highlights it.
- **Pop out** shows the file over the terminal.

### Default branch

![A repo's default branch](docs/screenshots/repo-main.png)

- Click a repo's name in the sidebar to browse its default branch, no session needed.
- The default branch is set per repo. **Add repo** preselects the one GitHub reports, else the one `origin/HEAD` points at, else `main` or `master`. Change it with **…** → **Default branch…** on the board.
- Cockpit reads `origin/<default>`, else the local `<default>`. The header shows it and its commit, e.g. `origin/main @ 8e2f9a1`.
- It shows what is committed, not the repo folder's uncommitted edits, and leaves submodules out. It never fetches: `origin/<default>` is as fresh as your last `git fetch`. If neither branch exists, the view says so and points to **Default branch…**.
- **Reload** reads the branch again. Same viewer as the Files tab: highlighted code, rendered Markdown with **Source**. Images there stay links.

### Diff

![Diff tab](docs/screenshots/diff.png)

- Everything since the merge-base with `origin/<base>`: commits plus uncommitted changes.
- While the branch has an open PR, `<base>` is the branch the PR merges into (fetched from origin), so the header's line counts and this tab match GitHub. Once the branch has no PR, it goes back to the base the session started from.
- Filter by **Committed**, **Uncommitted** or a single commit.
- **Unified** or **Side by side**. **Pop out** shows the diff over the terminal.

### Pull request

![PR tab](docs/screenshots/pr.png)

- Description, checks, review, unresolved threads and a GitHub link, read through your `gh` login.
- The header shows where the PR goes, in GitHub's order: `main ← fix/login`, the branch it merges into, then its own branch. The confirm button names it too: **Confirm squash into main**.
- The description shows its images: pasted screenshots, images from the repo and badges, private repos included. They load through `gh` too. An image that can't load shows as its text, a link when it has a web address.
- Merge or squash-merge after a confirm. The menu offers only the methods the repo and the rules on the PR's base branch allow. Cockpit remembers your method per repo and base branch.
- When no method is allowed (a base that takes only rebase merges, say), the tab says why. Merge on GitHub.
- **Update branch** when the branch is behind. When the PR's own branch is protected on GitHub (a long-lived branch such as `dev`), Cockpit leaves it to GitHub: no **Update branch**, and GitHub keeps the branch after the merge even when the repo deletes merged branches.
- A draft PR gets **Ready for review** in place of the merge button. One click marks it ready on GitHub.
- Refreshes every ~30 s, or on demand.
- A session's PR is the one whose branch matches the session's. When the names differ, Claude can link an existing PR of the repo through Cockpit's `link_pr` tool. The badge tooltip then says "linked by the agent", and the PR tab's header shows the PR's own branch. Claude can remove the link the same way.

### Macros

![Macros page](docs/screenshots/macros.png)

A macro is a short chain of commands, e.g. `/clear` then `/code-review pr {pr}`.

- Edit them on the **Macros** page.
- Run one from the buttons above the side pane's tabs. Each command is sent when Claude's turn ends. **Cancel** stops the chain.
- **Or…** adds fallback commands. Cockpit sends the first one installed.
- **Show at session create** offers the macro in the new-session form.

Placeholders fill themselves:

| Placeholder | Value |
|---|---|
| `{pr}` | PR number. A macro that uses it is disabled until the branch has a PR. |
| `{pr_url}` | PR link |
| `{branch}` | The session's branch |
| `{base}` | The base branch: the PR's while the branch has an open one |

### Workflows

![Workflows page](docs/screenshots/workflows.png)

A workflow is a list of prompts and macros you step through, e.g. build → review → fix → release notes.

- Edit them on the **Workflows** page. `{name}` becomes a variable you fill in; the four placeholders above fill themselves.
- Pick one in the new-session form. Step 1 becomes the first prompt.

![Workflow tab](docs/screenshots/workflow.png)

- The **Workflow** tab shows the steps, the next one and the variables.
- **Run** sends the next step on your turn. **Skip** passes it.
- The header badge shows progress.

### Cockpit view

![Cockpit view](docs/screenshots/cockpit.png)

Every running session in one grid, as its terminal or its chat. Answer prompts without switching sessions.

- Each tile shows the session's own view. The terminal and chat icons in its header switch it, and the session view follows.
- A chat tile has what the session's chat has: running work, answer cards and the composer.
- **All tiles**, above the grid, switches every tile at once. It shows a view only while every tile is on it.
- Blocked tiles glow. A pill points to blocked tiles scrolled out of view: click it to jump there, ready to answer.
- ⤢ opens the session.

### Resume and archive

![Archive dialog](docs/screenshots/archive.png)

| | |
|---|---|
| **Resume** | An Ended session keeps its card. Opening it runs `claude --resume` (unless turned off in Settings). The header has a **Resume** button too. |
| **Archive** | Stops Claude, deletes the worktree and the branches Cockpit created. Keeps branches that already existed, the repo's default branch, `main`, `master` and the PR's base branch. Never touches the remote. Warns first about uncommitted changes and unpushed commits. |

### Usage meters

![Usage meters](docs/screenshots/usage.png)

The sidebar footer shows your plan's 5-hour and weekly limits: % used and reset time. The bar changes color near a limit. Numbers appear once a session has run. When you sign in to Claude Code, or sign in as another account, they update within seconds.

### Settings

![Settings](docs/screenshots/settings.png)

| Setting | Default |
|---|---|
| Jump into session on create | Off |
| Warn about uncommitted changes on archive | On |
| Resume when opening an Ended session | On |
| Keep Mac awake while sessions work (plugged in) | Off |
| Theme | Slate (Ledger is a darker alternative) |
| Default model | Opus |
| Repo order | Alphabetical (**Manual** keeps the order you drag repo rows into on the Board) |
| New sessions open in | Terminal (**Chat** opens new sessions in the chat view) |
| Chat text | Fade (how new chat text appears: **Fade** line by line, **Type** letter by letter, or **None**) |
| Beta builds | Off (installed app only, see [Beta builds](#beta-builds)) |

The installed app also shows its version and **Check for updates**. The last row, **License**, shows Cockpit's license and the licenses of the open-source software it includes. **Cockpit → About Cockpit** shows the version and the copyright.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| <kbd>⌘</kbd> <kbd>B</kbd> | Back to the board |
| <kbd>⌘</kbd> <kbd>+</kbd> / <kbd>⌘</kbd> <kbd>−</kbd> / <kbd>⌘</kbd> <kbd>0</kbd> | Terminal and chat text bigger / smaller / reset |
| <kbd>⌘</kbd> <kbd>↩</kbd> | Run the next workflow step |
| <kbd>⌥</kbd> <kbd>⌘</kbd> <kbd>↩</kbd> | Skip the next workflow step |
| <kbd>⇧</kbd> <kbd>↩</kbd> | New line in the terminal or the composer, without sending |
| <kbd>↩</kbd> | Send the composer's prompt |
| <kbd>⌘</kbd>-click a file path in the terminal | Open the file in the Files tab, at its line |

---

## Updating

![Update ready](docs/screenshots/update.png)

- Cockpit checks for a new version at launch and every 6 hours.
- It downloads and installs a new version in the background while you work. Your sessions keep running.
- When the new version is ready, **Cockpit vX.Y.Z is ready** appears. The update applies the next time you launch Cockpit.
- Click **Restart now** to switch at once, or **Later** to keep working. Until you restart, the sidebar shows **Restart to update** (in the collapsed rail, an ↑ button).
- If sessions are running, Cockpit asks before it restarts, because restarting ends them. Resume them afterwards from the board.
- If a download or install fails, Cockpit tries again later, less and less often: after 6 hours, then 12, 24 and 48 hours, then every 4 days.

Settings shows the installed version and has **Check for updates** to check right away.

> [!NOTE]
> If your Mac would ask for an administrator password to replace Cockpit (it sits in a folder you can't write to), Cockpit never installs on its own. The sidebar shows **Update to vX.Y.Z** and Settings shows **Install update** instead. Click either one and enter your password when macOS asks.

> [!NOTE]
> The app holds no GitHub token. It reads the public feed at `https://github.com/kaimunlau/cockpit-releases/releases/latest/download/latest.json` and verifies each update's signature before installing it.

### Beta builds

![Beta builds](docs/screenshots/beta-builds.png)

- Turn on **Beta builds** in Settings to get changes before they're released.
- Cockpit checks for a beta at once, then on the usual schedule. Betas install the same way as releases, and their version ends in `-beta.N`.
- A release newer than your beta still installs.
- Turn **Beta builds** off to go back to the latest release. Cockpit checks at once and installs it, even though it is older than your beta.
- Going back keeps your repos, sessions, workflows and macros. A value only the beta knows (a new setting option, say) goes back to its default. Cockpit first keeps a copy of the beta's data, as `state.json.beta-<time>` in `~/Library/Application Support/com.cc-cockpit.app/`.

> [!WARNING]
> Beta builds may be unstable.

---

## License

Cockpit is free to use, for any purpose, under the [MIT License with the Commons Clause](LICENSE). You may not sell it. Copies must keep the copyright notice.
