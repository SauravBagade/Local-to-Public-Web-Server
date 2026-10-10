# Local-to-Public-Web-Server-
Local-to-Public Web Server Using Python HTTP Server and Cloudflare Tunnel
# Host Local Files Online with Python HTTP Server and Cloudflare Tunnel

A step-by-step guide for Windows 

## Goal

Share files from a folder on your Windows laptop over the internet
using:

-   Python's built-in HTTP server
-   Cloudflare Tunnel (`cloudflared`)
-   A temporary public HTTPS URL

You do not need to configure router port forwarding. Your laptop must
remain powered on, connected to the internet, and running both the
Python server and Cloudflare Tunnel.

## Architecture

``` text
Mobile / Another Computer
          |
          | HTTPS public URL
          v
Cloudflare Quick Tunnel
          |
          | forwards requests to localhost:8080
          v
Python HTTP Server (port 8080)
          |
          v
C:\\PublicFiles
```

## Step 1 --- Install Python

1.  Open the official Python downloads page:
    <https://www.python.org/downloads/windows/>.
2.  Download a current Python 3 release for Windows.
3.  Run the installer.
4.  If the installer shows the option **Add python.exe to PATH**, select
    it.
5.  Select **Install Now**.
6.  Open **Command Prompt (CMD)** and verify:

``` cmd
python --version
```

If Windows cannot find `python`, try:

``` cmd
py --version
```

If neither command works, reopen CMD after installation. If needed,
rerun the installer and enable PATH setup.

## Step 2 --- Create a folder to share

Open CMD and run:

``` cmd
mkdir C:\PublicFiles
```

If the folder already exists, CMD may report that it already exists; you
can continue.

## Step 3 --- Add a test file

Run:

``` cmd
echo Hello from my Windows laptop > C:\PublicFiles\index.html
```

This creates a simple HTML file. You can later copy other files into
`C:\PublicFiles`.

**Security:** Put only files you intend to share in this folder. Do not
place passwords, private documents, SSH keys, API tokens, or other
sensitive files here.

## Step 4 --- Start the Python HTTP server

In CMD, run:

``` cmd
cd /d C:\PublicFiles
python -m http.server 8080 --bind 127.0.0.1
```

The `--bind 127.0.0.1` option makes the server listen only on the
laptop's loopback interface. Cloudflare Tunnel can connect to it
locally.

Keep this CMD window open.

### Test locally

Open this address in the laptop's browser:

<http://localhost:8080>

You should see the files in `C:\PublicFiles`. The server lists directory
contents when no `index.html` page is available.

## Step 5 --- Download and install Cloudflare Tunnel

1.  Open the official Cloudflare `cloudflared` releases page:
    <https://github.com/cloudflare/cloudflared/releases/latest>.
2.  For most Intel/AMD 64-bit Windows computers, download
    **`cloudflared-windows-amd64.msi`**.
3.  If your Windows installation is 32-bit, choose the 386 build
    instead. Most Windows 10 Pro laptops are 64-bit; verify at
    **Settings → System → About → System type**.
4.  Run the MSI installer and complete setup.

Open a **second** CMD window and verify:

``` cmd
cloudflared --version
```

If Windows says the command is not recognized, close and reopen CMD. If
it still fails, confirm the installation completed and check the install
location.

## Step 6 --- Create a temporary public URL

Keep the Python server running in the first CMD window. In the second
CMD window, run:

``` cmd
cloudflared tunnel --url http://localhost:8080
```

Wait for `cloudflared` to display a URL ending in `trycloudflare.com`,
for example:

``` text
https://random-words.trycloudflare.com
```

The exact URL will be generated for your tunnel; do not use the example
URL above.

## Step 7 --- Test from your mobile or another computer

1.  Copy the actual `https://...trycloudflare.com` URL shown in CMD.
2.  Open it on your phone using mobile data, or on another computer
    using a different internet connection.
3.  You should see the files hosted from `C:\PublicFiles`.

## Step 8 --- Stop or restart the server

To stop either process, focus its CMD window and press **Ctrl+C**.

-   If you stop Python, the files will no longer be served.
-   If you stop `cloudflared`, the public tunnel will stop working.
-   Keep both processes running while you want the site available.
-   A Quick Tunnel URL is temporary and may change when you restart it.

## Troubleshooting

### `python` is not recognized

Try:

``` cmd
py --version
py -m http.server 8080 --bind 127.0.0.1
```

Run the server command from `C:\PublicFiles` after changing into that
directory.

### `cloudflared` is not recognized

Close and reopen CMD, then run:

``` cmd
cloudflared --version
```

If it still fails, confirm that the MSI installation succeeded or run
the executable from its installed directory.

### Port 8080 is already in use

Choose another port, such as 8081:

``` cmd
python -m http.server 8081 --bind 127.0.0.1
```

Then, in the second CMD window, use the matching port:

``` cmd
cloudflared tunnel --url http://localhost:8081
```

### The public URL does not load

-   Confirm the Python server works at `http://localhost:8080`.
-   Confirm both CMD windows are still running.
-   Check that the tunnel command points to the same port as the Python
    server.
-   Review error messages in both windows.

## Important security notes

-   Python's built-in HTTP server is intended for simple testing and
    file sharing, not as a hardened production web server.
-   Anyone who can reach the public URL may be able to view or download
    files served by this setup.
-   Do not use this setup to publish confidential files or to provide
    full remote desktop access.
-   This configuration exposes the files in the selected folder; it does
    **not** let visitors control your Windows desktop.
-   For private access, use an authenticated solution such as Cloudflare
    Access with a properly configured tunnel, or a remote-access tool
    configured with strong authentication.

## Official references

-   Python for Windows: <https://www.python.org/downloads/windows/>
-   Python `http.server` documentation:
    <https://docs.python.org/3/library/http.server.html>
-   Cloudflare Tunnel downloads:
    <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/>
-   Cloudflare Quick Tunnels:
    <https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/>

------------------------------------------------------------------------

**Quick command summary**

CMD window 1:

``` cmd
cd /d C:\PublicFiles
python -m http.server 8080 --bind 127.0.0.1
```

CMD window 2:

``` cmd
cloudflared tunnel --url http://localhost:8080
```

Open the generated `https://...trycloudflare.com` address from another
device.
