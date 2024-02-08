# Running Commands in the Background

To ensure commands continue running even if you're disconnected from the shell or SSH, you can use tools like `nohup`, `screen` or `tmux`.

## Using `nohup`

`nohup` allows a command to run in the background and continue running even after you log out.

```bash
nohup cp -R /path/to/SOURCE_FOLDER/* /path/to/DESTINATION_FOLDER/ &
```

The `&` at the end runs the command in the background, and `nohup` ensures it keeps running after logout.

## Using `screen`

`screen` is a more powerful tool that allows you to create sessions that can be detached and reattached.

1. Start a new screen session:
   ```bash
   screen -S session_name
   ```
2. Run your command without the `&` since `screen` keeps it running:
   ```bash
   rsync -a /path/to/SOURCE_FOLDER/ /path/to/DESTINATION_FOLDER/
   ```
3. Detach from the screen session by pressing `Ctrl+A` then `D`.

You can reconnect to the session later with `screen -r session_name`.

## Using `tmux`

`tmux` is another powerful tool, similar to `screen`, that allows for managing multiple terminal sessions inside a single window. It's especially useful for running long processes in the background, like large data transfers, and for maintaining sessions after disconnection from an SSH or shell session.

Using `tmux`, you can start a new session, run your command, and detach, with the assurance that your process will continue running in the background. Here’s how to use `tmux` for our data copying tasks:

1. **Starting a New `tmux` Session:**
   To begin, start a new `tmux` session by typing:
   ```bash
   tmux new -s session_name
   ```
   This command creates a new session named `session_name`.

2. **Running Your Command:**
   Within the `tmux` session, you can run any of the data copying commands previously mentioned. For example, to use `rsync`:
   ```bash
   rsync -a /path/to/SOURCE_FOLDER/ /path/to/DESTINATION_FOLDER/
   ```
   There's no need to use `&` to background the process, as `tmux` keeps it running inside the session.

3. **Detaching from `tmux` Session:**
   To detach from the session and leave your process running in the background, press `Ctrl+B` followed by `D`. This key combination detaches you from the session but leaves it running.

4. **Reattaching to a `tmux` Session:**
   If you need to check the progress of your command or start a new task, you can reattach to your `tmux` session using:
   ```bash
   tmux attach-session -t session_name
   ```
   This command reopens your `tmux` session where you left off.

!!! INFO "Benefits of Using `tmux`"

    - **Multiple Windows and Panes:** Unlike `nohup`, `tmux` allows for complex window management, including multiple windows and panes within a single session, enhancing productivity and monitoring capabilities.
    - **Session Management:** `tmux` provides robust session management, allowing you to detach and reattach to sessions as needed, which is ideal for long-running processes or tasks initiated over SSH.
    - **Customization:** `tmux` supports extensive customization, including key bindings and appearance settings, making it a versatile tool for power users.