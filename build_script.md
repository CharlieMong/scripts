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
#!/bin/bash

# Prevent nesting tmux sessions
if [ -n "$TMUX" ]; then
  return
fi

LOG_DIR="$HOME/logs"
mkdir -p "$LOG_DIR"

SESSION_NAME="auto_session"

# If session already exists, attach
tmux has-session -t $SESSION_NAME 2>/dev/null
if [ $? -eq 0 ]; then
  exec tmux attach -t $SESSION_NAME
fi

# Create new session detached
tmux new-session -d -s $SESSION_NAME

# Split into 3 panes
tmux split-window -h -t $SESSION_NAME
tmux split-window -v -t $SESSION_NAME:0.0

# Function to setup each pane
setup_pane() {
  PANE_ID=$1
  tmux send-keys -t $PANE_ID '
read -p "Enter name for this pane: " PANE_NAME
LOG_FILE="$HOME/logs/${PANE_NAME}_$(date +%Y%m%d_%H%M%S).log"
echo "Logging to $LOG_FILE"
exec > >(awk '"'"'{ print strftime("[%Y-%m-%d %H:%M:%S]"), $0; fflush(); }'"'"' | tee -a "$LOG_FILE") 2>&1
clear
' C-m
}

# Setup each pane
setup_pane "$SESSION_NAME:0.0"
setup_pane "$SESSION_NAME:0.1"
setup_pane "$SESSION_NAME:0.2"

# Attach to session
exec tmux attach -t $SESSION_NAME
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
