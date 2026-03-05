Create  ~/.tmux.conf

Open:
```
nano ~/.tmux.conf
```
Paste this:
```
# Always start with 3 panes
new-session

# Split panes
split-window -h
split-window -v
select-layout tiled

# Create logs directory automatically
run-shell "mkdir -p $HOME/logs"

# For each pane: prompt for name and enable logging with timestamps
set-hook -g pane-focus-in 'run-shell "
PANE_ID=#{pane_id}
tmux send-keys -t $PANE_ID \"clear; \
echo Enter name for this pane:; \
read PANE_NAME; \
[ -z \\\"\\$PANE_NAME\\\" ] && PANE_NAME=pane; \
LOG_FILE=\\\"$HOME/logs/\\${PANE_NAME}_$(date +%Y%m%d_%H%M%S).log\\\"; \
echo Logging to \\$LOG_FILE; \
exec > >(awk \\\"{ print strftime(\\\\\\\"[%Y-%m-%d %H:%M:%S]\\\\\\\"), \\\\\\$0; fflush(); }\\\" | tee -a \\\"\\$LOG_FILE\\\") 2>&1\" C-m
"'
```
Save and exit.

STEP 3 — Make Terminal Start tmux Automatically

This depends on your terminal:

🟢 If Using macOS Terminal.app

Go to:

Terminal → Settings → Profiles → Shell

Change:

Shells open with:

➡ Command (complete path)

Enter:
```
/usr/bin/tmux
```
🟢 If Using iTerm2

Go to:

Settings → Profiles → General

Command:

➡ Command

Enter:
```
tmux
```
🟢 If Using GNOME Terminal (Linux)

Go to:

Preferences → Profile → Command

Enable:

☑ Run a custom command

Enter:
```
tmux
```
Now every new terminal or tab launches tmux directly.

No .zshrc hacks required.
