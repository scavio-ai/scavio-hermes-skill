# Install Scavio for Hermes Agent

Detailed walkthrough. For the fast path see [README.md](README.md).

## Prerequisites

- Hermes Agent installed and runnable. If not, follow the
  [Hermes quickstart](https://hermes-agent.nousresearch.com/docs/).
- A model provider configured in Hermes (`hermes model`).
- A Scavio account.
  [Sign up](https://dashboard.scavio.dev/sign-up?utm_source=hermes) for the free
  tier: 50 credits on signup, one time, no credit card required.

## Step 1. Get a Scavio API key

1. Sign up at
   [dashboard.scavio.dev/sign-up](https://dashboard.scavio.dev/sign-up?utm_source=hermes).
2. From the dashboard, open the API Keys page and create a key. Copy it now;
   you will not be able to view it again.
3. Keys look like `sk_live_xxxxxxxxxxxx` (or `sk_test_...`).

## Step 2. Store the key in Hermes' env file

Hermes reads environment variables from `~/.hermes/.env`.

```sh
mkdir -p ~/.hermes
echo "SCAVIO_API_KEY=sk_live_xxxxxxxxxxxx" >> ~/.hermes/.env
chmod 600 ~/.hermes/.env
```

You can also export the variable in your shell rc file. Hermes resolves both.

## Step 3. Install

Pick one of the two paths below.

### Option A. As a Hermes plugin (recommended)

This repo carries an Agent Plugins 1.0.0 `plugin.json` at its root, so Hermes
can install it directly:

```sh
hermes plugins install scavio-ai/hermes-agent
```

That pulls the repo and picks up the five skills under `skills/`. You still
need the MCP server registered, so continue with Option B step 2 or step 3
below if the plugin install does not wire it up on your Hermes version.

### Option B. Manual

**1. Clone the repo.**

```sh
git clone https://github.com/scavio-ai/hermes-agent.git
cd hermes-agent
```

**2. Register the MCP server.** Either run the interactive command:

```sh
hermes mcp add scavio --url https://mcp.scavio.dev/mcp
```

Hermes writes the entry to `~/.hermes/config.yaml`. Then open that file and add
the auth header, since the hosted server authenticates with `x-api-key`:

```yaml
mcp_servers:
  scavio:
    url: https://mcp.scavio.dev/mcp
    headers:
      x-api-key: "${SCAVIO_API_KEY}"
```

Or skip the command and merge
[config/hermes.config.snippet.yaml](config/hermes.config.snippet.yaml) into
`~/.hermes/config.yaml` under the top-level `mcp_servers` key. The snippet is
the recommended form: it also narrows the tool surface, which matters (see
Step 4).

Note there is no `transport` key. Hermes infers the transport from the entry:
`url` means HTTP, `command` means stdio.

**3. Install the skills.** Hermes loads skills from `~/.hermes/skills/`.

```sh
cp -r skills/product-deep-research      ~/.hermes/skills/
cp -r skills/competitive-intel-brief    ~/.hermes/skills/
cp -r skills/price-drop-watch           ~/.hermes/skills/
cp -r skills/reddit-sentiment-pulse     ~/.hermes/skills/
cp -r skills/youtube-research-digest    ~/.hermes/skills/
```

Or symlink, so future `git pull`s update them in place:

```sh
for s in skills/*; do
  ln -sf "$(pwd)/$s" ~/.hermes/skills/
done
```

## Step 4. Choose your tool surface

The Scavio MCP server exposes 191 tools across 31 platforms plus Extract.
Registering all of them writes roughly 70k tokens into every session before you
type anything, and exceeds the tool limit some clients enforce. So the server
registers a default subset, and you narrow from there.

**Server side, `x-scavio-platforms`.** A comma-separated allowlist of platform
keys sent as a header on the MCP connection. Omit it and you get the default:
`extract`, `google`, `youtube`, `amazon`, `walmart`, `reddit`, `tiktok`,
`tiktok-shop`, `instagram`, `x`, `linkedin` (106 tools). The keyword `default`
expands to that set and is additive, so `default,zillow,sec` adds two
platforms. `all` registers all 191 and is a large payload in every session.

Full platform keys: `extract`, `google`, `youtube`, `tiktok`, `instagram`,
`reddit`, `x`, `linkedin`, `threads`, `kuaishou`, `amazon`, `walmart`, `ebay`,
`target`, `homedepot`, `tiktok-shop`, `booking`, `airbnb`, `tripadvisor`,
`yelp`, `zillow`, `redfin`, `indeed`, `glassdoor`, `appstore`, `googleplay`,
`g2`, `capterra`, `googleads`, `metaads`, `sec`, `companieshouse`.

**Client side, `tools.include`.** Hermes filters the registered tools down to
an explicit list. The shipped snippet requests six platforms, then includes 18
tools, which covers every platform the five skills in this repo actually call.

Widen either list when you want more of the catalog. If a skill starts failing
because a tool is missing, it was filtered out here.

## Step 5. Try a query

```sh
hermes chat
```

Then prompt:

> Research the Sony WH-1000XM5 across Amazon, Walmart, YouTube reviews, and
> Reddit. Give me a buying brief.

Hermes activates `product-deep-research` and returns a structured brief. A
successful first call is the real verification that the key, the header, and
the tool filter are all correct.

## Troubleshooting

**401 Unauthorized from scavio.**
The key is missing or wrong. Confirm `echo $SCAVIO_API_KEY` resolves in the
shell that launched Hermes; if the key only lives in `~/.hermes/.env`, restart
Hermes after editing that file. Confirm the header is `x-api-key` and not
`Authorization` (the REST API at `api.scavio.dev` uses
`Authorization: Bearer`, the MCP server at `mcp.scavio.dev` uses `x-api-key`).
Regenerate the key from the API Keys page at
[dashboard.scavio.dev](https://dashboard.scavio.dev?utm_source=hermes) if you
suspect it was truncated on copy.

**Out of credits.**
Check your balance with the `get_usage` tool, or on
[dashboard.scavio.dev](https://dashboard.scavio.dev?utm_source=hermes). The
free tier is 50 credits granted once at signup; it does not refill monthly.
Cost per call varies by platform and by endpoint, so a run that fits in the
free tier on Google and Amazon (1 credit each) will not go as far on G2
(5 credits) or the heavier Kuaishou endpoints. Full table in the
[docs](https://scavio.dev/docs?utm_source=hermes).
[Upgrade here](https://scavio.dev/pricing?utm_source=hermes).

**A tool the skill needs is missing.**
Its platform is outside `x-scavio-platforms`, or its name is outside
`tools.include`. Add it in `~/.hermes/config.yaml` and reconnect. See Step 4.

**Skills not activating.**
Check that `~/.hermes/skills/<name>/SKILL.md` exists and that the `name`
frontmatter field matches the directory name.

**General config problems.**
`hermes doctor` diagnoses configuration and dependency issues, and
`hermes doctor --fix` attempts repairs.

**Hermes config key names look different on your version.**
Hermes iterates quickly. Compare `~/.hermes/config.yaml` against
`config/hermes.config.snippet.yaml` and open an issue on this repo so we can
update the snippet.

## Uninstall

If you installed as a plugin:

```sh
hermes plugins remove scavio
```

If you installed manually:

```sh
rm -rf ~/.hermes/skills/{product-deep-research,competitive-intel-brief,price-drop-watch,reddit-sentiment-pulse,youtube-research-digest}
sed -i.bak '/^SCAVIO_API_KEY=/d' ~/.hermes/.env
```

Then delete the `scavio:` entry under `mcp_servers` in `~/.hermes/config.yaml`.
