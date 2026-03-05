Ubuntu/Debian:

```
sudo apt install tmux
```

✅ Step 2: Create the Setup Script

Create this file:
```
nano ~/.zsh_tmux_auto.sh
```

Paste this:
```
# --- Auto 3-pane tmux setup ---
if [[ -o interactive ]] && [[ -z "$TMUX" ]] && [[ -z "$ZSH_TMUX_AUTO_STARTED" ]]; then
  export ZSH_TMUX_AUTO_STARTED=1

  LOG_DIR="$HOME/logs"
  mkdir -p "$LOG_DIR"

  SESSION_NAME="auto_$(date +%s)"

  tmux new-session -d -s "$SESSION_NAME" zsh
  tmux split-window -h -t "$SESSION_NAME" zsh
  tmux split-window -v -t "$SESSION_NAME:0.0" zsh
  tmux select-layout -t "$SESSION_NAME" tiled

  sleep 0.2

  for pane in $(tmux list-panes -t "$SESSION_NAME" -F "#{pane_id}"); do
    tmux send-keys -t "$pane" '
clear
echo "---------------------------------------"
read "PANE_NAME?Enter name for this pane: "
[[ -z "$PANE_NAME" ]] && PANE_NAME="pane_$(tmux display-message -p "#{pane_index}")"
LOG_FILE="$HOME/logs/${PANE_NAME}_$(date +%Y%m%d_%H%M%S).log"
echo "Logging to $LOG_FILE"
echo "---------------------------------------"

exec > >(awk '"'"'{ print strftime("[%Y-%m-%d %H:%M:%S]"), $0; fflush(); }'"'"' | tee -a "$LOG_FILE") 2>&1
' C-m
  done

  exec tmux attach -t "$SESSION_NAME"
fi
# --- End Auto Setup ---
```

### Make it executable:
```
chmod +x ~/.zsh_tmux_auto.sh
```

### Step 3: Make Zsh Run It Automatically

Add this to the bottom of your ~/.zshrc:
```
# Auto-start 3 pane tmux layout
if command -v tmux >/dev/null 2>&1; then
  source ~/.zsh_tmux_auto.sh
fi
```
Reload zsh:
```
source ~/.zshrc
```
### What Happens Now

Every time you:

Open Terminal

Open a new tab

Start a new Zsh shell

It will:

Launch tmux

Split into 3 panes

Prompt each pane:

Enter name for this pane:

Create logs like:

~/logs/server_20260305_143522.log

Every line will look like:

[2026-03-05 14:35:22] npm start
Layout Overview
 -----------------------
|        |              |
| Pane 0 |   Pane 1     |
|        |              |
|--------|              |
| Pane 2 |              |

 -----------------------


