# Using EVE‑NG on Bluefin Linux: Opening Telnet Sessions Inside a Distrobox Container with Ptyxis

Accessing EVE‑NG from an immutable Linux like Bluefin (a Fedora Silverblue spin) is a fantastic setup for network labs, but there’s one snag I hit immediately:

You click a device in EVE‑NG expecting a console window to open…
and nothing happens.

Yet telnet works perfectly when you run it manually:
```
telnet <eve-ng-ip> <port>
```

## So what’s going on?

The short answer is simple:
> Your browser can’t launch telnet:// links because the host has no handler for that protocol.

On immutable systems, this is expected. The host doesn’t have telnet, xterm, or any terminal capable of handling the link. But the fix is elegant:
**Use a distrobox container and launch the console inside Ptyxis**, Bluefin’s native terminal.

This post is how I built my setup.

## Why Telnet Links Don’t Work on Bluefin

When you click a device in EVE‑NG, your browser tries to open a URL like:
```
telnet://10.0.0.5:32770
```

GNOME hands this to xdg-open, which hands it to GIO, which looks for a handler for the telnet:// scheme.

On Bluefin/Silverblue:

   -  No telnet client exists on the host
   -  No terminal exists on the host
   -  No handler is registered
   -  The host cannot see apps inside your distrobox container

So nothing happens.

The solution is to:

   -  Create a tiny host-side handler script
   -  Register it as the telnet:// protocol handler
   -  Forward the telnet session into your distrobox container
   -  Launch Ptyxis inside the container running telnet

This keeps the host clean and fully immutable.

### Step 1 — Install Telnet Inside Your Distrobox Container

Enter your container:
```
distrobox enter cloud
```

Install telnet:
```
sudo dnf install telnet
```

Ptyxis is already available on Bluefin, and distrobox exposes it inside containers automatically.

### Step 2 — Create the Telnet Handler Script on the Host

Create:
```
sudo nano /usr/local/bin/eve-ng-telnet
```

Paste the full working script:
```
#!/usr/bin/env bash

# Log calls from GNOME/GIO (optional)
echo "EVE-NG handler called with: $1" >> /tmp/eve-ng.log

URL="$1"

# Extract host and port
HOST=$(echo "$URL" | sed 's/telnet:\/\/\([^:]*\):.*/\1/')
PORT=$(echo "$URL" | sed 's/.*://')

# Absolute path to distrobox (GNOME requires this)
DISTROBOX_BIN="/usr/bin/distrobox"

# Your container name
CONTAINER="cloud"

# Launch Ptyxis inside the container running telnet
$DISTROBOX_BIN enter "$CONTAINER" -- ptyxis -- telnet "$HOST" "$PORT"
```

Make it executable:
```
sudo chmod +x /usr/local/bin/eve-ng-telnet
```

This script is the bridge between GNOME and my container.

### Step 3 — Create the Desktop Entry

Create:
```
mkdir -p ~/.local/share/applications
nano ~/.local/share/applications/eve-ng-telnet.desktop
```

Paste:
```
[Desktop Entry]
Name=EVE-NG Telnet Handler (Ptyxis + Distrobox)
Exec=/usr/local/bin/eve-ng-telnet %u
Terminal=false
Type=Application
MimeType=x-scheme-handler/telnet;
```

Register it:
```
xdg-mime default eve-ng-telnet.desktop x-scheme-handler/telnet
```

You should see:
```
eve-ng-telnet.desktop
```

### Step 4 — Test the Handler

Run:
```
xdg-open telnet://127.0.0.1:23
```

We should now see Ptyxis open inside your distrobox container with a working telnet session.

If we click a device in EVE‑NG, a terminal will open connected to the relevant device.

## Final Result

We now have:

   -  A fully immutable Bluefin/Silverblue host
   -  A distrobox container with telnet installed
   -  A clean GNOME/GIO protocol handler
   -  EVE‑NG console windows opening inside Ptyxis, the native Bluefin terminal

So far, this is the most seamless, native-feeling way to integrate EVE‑NG with Bluefin Linux.



