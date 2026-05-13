---
title: "Building a Claude Code Desktop Notifier with Claude Code"
date: 2026-03-23 09:36:00
tags: [claude-code, cli, macos, ai]
---

## My setup: tmux + multiple Claude agents

I live in the terminal. My daily setup is tmux + vim — multiple panes, multiple sessions. Claude Code fits naturally into this workflow because I can spin up several Claude agents in separate tmux panes, each working on a different task or a different branch. One agent is refactoring a module, another is writing tests, a third is investigating a bug. They all work concurrently and I switch between panes to check on progress.

This is where things get interesting — and where the problem starts.

## The problem: silent agents waiting for attention

Claude Code agents don't always run to completion on their own. Sometimes an agent hits a permission prompt — it wants to run a command, edit a file, or access a tool that requires approval. Other times it asks a clarification question and waits for your response. In both cases, the agent just sits there, doing nothing, until you interact with it.

With a single agent, this is fine. You're watching the output. But with multiple agents running in separate tmux panes, you don't know an agent is blocked until you manually switch to that pane and look. By that point, it might have been sitting idle for minutes. Multiply that across several agents and you're wasting a lot of time.

I needed a way to get notified when any agent needs my attention.

## Iteration 1: "Can I get notified?"

I described the problem to Claude and asked for a solution. Claude introduced me to **hooks** — a feature in Claude Code's `settings.json` that lets you run shell commands in response to specific events. There are matchers for events like `permission_prompt` (agent needs approval) and a `Stop` hook (agent finished its task).

Claude wired up an `osascript` one-liner to send a native macOS notification:

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Needs your permission\" with title \"Claude Code\" sound name \"Ping\"'",
            "timeout": 5
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Task finished\" with title \"Claude Code\" sound name \"Glass\"'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

It worked. I got a macOS notification banner with a sound every time an agent needed permission or finished a task.

{% asset_img Iter1.png Basic notification with no context %}

But with multiple agents running, the notification just said "Needs your permission" — I had no idea _which_ project or task needed me.

## Iteration 2: "Which task needs me?"

I told Claude the problem: with multiple agents, a generic notification isn't enough. I need to know which project and branch needs attention so I can switch to the right tmux pane.

Claude updated the approach — instead of an inline `osascript` one-liner, it moved the logic to a shell script that reads the hook's stdin JSON. Claude Code passes context to hooks via stdin, including the working directory. The script extracts the project name and git branch:

```bash
PROJECT_DIR=$(/usr/bin/python3 -c \
  "import sys,json; print(json.load(sys.stdin).get('cwd',''))" 2>/dev/null)
PROJECT=$(basename "$PROJECT_DIR")
BRANCH=$(git -C "$PROJECT_DIR" rev-parse --abbrev-ref HEAD 2>/dev/null || echo "")

SUBTITLE="${PROJECT}"
[ -n "$BRANCH" ] && SUBTITLE="${PROJECT} · ${BRANCH}"
```

Now the notification showed something like **zenn-dev · chenshu/cc-notifier** in the subtitle. Much better — I could immediately tell which agent needed me and switch to the correct pane.

{% asset_img Iter2.png Notification with project and branch context %}

## Iteration 3: "Take me back to the terminal"

The next annoyance: clicking the macOS notification toast opened Script Editor instead of my terminal — because `osascript` owns the notification.

{% asset_img scriptEditor.png Clicking the notification opens Script Editor instead of the terminal %}

I wanted clicking the notification to bring my terminal app to the foreground, so I could see the agent that needed attention right away.

