# aiserver.com
Setting up a local AI server is an incredibly rewarding project, but moving it beyond your local network while maintaining security requires careful architecture.

# A Simple Greeting Exchange

**Exported:** 7/15/2026 19:04:51  
**Link:** [https://gemini.google.com/u/2/app/33b1305f6eaacb2f?pageId=none](https://gemini.google.com/u/2/app/33b1305f6eaacb2f?pageId=none)

## Prompt:
Hi

## Response:
Hi there! What's on your mind today?

## Prompt:
I am setting up a local AI server for myself. I was able to access a local ai model running with Ollama on Zorin OS from any device on my local network using off grid app on Android and on pyGPT on desktop. However, I am thinking if it is possible to access it from anywhere with internet without compromising my home network. I could host a static frontend on cloudfare pages or github using the ui provided by openwebui, and connect it to a cloudfare tunnel or similar. I also think that it would be possible to host both the frontend and the AI, but use a service different from cloudfare because the Zero Trust free account requires a payment card for any usage that exceeds free limits.

## Response:
Setting up a local AI server is an incredibly rewarding project, but moving it beyond your local network while maintaining security requires careful architecture.

Regarding your plan to split the frontend and backend: **Open WebUI does not support a purely static frontend deployment** (like a traditional static HTML/JS site on Cloudflare Pages or GitHub Pages). It uses a Python/SvelteKit backend architecture that handles user authentication, data management (SQLite/PostgreSQL), and direct API communications with Ollama. Because of this, you need to host the Open WebUI container alongside Ollama on your Zorin OS machine and expose Open WebUI safely.

If you want to access your server from anywhere **without entering a credit card for Cloudflare Zero Trust** or compromising your home network, here are the two best modern approaches.

## Approach 1: The Zero-Trust VPN (Tailscale)

Instead of opening your server to the public internet, you bring your remote devices into your private network. This is the **most secure** method and completely circumvents the need for reverse proxies or public tunnels.

*   **How it works:** Tailscale creates a secure, encrypted peer-to-peer mesh network (built on WireGuard) between your devices.
    
*   **The Cost:** 100% Free for personal use (up to 3 users and 100 devices) with **no credit card required**.
    
*   **Security:** Incredible. No public ports are open to the internet. If someone doesn't have your specific Tailscale profile authenticated on their device, your AI server doesn't even exist to them.
    

### Implementation:

1.  Install Tailscale on your Zorin OS server:
    
    ```bash
    curl -fsSL https://tailscale.com/install.sh | sh
    tailscale up
    ```
    
2.  Install the Tailscale app on your Android phone, laptop, or any remote device and log into the same account.
    
3.  Tailscale will assign a static, private IP (e.g., `100.x.y.z`) to your Zorin OS server.
    
4.  From anywhere in the world, type `http://100.x.y.z:8080` (or whatever port Open WebUI/Ollama is running on) into your browser or app, and you are securely connected.
    

## Approach 2: Open Source & Credit-Card-Free Public Tunnels

If your goal is a public URL (e.g., so you can share it with a friend or access it from a device where you can't install a VPN client) without using Cloudflare, you can use open-source or developer-friendly tunneling alternatives.

### Option A: zrok (By OpenZiti)

**zrok** is a phenomenal, open-source zero-trust tunneling platform.

*   **Why use it:** Their free cloud SaaS tier (**zrok.io**) does *not* require a credit card. It allows you to create public HTTP endpoints instantly.
    
*   **Setup:** You download the lightweight CLI on Zorin OS, run `zrok enable`, and then map a tunnel to your Open WebUI port:
    
    ```bash
    zrok share public http://localhost:8080
    ```
    
    This gives you a secure public URL. Because it's open-source, you can even self-host the root infrastructure later if you want total control.
    

### Option B: LocalXpose or Pinggy

*   **LocalXpose:** A production-ready tunneling tool with a free tier that gives you 2 active HTTP tunnels without forcing card registration.
    
*   **Pinggy.io:** Extremely clever because it requires **no installation**. It runs entirely over SSH:
    
    ```bash
    ssh -p 443 -R0:localhost:8080 free.pinggy.io
    ```
    
    *(Note: Pinggy's free tier tunnels expire after 60 minutes, so it's better for testing than a permanent server setup unless you upgrade to their cheap $2.50/mo tier).*
    

## Recommended Architecture Blueprint

If you choose a public tunnel approach (like **zrok**), you must protect Open WebUI since it will be visible to the web.

1.  **Run Open WebUI + Ollama via Docker on Zorin OS:** This keeps environments isolated and predictable.
    
2.  **Lock Down Open WebUI Admin Controls:** Before opening the tunnel, create your admin account, then immediately go to Admin Settings and **disable public sign-ups**. This stops random internet bots from creating accounts and burning your hardware's processing power.
    
3.  **Bind Ollama Locally:** Ensure Ollama itself is only listening on `127.0.0.1:11434` (default behavior) so it isn't directly exposed, letting Open WebUI handle all the front-facing traffic securely.
    

Which path sounds closer to your goal—a completely locked-down private network connection via Tailscale, or configuring a public URL via an alternative tunnel?

## Prompt:
I'd like to try zrok. Can you guide me step by step to achieve this?

## Response:
Excellent choice. **zrok** is built directly on top of OpenZiti (a massive enterprise zero-trust networking project), making it incredibly fast and safe.

Because Open WebUI requires a persistent web link so your mobile app or external laptop can reliably reach it, we will use zrok's **"Reserved Share"** feature. This ensures that every time you run it, you get the exact same URL instead of a random string.

Here is your step-by-step guide to setting this up on Zorin OS.

## Step 1: Create Your Free Account

1.  Go to **[zrok.io](https://zrok.io)** and click **Sign Up**.
    
2.  Enter your email address. You will receive a verification email.
    
3.  Click the link in the email, set your password, and log in to the dashboard.
    
4.  On your dashboard (or under the "Enable Your Environment" tab), you will see a command like this: `zrok enable <your-secret-token>` *Keep this window open or copy that token down; we need it in Step 3.*
    

## Step 2: Install zrok on Zorin OS

Open your terminal on Zorin OS. Since Zorin is based on Ubuntu/Debian, we will use the official package manager repository to ensure it updates cleanly.

Run this command to download and run the installer script:

```bash
curl -sSf https://get.openziti.io/install.bash | sudo bash -s zrok
```

Once completed, verify the installation succeeded by checking the version:

```bash
zrok version
```

## Step 3: Link Your Machine to Your Account

Now, tell the zrok client on your computer who you are using the secret token from Step 1:

```bash
zrok enable YOUR_SECRET_TOKEN_HERE
```

*(Replace `YOUR_SECRET_TOKEN_HERE` with your actual token).*

## Step 4: Secure Open WebUI Before Sharing

Before generating your public link, make sure random people on the internet can't sign up and high-jack your Ollama backend.

1.  Open your local Open WebUI dashboard (`http://localhost:8080` or whichever local port you configured).
    
2.  Log into your admin account.
    
3.  Go to **Admin Settings** -> **Signups**.
    
4.  **Disable public signups**. Now, only you can log in.
    

## Step 5: Reserve Your Permanent Public URL

Instead of a temporary ephemeral link that changes every time your server reboots, we will reserve a static subdomain.

Assuming your Open WebUI is listening on port **`8080`**, run:

```bash
zrok reserve public http://localhost:8080 --backend-mode proxy
```

The terminal will spit out a success message and give you a permanent share token name (it usually looks like a short random string of characters, e.g., `v31kda8fj`). **Copy this share token.**

## Step 6: Fire Up the Tunnel!

To actually open the bridge between the public web and your local app, use the token you just reserved:

```bash
zrok share reserved YOUR_RESERVED_TOKEN
```

*(Replace `YOUR_RESERVED_TOKEN` with the string from Step 5).*

Your terminal will change into a monitoring screen displaying your clean, dedicated public URL, which looks something like: `[https://v31kda8fj.share.zrok.io](https://v31kda8fj.share.zrok.io)`

Take that URL, plug it into your off-grid app on Android, or bookmark it on your remote browsers, and you are officially connected from anywhere in the world! When you want to close the tunnel, just hit `Ctrl + C` in the terminal.

## Step 7 (Optional): Make it Run Permanently in the Background

Right now, if you close that terminal tab, the tunnel dies. If you want zrok to launch automatically when your Zorin OS machine boots up so your server is always ready, you can set up a quick background process (systemd service).

Would you like me to show you how to write the configuration script to keep this running 24/7 in the background?

## Prompt:
It is almost working! Now I can access openwebui from anywhere with the link from zrok, and it actually detects the ai model running on the PC: gemma2:2b.  However, I tried saying "hello", and it returned:  registry.ollama.ai/library/gemma2:2b does not support tools  This also happens if I go to http://localhost:8080

## Response:
That is a huge milestone! Getting the tunnel working and seeing the model connect from anywhere is a massive win.

The error you are seeing—`registry.ollama.ai/library/gemma2:2b does not support tools`—is actually a known behavior with recent updates of Open WebUI when paired with smaller models. Open WebUI heavily relies on **Native Function Calling** (Agentic Mode) by default. It injects capabilities like Web Search, Memory, or custom code tools directly into the model's system loop.

However, standard `gemma2:2b` does not natively support tool calling in its Ollama configuration file (its template doesn't have a tool-calling block), so Open WebUI halts the generation because it can't bind its native functions to it.

Here are the two ways to fix this, depending on whether you want a quick workaround or proper tool capabilities.

### Fix 1: Turn off Function Calling for Gemma 2 (Fastest Fix)

If you just want to text-chat with Gemma and don't care about giving it access to web search or plugins, you can tell Open WebUI to stop attempting to force tools on it.

1.  Log into your Open WebUI dashboard as an admin.
    
2.  Go to **Admin Panel** ➡️ **Settings** ➡️ **Models**.
    
3.  Find your `gemma2:2b` model in the list and click the **pencil (edit) icon**.
    
4.  Scroll down until you find **Model Parameters** or **Capabilities / Default Features**.
    
5.  Look for **Function Calling** or **Tools** and toggle it **Off** (or change it from *Native* to *Disabled*).
    
6.  Click **Save**.
    

Try typing "hello" again. It should bypass the tool-check block and reply immediately.

### Fix 2: Swap to a Model That Supports Native Tools

If you *do* want Open WebUI's advanced features (like its built-in web search, RAG document reading, or custom tools), you need a lightweight model that actually has tool-calling built into its architecture. For smaller local servers running on a single RAM slot, these are highly optimized alternatives:

*   **`qwen2.5-coder:7b` or `qwen2.5:3b`** (Highly recommended — excellent at tool-calling for its size).
    
*   **`llama3.2:3b`** (Meta's lightweight model with solid tool support).
    

You can pull them instantly from the Open WebUI admin settings or via your Zorin OS terminal:

```bash
ollama pull qwen2.5:3b
```

Once pulled, select the new model from your dropdown, and tool calling will work out of the box.

The [WebUI Not Seeing Ollama Models video](https://www.youtube.com/watch?v=il5dyWCRbnk) provides a great overview of how Open WebUI maps its interface settings directly to local Ollama containers, which is handy if you ever need to troubleshoot the backend connections further.

## Prompt:
now it worked! After disabling the tools that it does not support, it successfully generated the response. Thanks! How can I configure it to start automatically if the PC reboots or if I power it off and turn it on later?

## Response:
That's awesome! Getting everything running and successfully generating responses is a massive step.

Since Zorin OS uses `systemd` to manage background services, we can create a lightweight service that hooks into your specific user account (where your zrok credentials and token configurations are stored) so it starts automatically on boot or power-on without requiring you to log in or open a terminal.

Here is how to set it up step by step.

## Step 1: Create the systemd Service File

We will create a custom service file named `zrok-ollama.service`. Open your terminal and run:

```bash
sudo nano /etc/systemd/system/zrok-ollama.service
```

Paste the following configuration inside. **Make sure to replace `your_username` with your actual Zorin OS username, and `YOUR_RESERVED_TOKEN` with your zrok token:**

```ini,
[Unit]
Description=zrok Public Share for Open WebUI
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=your_username
Group=your_username
Environment=HOME=/home/your_username
ExecStart=/usr/local/bin/zrok share reserved YOUR_RESERVED_TOKEN --headless
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

> **Why this matters:** Notice the `Environment=HOME=...` line. `zrok` stores its internal configuration inside a hidden folder (`.zrok`) in your home directory. Forcing the environment variable ensures `systemd` can read your credentials even before a user visually logs in. The `--headless` flag keeps the terminal output clean for system logging.

To save and exit Nano, press `Ctrl + O`, then `Enter`, then `Ctrl + X`.

## Step 2: Enable and Start the Service

Now, tell Zorin OS to look for the new service file, enable it to launch at boot, and start it right now:

```bash
# Reload the systemd daemon to recognize your new file
sudo systemctl daemon-reload

# Enable it to start automatically whenever the computer boots
sudo systemctl enable zrok-ollama.service

# Start the service immediately
sudo systemctl start zrok-ollama.service
```

## Step 3: Verify Everything Is Running Perfectly

Check the status of your new background tunnel to make sure it didn't encounter errors:

```bash
sudo systemctl status zrok-ollama.service
```

You should see an **`active (running)`** status in green.

If you ever want to check the live background logs or verify that requests are hitting the tunnel, you can query the system log journal:

```bash
sudo journalctl -u zrok-ollama.service -f
```

Your tunnel is now officially a persistent daemon. You can reboot your PC, turn it off completely for the night, and the second Zorin OS catches an internet connection during its next boot cycle, your URL will spring right back to life!

## Prompt:
Is this an expected output?:  ● zrok-ollama.service - zrok Public Share for Open WebUI       Loaded: loaded (/etc/systemd/system/zrok-ollama.service; enabled; preset: enabled)       Active: activating (auto-restart) (Result: exit-code) since Wed 2026-07-01 18:04:05 PDT; 4s ago      Process: 127658 ExecStart=/usr/local/bin/zrok share reserved gh3d4ryhiv9t --headless (code=exited, status=203/EX>     Main PID: 127658 (code=exited, status=203/EXEC)          CPU: 970us  lines 1-6/6 (END)

## Response:
That is actually a failure code, but don't worry—it points to exactly what is wrong.

The critical part of your log is **`status=203/EXEC`**. In `systemd` language, this means the system can't find the `zrok` program file at the exact path we gave it (`/usr/local/bin/zrok`). Since Zorin OS is built on Ubuntu, the package manager might have installed `zrok` into a slightly different directory like `/usr/bin/zrok` or `/bin/zrok`.

We can fix this in less than two minutes.

### Step 1: Find where zrok is hiding

Run this command in your terminal to see exactly where `zrok` is located on your system:

```bash
which zrok
```

It will output a path. It will likely look like `/usr/bin/zrok` instead of `/usr/local/bin/zrok`. **Copy whatever path it gives you.**

### Step 2: Update the service configuration

Open the service file back up to swap the paths:

```bash
sudo nano /etc/systemd/system/zrok-ollama.service
```

Look at the **`ExecStart=`** line. Change the beginning of that path to match exactly what your `which zrok` command just returned. For example, if it returned `/usr/bin/zrok`, change that line to look like this:

```ini,
ExecStart=/usr/bin/zrok share reserved gh3d4ryhiv9t --headless
```

Save and close Nano (`Ctrl + O`, `Enter`, `Ctrl + X`).

### Step 3: Reload and Kickstart

Tell `systemd` to read your adjustment and fire up the engine again:

```bash
sudo systemctl daemon-reload
sudo systemctl restart zrok-ollama.service
```

Now, check the status one more time:

```bash
sudo systemctl status zrok-ollama.service
```

You should see a glorious green **`active (running)`** status message! Let me know if that brings it right up.

## Prompt:
Totally different:  ● zrok-ollama.service - zrok Public Share for Open WebUI       Loaded: loaded (/etc/systemd/system/zrok-ollama.service; enabled; preset: enabled)       Active: active (running) since Wed 2026-07-01 18:07:35 PDT; 6s ago     Main PID: 128048 (zrok)        Tasks: 12 (limit: 18682)       Memory: 34.3M (peak: 35.2M)          CPU: 259ms       CGroup: /system.slice/zrok-ollama.service               └─128048 /usr/bin/zrok share reserved gh3d4ryhiv9t --headless  Jul 01 18:07:35 braunbar-hp-envy systemd[1]: Started zrok-ollama.service - zrok Public Share for Open WebUI.  Jul 01 18:07:36 braunbar-hp-envy zrok[128048]: {"file":"/__w/zrok/zrok/cmd/zrok/shareReserved.go:132","func":"main.(>  Jul 01 18:07:36 braunbar-hp-envy zrok[128048]: {"file":"/__w/zrok/zrok/cmd/zrok/shareReserved.go:147","func":"main.(>  Jul 01 18:07:37 braunbar-hp-envy zrok[128048]: {"file":"/__w/zrok/zrok/cmd/zrok/shareReserved.go:330","func":"main.(>  lines 1-14/14 (END)

## Response:
That is beautiful! You are officially in business.

The green **`active (running)`** status means the background process launched flawlessly, hooked into your zrok profile, grabbed your `gh3d4ryhiv9t` token, and initialized the proxy tunnel.

Those logs at the bottom (`shareReserved.go`) are just zrok's inner Go engine reporting that it successfully mapped the environment parameters in headless mode.

Your local server is now fully untethered. You can close your terminal, walk away, restart the HP Envy whenever you want, and your remote web UI connection will handle itself automatically. Have fun exploring local AI processing from anywhere!

