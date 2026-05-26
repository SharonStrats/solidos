# Playground Hosting Checklist

This is a practical checklist for hosting a SolidOS Pivot-based playground from your forks.

The goal is to help you learn the full path:

- local development
- GitHub forks
- Ubuntu server setup
- domain and DNS
- HTTPS
- deployment and updates

This checklist assumes you want to:

- develop from your own forks
- host the app early
- use Pivot rather than the other local server flows
- add a clear landing page that warns users the site is a temporary playground

## 1. Decide the hosting shape

Use this mental model:

- GitHub stores your source code
- a Linux server runs the app
- Cloudflare manages your domain and DNS
- Nginx or Caddy serves HTTPS and forwards traffic to the app

This app is not just a static site, so GitHub Pages is not enough.

For learning, the simplest and clearest option is:

- buy a domain in Cloudflare
- create a small Ubuntu VPS at DigitalOcean, Hetzner, or similar
- deploy your app on that VPS

DigitalOcean is a good choice if you want a straightforward tutorial-friendly UI.

## 2. Create the GitHub repos you will use

Make sure these forks or repos exist under your GitHub account:

- `SharonStrats/solidos`
- `SharonStrats/solid-logic`
- `SharonStrats/solid-ui`
- `SharonStrats/solid-panes`
- `SharonStrats/mashlib`
- `SharonStrats/pivot`
- `SharonStrats/chat-pane`

Keep these upstream unless you know you need to change them:

- `rdflib`
- `node-solid-server`
- `css-mashlib`
- `pane-registry`

Note: if a repo cannot be forked normally, create an empty repo in your account and push your local clone to it.

## 3. Prepare the local development environment

On your Mac:

1. Install `nvm` if you do not already have it.
2. Use the repo Node version.
3. From the `solidos` repo root, run setup.
4. Start the Pivot development flow.

Commands:

```sh
cd /Users/sharon/2026Dev/solidos
nvm install
nvm use
npm run setup
npm run watch-pivot
```

What success looks like:

- dependencies install cleanly
- sibling repos are cloned into `workspaces/`
- the watched packages build
- Pivot starts a Community Solid Server on port `3100`

If setup fails because a fork does not exist yet, either create that GitHub repo or point that package back to upstream in `scripts/add`.

## 4. Add the playground landing page before hosting

Before exposing the app publicly, add a front-door page or dashboard that says:

- this is a playground environment
- data may be changed or deleted at any time
- do not store sensitive or important data
- the service is for testing and demos

Recommended behavior:

1. Show the landing page at `/`
2. Put the actual app behind `/app` or a continue button
3. Add a clear acknowledgement button such as `Enter Playground`
4. Optionally require a checkbox acknowledgement before entering

This is worth doing before deployment so the hosted version communicates the environment clearly from day one.

## 5. Buy the domain in Cloudflare

Suggested approach:

1. Create a Cloudflare account
2. Buy a domain there, or transfer one there
3. Decide whether you want the app at the root domain or a subdomain

Examples:

- `myplayground.net`
- `playground.example.com`

For an experiment, a subdomain is often cleaner if you already own a main domain.

## 6. Create the Ubuntu server

DigitalOcean example:

1. Sign in to DigitalOcean
2. Create a new Droplet
3. Choose Ubuntu LTS
4. Pick a basic shared CPU plan to start
5. Add your SSH key during creation
6. Choose a datacenter region near your users
7. Create the server

Suggested starting size:

- 1 GB RAM for early experiments may work, but 2 GB is safer

If the DigitalOcean UI shows a lot of options, pick these:

- Product: `Droplets`
- Droplet type: `Basic`
- Image: latest `Ubuntu LTS`
- CPU option: `Regular` shared CPU
- Size: start with `2 GB / 1 vCPU` if available; use `1 GB / 1 vCPU` only if you are minimizing cost and accept tighter memory
- Datacenter region: the closest region to you or your demo users, for example `New York` if you are mainly on the US east coast
- Authentication: `SSH key`, not password
- Hostname: something clear like `solidos-playground`
- Project: a dedicated project like `SolidOS Playground`
- Backups: `off` at first to save money, unless you expect to keep important state on the box
- Monitoring: `on`
- IPv6: optional; safe to leave `off` for the first pass
- User data: leave blank

Avoid these options for the first deployment:

- `App Platform`
- `Kubernetes`
- `GPU Droplets`
- premium CPU or memory-optimized plans

Why this is the right starting point:

- Pivot is a long-running Node server, not just a static site
- you want direct control over Node, pm2, nginx, and the filesystem
- a small Ubuntu Droplet is the simplest path to understand and debug

Estimated cost shape:

