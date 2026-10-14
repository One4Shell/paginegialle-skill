# paginegialle

Installs the **paginegialle** agent skill into any project: `SKILL.md`,
the `references/` docs, `scripts/` helpers, and any bundled `assets/`.

Search Italian businesses and build enriched B2B contact lists from paginegialle.it with the bundled paginegialle CLI. Use whenever the user wants to find Italian companies or professionals by category and city, look up a business's address/phone/email/VAT/website, or produce a lead/contact list for B2B outreach in Italy.

## Requirements

- The skill bundles the `paginegialle` CLI at `scripts/paginegialle` as a
  statically linked **linux/amd64** binary. It runs on Linux x86-64 with no
  dependencies; on other OS/architectures it will not execute and you'll need
  a build for your platform.
- Network access to `paginegialle.it` at runtime. No API key is required.

## Install (recommended)

This repo follows the [Agent Skills specification](https://agentskills.io),
so it installs with a single command via [`npx skills`](https://github.com/vercel-labs/skills)
— no cloning, no config:

```bash
npx skills add https://github.com/One4Shell/paginegialle-skill --skill paginegialle
```

`npx skills` auto-detects which coding agents you have installed and copies
(or symlinks) the skill into the right directory for each — `.agents/skills/`
for OpenCode/Codex/Cursor/Amp, `.claude/skills/` for Claude Code, and so on.
Add `-a <agent>` to target a specific one, or `-g` to install globally
instead of per-project.

## Install (no npm required)

If you'd rather not depend on the `skills` CLI, use the plain shell
installer bundled in this repo instead:

```bash
curl -fsSL https://raw.githubusercontent.com/One4Shell/paginegialle-skill/main/install.sh | bash
```

To pass options through the pipe, use `bash -s --`:

```bash
curl -fsSL https://raw.githubusercontent.com/One4Shell/paginegialle-skill/main/install.sh | bash -s -- --claude
```

Or clone and run it locally (no network access needed at install time):

```bash
git clone https://github.com/One4Shell/paginegialle-skill.git
cd paginegialle
./install.sh --dir /path/to/your/project
```

### `install.sh` options

```
--dir <path>     Project root to install into (default: current directory)
--path <path>    Custom install path, relative to --dir
                  (default: .agents/skills/paginegialle)
--claude         Shorthand for --path .claude/skills/paginegialle
--force          Overwrite existing files (default: skip files that already exist)
-h, --help       Show help
```

Env vars (only used in the `curl | bash` case, i.e. no local checkout):

```
PAGINEGIALLE_REPO     GitHub "owner/repo" to fetch from (default: baked into install.sh)
PAGINEGIALLE_BRANCH   Branch/ref to fetch (default: main)
```

Either method never overwrites existing files unless `--force` is passed, so
it's safe to re-run.

## What gets installed

```
skills/paginegialle/            # repo layout — also discoverable as .agents/skills/paginegialle
├── SKILL.md                      # entry point the agent reads
├── scripts/                      # executable helpers, if any
├── references/                   # docs loaded into context as needed
└── assets/                       # files used in output, if any
```

## After installing

Point your agent at a project containing the skill and describe what you
need — it will pick up `SKILL.md` and follow its instructions from there.

## License

MIT
