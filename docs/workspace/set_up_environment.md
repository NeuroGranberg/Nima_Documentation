# Setting Up The Workspace

## VS Code and SSH-Key

Visual Studio Code (VS Code) is a powerful code editor that supports a variety of programming languages and tools. One of its features is the ability to connect to a remote server via SSH, allowing you to edit files and execute commands directly on the server. 

!!! WARNING "Best Practice For SSH Remote Connection"

    Using SSH keys over passwords enhances security and workflow efficiency. SSH keys employ cryptographic techniques, making unauthorized access much harder than cracking passwords. They streamline the login process, removing the need for password entry after initial setup, which is a boon for automation and managing multiple servers. This reliability is crucial for maintaining automated tasks, like CI/CD pipelines, without the hassle of password management.

!!! WARNING "Why VS Code?"

    Visual Studio Code (VS Code) stands out for handling complex data, like MRI imaging's DCM and NIFTI files, due to its extensive extension library and seamless integration with various tools. While VS Code doesn't natively open these files, it supports extensions and external tools for indirect access, making it ideal for software development in medical imaging. Its remote development capabilities are particularly valuable for working with large datasets on powerful remote servers, facilitating efficient processing and collaborative work environments.



In essence, SSH keys provide a secure, stable connection framework, while VS Code offers a versatile platform for development, especially in fields requiring handling of large and complex data formats.

### Prerequisites

- Visual Studio Code installed on your local machine.
- An SSH client installed on your local machine (typically available by default on Linux and macOS, Windows users can use Git Bash or PowerShell).
- Access to a remote server with SSH enabled and your credentials (username and password or an SSH key).

### Step 1: Install the Remote - SSH Extension

1. Open VS Code.
2. Navigate to the Extensions view by clicking on the square icon on the sidebar or pressing `Ctrl+Shift+X`.
3. Search for "Remote - SSH" and click on the install button to add it to VS Code.

### Step 2: Generate an SSH Key Pair (If Required)

If you do not already have an SSH key or wish to create a new one for this connection:

```bash
ssh-keygen -t rsa -b 4096
```

Follow the prompts to generate your key pair, and remember the location where you save it.

### Step 3: Copy the SSH Public Key to Your Remote Server

If using `ssh-copy-id`:

```bash
ssh-copy-id your_username@your_server_ip
```

If manually copying the key, append the content of your `~/.ssh/id_rsa.pub` to the `~/.ssh/authorized_keys` file on the remote server like so:

1. Display your public key using:
   ```
   cat /home/nima/.ssh/id_rsa.pub
   ```
2. Copy the output of the command.

3. Log in to your remote server using your username and password.

4. Once logged in, run the following commands to add your public key to the `authorized_keys` file:
   ```
   mkdir -p ~/.ssh
   echo your_public_key >> ~/.ssh/authorized_keys
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```
   Replace `your_public_key` with the key you copied earlier. Make sure to paste it inside the quotes.


### Step 4: Connect to Your Remote Server Using VS Code

1. Open the Command Palette in VS Code with `Ctrl+Shift+P` or `F1`.
2. Type "Remote-SSH: Connect to Host..." and select it.
3. Enter the SSH connection command in the format `username@hostname` and press Enter.
4. If prompted, select the SSH key you generated or enter your password.

### Step 5: Working on the Remote Server

Once connected, you can:

- Open, edit, and save files on the remote server.
- Use the integrated terminal in VS Code to run commands directly on the server.
- Install extensions on the remote server for a fully integrated development environment.

!!! INFO "Tips for Efficient Remote Development"

    - Use the `Remote-SSH: New Window` command to start a new VS Code window connected to the remote server.
    - Configure VS Code settings and extensions specifically for your remote environment to tailor the development experience to your needs.
    - Utilize port forwarding features of the Remote - SSH extension to access web applications running on the remote server directly from your local machine.