- a small basic Droplet is the cheapest realistic option
- adding backups increases monthly cost
- a reserved IP is not needed for the first pass

If you want the shortest path with the least decision-making, choose:

- `Droplets`
- `Ubuntu LTS`
- `Basic`
- `Regular shared CPU`
- `2 GB / 1 vCPU`
- your nearest region
- `SSH key`
- backups off
- monitoring on
- hostname `solidos-playground`

Save these details:

- server IP address
- SSH username, usually `root` at first
- private key location on your Mac

## 7. Point Cloudflare DNS to the server

In Cloudflare DNS:

1. Create an `A` record
2. Point it to the server IP
3. Use either the root domain or a subdomain

Examples:

- `@` -> `YOUR_SERVER_IP`
- `playground` -> `YOUR_SERVER_IP`

At first, it is usually easier to set the record to DNS-only while you get the origin working.

## 8. Log into Ubuntu and do basic hardening

From your Mac:

```sh
ssh root@YOUR_SERVER_IP
```

On the server:

1. Update packages
2. Create a non-root sudo user
3. Add your SSH key to that user
4. Disable password auth if appropriate
5. Enable the firewall

Typical commands:

```sh
apt update && apt upgrade -y
adduser sharon
usermod -aG sudo sharon
ufw allow OpenSSH
ufw allow 80
ufw allow 443
ufw enable
```

Then reconnect as your normal user:

```sh
ssh sharon@YOUR_SERVER_IP
```

## 9. Install runtime tools on Ubuntu

Install the basics:

```sh
sudo apt update
sudo apt install -y git build-essential nginx
```

