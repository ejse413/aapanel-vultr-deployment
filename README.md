# aaPanel Vultr: Install the Panel on a Vultr VPS, Link Your Account via API, and Pick a Plan That Actually Fits

aaPanel runs comfortably on even Vultr's smallest servers — the panel itself only asks for 1 CPU core, 512 MB of RAM, and 1 GB of disk — and there are two distinct ways the two products work together. You can install aaPanel directly on a Vultr VPS and run your sites from its web dashboard, or you can connect your Vultr account to aaPanel's aaCloud feature and manage your Vultr servers from inside the panel. This guide covers both routes, plus the ports, plan sizes, and small print that decide whether everything works on day one.

> **Quick picks.** Just learning, or hosting one small site? Vultr's entry-level plan (1 vCPU, 1 GB RAM, 25 GB SSD, 1 TB of transfer — $5/month in the benchmark below; check current rates before deploying) is enough, since aaPanel's official minimum is even lower. Running something that earns money? The benchmark author's floor for production is 2 vCPUs and 4 GB of RAM. 👉 [check Vultr's current plans and pricing](https://www.vultr.com/pricing/?ref=7268283) before you deploy.

## Two different ways to put aaPanel and Vultr together

Most tutorials only cover the first route: install aaPanel on a Vultr server and use it as your hosting control panel. That's the classic setup — a web dashboard for sites, databases, SSL, backups, and Docker on a single VPS.

The second route gets far less coverage. aaPanel's aaCloud docs (updated January 2026) describe connecting a Vultr account directly through Vultr's API, so the panel can list, sync, and manage your Vultr servers from one place. If you run more than one Vultr machine, this is the part worth setting up. Both routes are covered below.

## Installing aaPanel on a Vultr server, step by step

The whole install is a short job: deploy an Ubuntu server, SSH in, run one script, and save the credentials it prints. Vultr maintains its own step-by-step aaPanel guide on its docs site (last updated September 2026), and aaPanel's official quick-start does the same. Here's the combined, current version.

### Step 1: Deploy the right operating system

aaPanel is developed against Ubuntu 22.04 and recommends it, so pick Ubuntu 22.04 when deploying your Vultr server. The officially supported list also includes Ubuntu 24 and 26, Debian 11–13, CentOS 9 and 10, AlmaLinux 8–10, and Rocky Linux 8–10 — but if you have no reason to deviate, don't.

Hardware-wise, aaPanel's stated minimum is 1 core, 512 MB RAM, and 1 GB of storage, which every Vultr plan clears. That doesn't mean every plan is a good idea, though — plan sizing comes later.

### Step 2: SSH in and run the install script

Connect as root (or use your SSH key):


ssh root@your_server_ip


The install script expects root. If you deployed as a sudo user, switch first with `sudo -i` or `su`. Then run aaPanel's current one-liner:


URL=https://www.aapanel.com/script/install_panel_en.sh && if [ -f /usr/bin/curl ];then curl -ksSO "$URL" ;else wget --no-check-certificate -O install_panel_en.sh "$URL";fi;bash install_panel_en.sh forum


Confirm the install directory when prompted. In aaPanel's own documentation example, the whole thing finishes in about a minute. An independent benchmark from November 2022 got the same result: aaPanel installed on a Vultr entry-level box in 1 minute, versus 2 minutes on a same-spec DigitalOcean droplet.

### Step 3: Save the login details (seriously)

When the script finishes, it prints a secure panel URL with a random path, plus a random admin username and password. Copy all of it somewhere safe before you close the terminal. If you forget, `bt default` shows the login info again, and `bt 5` resets the password from the command line.

👉 [Deploy a Vultr server and try the install yourself](https://bit.ly/VulTR) — once the script prints that success banner, the hard part is done.

## Open the right ports — this is where most people get stuck

The install output ends with a warning line listing the ports you need to release, and this line is the single most important thing to read. In Vultr's own tutorial the example panel port is **8888**; in aaPanel's current documentation the example shows **31750**. The exact number depends on your install, so trust the warning your terminal actually printed rather than any tutorial.

Typically that line asks for your panel port plus 888, 80, 443, 20, and 21. Open those in your firewall, then confirm with:


sudo ufw status


If the panel URL refuses to load right after install, an unopened port is the first suspect — not a broken install.

## Your first half hour in the panel

Log in through the secure URL the script gave you. On first load, aaPanel offers to install a web stack — LEMP (Nginx) or LAMP (Apache). Pick one and let it run; Vultr's guide puts that at roughly five minutes depending on server specs.

From there, adding a site is a short loop: open **Website**, choose a PHP or Node.js project, click **Add Site**, enter your domain and PHP version, and select your database. To serve it over HTTPS, hit **Apply for SSL** and submit. Your files live under `/var/www/wwwroot/yourdomain.com/`, and you can upload a site directly or install something like WordPress through it.

## Lock the panel down before adding a real site

The random URL path and random credentials are a start, not a finish. Vultr's guide recommends a short hardening pass in **Settings**:

- Change the panel port away from its default to something less obvious
- Set a custom alias (panel name)
- Set a custom **Security Entrance** string — it's on by default, and without the string the panel can't be reached at all
- Replace the random panel username and password with your own
- Optionally restrict panel access to specific IPs under **Authorized IP**
- Set a session timeout of at least 360 seconds so idle logins expire

One more server-level detail: if the VPS reboots, the panel won't come back up on its own. Start it with:


sudo systemctl start bt


and check its state with `sudo systemctl status bt`.

## Connect your Vultr account to aaPanel's aaCloud

If you'd rather see and manage your Vultr servers from inside aaPanel, the aaCloud route uses Vultr's API, and the setup takes a few minutes.

On the Vultr side: open the **Account** menu, go to **API**, and click **Enable API**. Copy and store the API key immediately — Vultr shows it **only once**, and if you lose it you'll have to regenerate it. Then, in the same API interface, find **Access Control** and toggle on **Allow All IPv4** and **Allow All IPv6**. Be aware that this widens who can call the API against your account; it's what the official setup requires, but it's worth understanding before you flip it.

On the aaCloud side: click **Add account**, choose **Vultr** as the provider, paste the API key, give the account a name, and hit **Add**, then **Verify**. Once it's connected, the **Server** interface lets you use **Sync from Cloud** or **Add Server** to pull your Vultr machines into the panel. From there you can manage the fleet without logging into Vultr's own dashboard for every small task.

## Which Vultr plan does aaPanel need?

Technically, any of them — aaPanel's 512 MB minimum is below everything Vultr sells. Practically, the benchmark below gives better guidance: on a 1 vCPU / 1 GB entry plan, aaPanel itself installs in a minute, but a full software stack (Nginx, MySQL, PHP, phpMyAdmin, Node.js, and aaPanel's Docker plugin, compiled from source) took 44 minutes. Fine for a weekend project; slow if you're in a hurry or running several sites.

| Your use case | Sensible plan size | Why |
| --- | --- | --- |
| Learning aaPanel, one small site or blog | Entry plan (1 vCPU, 1 GB RAM, 25 GB SSD, 1 TB transfer) | Clears aaPanel's minimum with room to spare; the benchmark ran this exact config |
| Production site or client work | At least 2 vCPU / 4 GB RAM | The benchmark author's stated floor for production workloads |
| Multiple sites, databases, or Docker containers | 2 vCPU / 4 GB and up, more if traffic grows | Compiles and databases both get noticeably faster with headroom |

The entry plan was $5/month in that 2022 test; prices move, so 👉 [see what Vultr's plans cost right now](https://www.vultr.com/pricing/?ref=7268283) and pick from the current table.

## How Vultr compared with DigitalOcean and Hetzner in one benchmark

An independent benchmark published in November 2022 installed aaPanel on three brand-new entry-level VPSes — one each from Vultr, DigitalOcean, and Hetzner — all on Ubuntu 22.04, in Germany, and timed everything. The software stack compiled from source on each box: Nginx 1.21, MySQL 8.0, PHP 8.0, phpMyAdmin 5.0, Node.js 18, plus aaPanel's Node and Docker manager plugins.

| Provider | aaPanel install | Software stack install | Idle CPU usage |
| --- | --- | --- | --- |
| DigitalOcean | 2 min | 64 min | 7% |
| Vultr | 1 min | 44 min | 14% |
| Hetzner | 1 min | 41 min | 14% |

As listed in that test, the entry plans cost $6 at DigitalOcean (1 CPU, 1 GB RAM, 25 GB SSD, 1 TB transfer), $5 at Vultr (1 vCPU, 1 GB, 25 GB, 1 TB), and $4.51 at Hetzner (1 vCPU, 2 GB, 20 GB, 20 TB). Hetzner won on raw value and edge-of-the-race performance; DigitalOcean sat idler but compiled noticeably slower. One Vultr-specific gripe from the tester: the server monitoring metrics shown inside aaPanel were thinner than on the other two providers.

The takeaway matches the tester's own conclusion: for a hobby project or personal blog, any of the three is genuinely fine. For production, stop shopping at the entry tier and size up regardless of provider.

## Quick answers

**Is aaPanel free to use?**
Yes. Vultr's docs describe it as an open-source, lightweight Linux control panel, and the install doesn't involve any license step.

**Which OS should I choose when deploying on Vultr?**
Ubuntu 22.04. aaPanel develops against it and recommends it, and both official tutorials assume an Ubuntu-style environment.

**The panel won't load after my server rebooted — what now?**
Start the service: `sudo systemctl start bt`, then verify with `sudo systemctl status bt`. The panel doesn't auto-start after a reboot.

**I lost the panel URL or password.**
Run `bt default` to see the login info again, or `bt 5` to set a new password.

**Do I have to install as root?**
The script expects root. Either log in as root or switch with `sudo -i`; running it with plain `sudo bash` also works per aaPanel's own prompt.

**Why can't I open the panel even though the install succeeded?**
Almost always the port. Check the warning line from your install output, open that port (plus 888, 80, 443, 20, 21) in your firewall, and re-check `sudo ufw status`.

## Before you deploy

Match the plan to the job and this setup holds up well. Testing aaPanel or hosting one personal site: the entry-level Vultr plan is the right call, and aaPanel's minimum specs mean you'll never feel constrained at install time. Anything with real users on the other side: start at 2 vCPUs and 4 GB of RAM, because the compile times on a 1 GB box are the real bottleneck, not the panel. And if you're juggling several Vultr servers, spend the extra five minutes on the aaCloud API link — it's the part of this combo most tutorials never mention.

Three things to do in order: deploy with Ubuntu 22.04, copy the panel credentials the moment the script prints them, and screenshot the port warning line before closing your terminal. Then 👉 [start with the Vultr plan that fits your site](https://bit.ly/VulTR) and you'll have the panel running inside ten minutes.