I asked Claude to fix this. Claude researched the problem and suggested [`terminal-notifier`](https://github.com/julienXX/terminal-notifier), a CLI tool that supports click-to-activate behavior via macOS bundle IDs. Unlike `osascript` notifications, `terminal-notifier` can bring a specific app to the foreground when you click the notification:

```bash
terminal-notifier \
  -title "Claude Code" \
  -subtitle "$SUBTITLE" \
  -message "$MESSAGE" \
  -sound "$SOUND" \
  -activate "$BUNDLE_ID"
```

Install it with `brew install terminal-notifier`, map your terminal app to its bundle ID (e.g., `com.mitchellh.ghostty` for Ghostty, `com.googlecode.iterm2` for iTerm2), and clicking the notification takes you straight back to your terminal.

{% asset_img iter3.png terminal-notifier notification with click-to-activate %}

## Iteration 4: "It broke — I'm in tmux"

After switching to `terminal-notifier`, something didn't work right. The click-to-activate wasn't activating the correct app.

Claude figured out the issue: inside tmux, the `$TERM_PROGRAM` environment variable reports `tmux` instead of the actual terminal app (e.g., Ghostty, iTerm2). So the bundle ID lookup was failing.

Claude added tmux detection logic — first checking `$TERM_PROGRAM_OUTER` (which some terminals set), and if that's not available, walking the process tree from the tmux client PID up to find the real terminal:

```bash
TERM="${TERM_PROGRAM}"
if [ "$TERM" = "tmux" ] || [ -n "$TMUX" ]; then
  if [ -n "$TERM_PROGRAM_OUTER" ]; then
    TERM="$TERM_PROGRAM_OUTER"
  else
    TMUX_CLIENT_PID=$(tmux display-message -p '#{client_pid}' 2>/dev/null)
    if [ -n "$TMUX_CLIENT_PID" ]; then
      PARENT_PID="$TMUX_CLIENT_PID"
      while [ "$PARENT_PID" -gt 1 ] 2>/dev/null; do
        PARENT_NAME=$(ps -p "$PARENT_PID" -o comm= 2>/dev/null | xargs basename 2>/dev/null)
        case "$PARENT_NAME" in
          ghostty|iTerm2|Terminal|WezTerm|alacritty|kitty)
            TERM="$PARENT_NAME"
            break
            ;;
        esac
        PARENT_PID=$(ps -p "$PARENT_PID" -o ppid= 2>/dev/null | tr -d ' ')
      done
    fi
  fi
fi
```

This walks up the process tree until it finds a known terminal app. Problem solved.

## Iteration 5: "Take me to the exact pane"

The `-activate` flag brought the terminal to the foreground, but with multiple tmux panes open I still had to visually scan for the right one. Not ideal when you have six panes across three windows.

I asked Claude to make the notification click take me directly to the pane where the agent was waiting. Claude replaced `-activate` with `-execute`, which runs an arbitrary command when you click the notification. The command activates the terminal app via AppleScript and then uses `tmux select-window` + `tmux select-pane` to jump to the exact pane:

```bash
if [ -n "$TMUX" ] && [ -n "$TMUX_PANE" ]; then
  TMUX_SOCKET=$(echo "$TMUX" | cut -d, -f1)
  TMUX_BIN=$(which tmux)
  TMUX_SESSION=$(tmux display-message -t "$TMUX_PANE" -p '#{session_name}' 2>/dev/null)
  TMUX_WINDOW=$(tmux display-message -t "$TMUX_PANE" -p '#{window_index}' 2>/dev/null)
  # Map bundle ID to app name for AppleScript activation
  case "$BUNDLE_ID" in
    com.mitchellh.ghostty)    APP_NAME="Ghostty" ;;
    com.googlecode.iterm2)    APP_NAME="iTerm" ;;
    # ... other terminals
  esac
  ACTIVATE_CMD=""
  [ -n "$APP_NAME" ] && ACTIVATE_CMD="osascript -e 'tell application \"${APP_NAME}\" to activate' && "
  terminal-notifier \
    -title "Claude Code" \
    -subtitle "$SUBTITLE" \
    -message "$MESSAGE" \
    -sound "$SOUND" \
    -execute "${ACTIVATE_CMD}${TMUX_BIN} -S '${TMUX_SOCKET}' select-window -t '${TMUX_SESSION}:${TMUX_WINDOW}' && ${TMUX_BIN} -S '${TMUX_SOCKET}' select-pane -t '${TMUX_PANE}'"
fi
```

The key insight: the script captures `$TMUX` and `$TMUX_PANE` at the time the hook fires — these environment variables identify exactly which tmux session, window, and pane the agent lives in. By embedding them into the `-execute` command, the click always targets the correct pane, even if you've switched to a different window or session in the meantime.

For non-tmux setups, the script falls back to the simpler `-activate` behavior from Iteration 4.

## The final result

Here's the complete setup — the hooks configuration in `settings.json` and the full `notify.sh` script.

**`~/.claude/settings.json`** (hooks section):

```json
{
  "hooks": {
    "Notification": [
      {
        "matcher": "permission_prompt",
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/notify.sh 'Needs your permission' 'Ping'",
            "timeout": 5
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "bash ~/.claude/notify.sh 'Task finished' 'Glass'",
            "timeout": 5
          }
        ]
      }
    ]
  }
}
```

<details>
<summary><code>~/.claude/notify.sh</code></summary>

```bash
#!/bin/bash
# Usage: notify.sh <message> <sound>
# Reads hook JSON from stdin to extract project name
# Clicking the notification brings the terminal app to the foreground

MESSAGE="${1:-Notification}"
SOUND="${2:-default}"

PROJECT_DIR=$(/usr/bin/python3 -c "import sys,json; print(json.load(sys.stdin).get('cwd',''))" 2>/dev/null)
PROJECT=$(basename "$PROJECT_DIR")
BRANCH=$(git -C "$PROJECT_DIR" rev-parse --abbrev-ref HEAD 2>/dev/null || echo "")

SUBTITLE="${PROJECT}"
[ -n "$BRANCH" ] && SUBTITLE="${PROJECT} · ${BRANCH}"

# Resolve the actual terminal app (TERM_PROGRAM may be "tmux")
TERM="${TERM_PROGRAM}"
if [ "$TERM" = "tmux" ] || [ -n "$TMUX" ]; then
  # Use TERM_PROGRAM_OUTER if available, otherwise detect the parent terminal
  if [ -n "$TERM_PROGRAM_OUTER" ]; then
    TERM="$TERM_PROGRAM_OUTER"
  else
    # tmux server is detached — detect terminal from the tmux client process
    TMUX_CLIENT_PID=$(tmux display-message -p '#{client_pid}' 2>/dev/null)
    if [ -n "$TMUX_CLIENT_PID" ]; then
      PARENT_PID="$TMUX_CLIENT_PID"
      while [ "$PARENT_PID" -gt 1 ] 2>/dev/null; do
        PARENT_NAME=$(ps -p "$PARENT_PID" -o comm= 2>/dev/null | xargs basename 2>/dev/null)
        case "$PARENT_NAME" in
          ghostty|iTerm2|Terminal|WezTerm|alacritty|kitty)
            TERM="$PARENT_NAME"
            break
            ;;
        esac
        PARENT_PID=$(ps -p "$PARENT_PID" -o ppid= 2>/dev/null | tr -d ' ')
      done
    fi
  fi
fi

# Map terminal name to bundle ID for click-to-activate
case "$TERM" in
  ghostty)          BUNDLE_ID="com.mitchellh.ghostty" ;;
  iTerm.app|iTerm2) BUNDLE_ID="com.googlecode.iterm2" ;;
  Apple_Terminal|Terminal) BUNDLE_ID="com.apple.Terminal" ;;
  WezTerm)          BUNDLE_ID="com.github.wez.wezterm" ;;
  alacritty)        BUNDLE_ID="org.alacritty" ;;
  kitty)            BUNDLE_ID="net.kovidgoyal.kitty" ;;
  *)                BUNDLE_ID="" ;;
esac

if command -v terminal-notifier &>/dev/null && [ -n "$BUNDLE_ID" ]; then
  # If inside tmux, use -execute to switch to the exact window/pane on click
  if [ -n "$TMUX" ] && [ -n "$TMUX_PANE" ]; then
    TMUX_SOCKET=$(echo "$TMUX" | cut -d, -f1)
    TMUX_BIN=$(which tmux)
    TMUX_SESSION=$(tmux display-message -t "$TMUX_PANE" -p '#{session_name}' 2>/dev/null)
    TMUX_WINDOW=$(tmux display-message -t "$TMUX_PANE" -p '#{window_index}' 2>/dev/null)
    # Map bundle ID to app name for AppleScript activation
    case "$BUNDLE_ID" in
      com.mitchellh.ghostty)    APP_NAME="Ghostty" ;;
      com.googlecode.iterm2)    APP_NAME="iTerm" ;;
      com.apple.Terminal)       APP_NAME="Terminal" ;;
      com.github.wez.wezterm)   APP_NAME="WezTerm" ;;
      org.alacritty)            APP_NAME="Alacritty" ;;
      net.kovidgoyal.kitty)     APP_NAME="kitty" ;;
      *)                        APP_NAME="" ;;
    esac
    ACTIVATE_CMD=""
    [ -n "$APP_NAME" ] && ACTIVATE_CMD="osascript -e 'tell application \"${APP_NAME}\" to activate' && "
    terminal-notifier \
      -title "Claude Code" \
      -subtitle "$SUBTITLE" \
      -message "$MESSAGE" \
      -sound "$SOUND" \
      -execute "${ACTIVATE_CMD}${TMUX_BIN} -S '${TMUX_SOCKET}' select-window -t '${TMUX_SESSION}:${TMUX_WINDOW}' && ${TMUX_BIN} -S '${TMUX_SOCKET}' select-pane -t '${TMUX_PANE}'"
  else
    terminal-notifier \
      -title "Claude Code" \
      -subtitle "$SUBTITLE" \
      -message "$MESSAGE" \
      -sound "$SOUND" \
      -activate "$BUNDLE_ID"
  fi
else
  # Fallback to osascript if terminal-notifier is not installed
  osascript -e "display notification \"$MESSAGE\" with title \"Claude Code\" subtitle \"$SUBTITLE\" sound name \"$SOUND\""
fi
```

</details>

How it fits together: Claude Code fires a hook event → the hook runs `notify.sh` with a message and sound → the script reads the project directory from stdin JSON, resolves the git branch, detects the real terminal app (even inside tmux), and sends a notification via `terminal-notifier`. Click the notification and it brings your terminal to the foreground and switches to the exact tmux pane where the agent is waiting.

## How to set it up yourself

1. **Install terminal-notifier:**

   ```bash
   brew install terminal-notifier
   ```

2. **Create the notification script:**

   ```bash
   # Paste the notify.sh script above into this file
   vim ~/.claude/notify.sh
   chmod +x ~/.claude/notify.sh
   ```

3. **Add hooks to your settings:**
   Add the hooks section from above to `~/.claude/settings.json`. If you already have other settings in the file, merge the `hooks` key into your existing config.

4. **Customize sounds (optional):**
   The script uses `Ping` for permission prompts and `Glass` for task completion. You can pick any sound from `/System/Library/Sounds/`:
   ```bash
   ls /System/Library/Sounds/
   # Basso, Blow, Bottle, Frog, Funk, Glass, Hero, Morse, Ping, Pop, Purr, Sosumi, Submarine, Tink
   ```
   Preview a sound before choosing: `afplay /System/Library/Sounds/Glass.aiff`

That's it. Next time a Claude Code agent needs your attention, you'll hear it and see which project it's about.

## Reflection

Throughout this entire process — five iterations from a basic notification to a tmux-aware, pane-switching system — I didn't write a single line of code or do any manual research. Each iteration followed the same pattern:

1. Describe the problem or annoyance
2. Claude proposes a solution
3. Test it
4. Give feedback on what didn't work
5. Claude refines

I didn't look up how hooks work. I didn't read the `terminal-notifier` README. I didn't figure out how to detect the terminal app inside tmux. I just described what I wanted and what went wrong, and Claude handled the implementation and the research.

This is what working with an AI coding agent feels like when it works well — you stay focused on the _what_ and the _why_, and the agent handles the _how_.