Install `nvm` and Node for your user:

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
nvm install
nvm use
```

Install `pm2` globally:

```sh
npm install -g pm2
```

## 10. Clone your repos onto the server

Choose a working directory, for example:

```sh
mkdir -p ~/apps
cd ~/apps
git clone https://github.com/SharonStrats/solidos.git
cd solidos
```

Then run setup:

```sh
nvm use
npm run setup
```

Important:

- this depends on `scripts/add` pointing to repos that actually exist
- if some repos are intentionally not forked, keep those entries pointed at upstream

## 11. Decide how the hosted app should run

For the first deployment, keep it simple:

- use the same package set as local Pivot development
- run the app with `pm2`
- reverse proxy through Nginx

There are two practical ways to do this:

1. create a dedicated production start script in `solidos`
2. reuse an existing start path and adapt it for server deployment

The cleaner choice is a dedicated production start script once the landing page work is done.

## 12. Run the app with pm2

Example shape:

```sh
cd ~/apps/solidos
pm2 start npm --name solidos-playground -- run watch-pivot
pm2 save
pm2 startup
```

Note:

- `watch-pivot` is acceptable for an early learning deployment
- longer term, you will likely want a more production-specific command than a dev watcher

## 13. Put Nginx in front of the app

Create an Nginx site config such as:

```nginx
server {
    listen 80;
    server_name playground.example.com;

    location / {
        proxy_pass http://127.0.0.1:3100;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Then enable it:

```sh
sudo ln -s /etc/nginx/sites-available/solidos-playground /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

## 14. Enable HTTPS

Simplest starting path:

1. install Certbot
2. issue a certificate for your domain
3. let Certbot update the Nginx config

Typical commands:

```sh
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d playground.example.com
```

After that:

1. visit the domain in the browser
2. verify the certificate is valid
3. confirm the app loads over HTTPS

## 15. Connect deployment to GitHub changes

At first, keep deployment manual.

Typical update flow:

```sh
cd ~/apps/solidos
git pull
nvm use
npm install
npm run setup
pm2 restart solidos-playground
```

That is enough while you are learning.

Later, you can add GitHub Actions or a small deploy script.

## 15A. Use a staging branch for live playground deployment

This project is different from a typical single-repo app because your live playground may depend on coordinated changes across multiple repos.

Recommended branch model:

- `main` stays relatively stable
- `staging` is the live playground branch
- short-lived feature branches merge into `staging`

Recommended repos for a `staging` branch:

- `solidos`
- `pivot`
- `mashlib`
- `solid-ui`
- `solid-panes`
- `solid-logic`

You can add `staging` to other repos later if they become part of the playground work.

## 15B. How staging deployment should work

Do not deploy only one repo in isolation.

Because `mashlib`, `solid-ui`, `solid-panes`, and `solid-logic` are linked together, the deployment should refresh the whole workspace whenever any one of the tracked repos changes.

Recommended deployment behavior:

1. a push lands on `staging` in any tracked repo
2. the server pulls the latest `staging` branch for every tracked repo
3. the server reruns the workspace setup/build flow
4. the server restarts the hosted playground process

This avoids version skew between the linked repos.

## 15C. Example server-side staging deploy script

On the Ubuntu server, create one script that refreshes the whole stack.

Suggested file:

- `~/apps/solidos/deploy-staging.sh`

Example shape:

```sh
#!/usr/bin/env bash
set -e

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"

cd "$HOME/apps/solidos"

git checkout staging
git pull origin staging

for repo in pivot mashlib solid-ui solid-panes solid-logic; do
    cd "$HOME/apps/solidos/workspaces/$repo"
    git checkout staging || true
    git pull origin staging
done

cd "$HOME/apps/solidos"
nvm use
npm install
npm run setup
pm2 restart solidos-playground
```

Notes:

- this assumes the server clones are already present
- this assumes each tracked repo has a `staging` branch
- if some repos should remain on `main`, adjust the script explicitly instead of relying on defaults

## 15D. Triggering deployment from GitHub

The simplest setup is one GitHub Action in each repo that runs on pushes to `staging` and SSHes into the server to execute `deploy-staging.sh`.

High-level shape:

1. create an SSH deploy key for the server
2. store the SSH private key as a GitHub Actions secret in each repo
3. add a workflow that runs on push to `staging`
4. in that workflow, SSH to the server and run the deploy script

The same deploy script can be reused by all repos.

## 15E. Practical staging workflow

The simplest day-to-day workflow is:

1. make changes in a feature branch in one or more repos
2. merge those changes into `staging`
3. push `staging`
4. let the server refresh all tracked repos and restart the app
5. test the live playground

When you are happy with a set of changes, you can merge them from `staging` into `main` later.

## 15F. Why this is better than deploying main directly

Using `staging` for the live playground gives you:

- a place to experiment publicly without treating every test as production-ready
- an easier way to coordinate changes across multiple repos
- a stable `main` branch that you can keep cleaner
- a more realistic deployment model for future environments such as `production`

## 16. Verify the hosted playground behavior

Check all of these:

- the landing page warning is visible at first load
- the continue path into the app works
- the site loads over HTTPS
- static assets load correctly
- the app responds on refresh
- login or pod-related flows still work
- you understand where hosted data is stored
- you are comfortable deleting or resetting that data if needed

## 17. Decide where playground data lives

Before inviting users, answer this clearly:

- where does the server write data
- how will you reset it
- how often might you wipe it
- what exactly should users expect

At minimum, your landing page text should make this explicit.

## 18. Good first production refinements

After the first hosted version works, these are the next useful improvements:

1. create a non-watch production start command
2. add a deploy script for repeatable server updates
3. back up the server config and app data
4. add uptime monitoring
5. restrict access while the playground is still rough
6. add a short privacy and retention note

## 19. Minimum path if you want the shortest route

If you want the fastest viable learning path, do this:

1. finish local setup
2. add the playground landing page
3. buy the domain in Cloudflare
4. create an Ubuntu VPS in DigitalOcean
5. point DNS to the VPS
6. install Node, Nginx, and pm2
7. clone your forked `solidos`
8. run setup
9. start the app
10. enable HTTPS

## 20. Questions to answer before you deploy

Use this checklist before hosting publicly:

- Do all required GitHub repos exist under your account?
- Which repos should stay upstream instead of forked?
- What route should show the landing page?
- What exact warning text should the landing page display?
- Will users be allowed to sign in or only browse?
- Where will playground data be stored?
- How will you restart the service if it crashes?
- How will you update the server after each code change?

## Suggested next steps inside this repo

1. narrow `scripts/add` so only the repos you intend to own point at your account
2. get `npm run setup` working locally
3. identify where the current app entry route is implemented
4. add the playground landing page
5. only then provision the server

## Implementation note: where the landing page should go

The current default Pivot UI is served from Mashlib, not from a custom Pivot frontend.

What that means in practice:

- Pivot config serves `./node_modules/mashlib/dist/databrowser.html` as the default HTML UI
- Mashlib builds that file from `src/databrowser.html`
- Mashlib also copies `static/` files into `dist/`

So once you have run `npm run setup`, the first file to inspect for a playground warning or front-door experience will most likely be:

- `workspaces/mashlib/src/databrowser.html`

The supporting server-side config that makes Pivot use that page is in the Pivot repo at:

- `workspaces/pivot/config/customise-me.json`

Recommended first implementation path:

1. start in Mashlib and add the playground warning or landing behavior there
2. only change Pivot config if you decide you want a separate `/` landing page and the actual app under another route such as `/app`

This is the best first target because it changes the page users actually get from Pivot today, without requiring a bigger routing redesign.
