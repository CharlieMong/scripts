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

# Don't run inside tmux
if [ -n "$TMUX" ]; then
  return
fi

LOG_DIR="$HOME/logs"
mkdir -p "$LOG_DIR"

SESSION_NAME="auto_$(date +%s)"

# Create a new tmux session
tmux new-session -d -s "$SESSION_NAME"

# Create 3 panes (even layout)
tmux split-window -h -t "$SESSION_NAME"
tmux split-window -v -t "$SESSION_NAME:0.0"
tmux select-layout -t "$SESSION_NAME" tiled

# Function to configure pane
configure_pane() {
  local pane=$1

  tmux send-keys -t "$pane" "
clear
echo '---------------------------------------'
echo 'Enter name for this pane:'
read PANE_NAME
if [ -z \"\$PANE_NAME\" ]; then
  PANE_NAME=\"pane_\$(tmux display-message -p '#P')\"
fi
LOG_FILE=\"$LOG_DIR/\${PANE_NAME}_\$(date +%Y%m%d_%H%M%S).log\"
echo \"Logging to \$LOG_FILE\"
echo '---------------------------------------'

exec > >(awk '{ print strftime(\"[%Y-%m-%d %H:%M:%S]\"), \$0; fflush(); }' | tee -a \"\$LOG_FILE\") 2>&1
" C-m
}

# Give tmux a moment to initialize
sleep 0.2

# Configure each pane
for pane in $(tmux list-panes -t "$SESSION_NAME" -F "#{pane_id}"); do
  configure_pane "$pane"
done

# Attach
exec tmux attach -t "$SESSION_NAME"
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

