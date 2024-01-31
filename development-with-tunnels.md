# Development with Remote Tunnels

This document describes how to use the [Remote Tunnels][remote-tunnels] feature of VSCode to connect
from a local VSCode Desktop or VSCode online to a remote machine, WSL2 instance, or even a Docker
container.

The benefits of using this feature are:
- You don't need to open any ports, use any VPN, RDP, or configure any SSH tunnel in order to work
  on your remote machine.
- You can run **whatever you want** on your remote machine, **from your browser** or from VSCode
  Desktop.

In order to do that, you first need to have a Microsoft or a GitHub account. In this document, I am
using a Microsoft account.

Let's see how to do it for each of the following cases:

- [Remote machine in your network](#remote-machine-in-your-network)
- [WSL2 Instance](#wsl2-instance)
- [Docker container](#docker-container)

## Remote machine in your network

### Setting up the remote machine

#### Step 1 - Install `code CLI`

If you have VSCode installed, you should already have `code CLI` installed. However, if your remote
doesn't have VSCode installed, or if it doesn't have a UI, you can install it by it through a
[standalone install].

#### Step 2 - Turn on Remote Tunnel Access

> **Note:** You can do this step also from the VSCode UI on the remote machine by using the Remote
> Tunnels extension. However, I prefer to do it from the command line.

> :warning: There are some limitations for how many tunnels you can create per account and how much
> data you can transfer. See [Are there usage limits for the tunneling service?][usage-limits] for
> more information.

- Login to your Microsoft account:

  ```bash
  code tunnel user login --provider microsoft
  ```

- Start the tunnel:

  ```bash
  code tunnel service install --accept-server-license-terms --name "remote-tunnel"
  ```

### Connect to the remote machine

#### Step 1 - Connect with your browser

> **Note:** Connecting with your browser proves that you can basically work from anywhere and from
> any device, even from your phone.

- Open [vscode.dev] in your browser.
- Type `Ctrl+Shift+P` and select `Remote-Tunnels: Connect to Tunnel...`
- Choose `Microsoft Account` and login with the same account you used in Step 2.
- Now you should see a list of code tunnels you can connect to. Choose the one you created in Step
  2.

Enjoy running **whatever you want** on your remote machine, **from your browser**.
- Try whatever terminal you want.
- Try even `vim`, `tmux` or `screen`.
- Try SSH to other machines in your network.

#### Step 2 - Connect with VSCode Desktop

> **Note:** You have to install the Remote Tunnels extension in VSCode Desktop.

- Open VSCode Desktop.
- Type `Ctrl+Shift+P` and select `Remote-Tunnels: Connect to Tunnel...`
- Choose `Microsoft Account` and login with the same account you used in Step 2.
- Now you should see a list of code tunnels you can connect to. Choose the one you created in Step
  2.

Enjoy running **whatever you want** on your remote machine, **from VSCode Desktop**.

## WSL2 Instance

### Setting up the WSL2 Instance

#### Step 1 - Enable `systemd`

We want `systemd` to be enabled so that we can start the code tunnel as a service. To enable it,
edit the `/etc/wsl.conf` file and add the following lines:

```bash
[boot]
systemd=true
```

See [Systemd support is now available in WSL][systemd-in-wsl] for more information.

#### Step 2 - Install `code CLI`

Do the same as in Step 1 of [Remote machine in your network](#remote-machine-in-your-network).

### Step 3 - Turn on Remote Tunnel Access

Do the same as in Step 2 of [Remote machine in your network](#remote-machine-in-your-network).

### Connect to the remote WSL2

Now you can connect to the remote WSL2 instance from your browser or from VSCode Desktop, exactly in
the same way as you connect to a remote machine in your network.

## Docker container

### Setting up the Docker container

#### Step 1 - Build the Docker image

- In order to connect to a remote tunnel from a Docker container, you need to have `code CLI`
  installed in the container. The `Dockerfile` below is a good starting point for building such an
  image.

  ```Dockerfile
  FROM ubuntu:22.04

  # Install git and curl
  RUN apt-get update && \
      apt-get install -y git curl && \
      apt-get clean && \
      rm -rf /var/lib/apt/lists/*

  # Download and extract the code cli program
  RUN curl -sL "https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64" \
      --output /tmp/vscode-cli.tar.gz && \
      tar -xf /tmp/vscode-cli.tar.gz -C /usr/bin && \
      rm /tmp/vscode-cli.tar.gz

  # Keep the container running
  CMD [ "tail", "-f", "/dev/null" ]
  ```

- Run this command to build the image:

  ```bash
  docker build -t vscode-remote-tunnel .
  ```
#### Step 2 - Turn on Remote Tunnel Access

- Start the container with the following command:

  ```bash
  docker run -d \
      -v $(pwd):/home/workspace \
      -w /home/workspace \
      --name vscode-remote-tunnel \
      vscode-remote-tunnel
  ```

- Login to your Microsoft account:

  ```bash
  docker exec vscode-remote-tunnel code tunnel user login --provider microsoft
  ```

- Start the tunnel:

  ```bash
  docker exec -d vscode-remote-tunnel code tunnel --accept-server-license-terms --name "docker-tunnel"
  ```

### Connect to the remote Docker container

Now you can connect to the remote Docker container from your browser or from VSCode Desktop, exactly
in the same way as you connect to a remote machine in your network.

### Stopping the tunnel

- Make sure you stop the server for the tunnel and logout from your Microsoft account:

  ```bash
  docker exec vscode-remote-tunnel code tunnel kill
  docker exec vscode-remote-tunnel code tunnel user logout
  ```

## References

- [Developing with Remote Tunnels][remote-tunnels]

[remote-tunnels]: https://code.visualstudio.com/docs/remote/tunnels
[standalone install]: https://code.visualstudio.com/#alt-downloads
[systemd-in-wsl]: https://devblogs.microsoft.com/commandline/systemd-support-is-now-available-in-wsl/
[usage-limits]: https://code.visualstudio.com/docs/remote/tunnels#_are-there-usage-limits-for-the-tunneling-service
[vscode.dev]: https://vscode.dev
