# Forge Documentation

Documentation for [Forge](https://github.com/tnelson/forge), built with MkDocs Material and versioned with mike.

## Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Serve locally with hot reload (no version dropdown)
mkdocs serve

# Build static site
mkdocs build
```

## Versioning with mike

### How mike Works

- **Source markdown** lives in Git branches (e.g., `main`, `docs/4.2`)
- **Built HTML** is deployed to the `gh-pages` branch with version subdirectories
- **Version dropdown** appears when serving from gh-pages (via `mike serve` or GitHub Pages)

```
gh-pages branch (deployed):
├── 4.2/           ← built from docs/4.2 branch
├── 5.0/           ← built from main branch
├── versions.json  ← dropdown reads this
└── index.html     ← redirects to default version
```

### Branch Structure

| Branch | Contains | Deploy Command |
|--------|----------|----------------|
| `main` | Current version (5.x) docs | `mike deploy 5.0 latest` |
| `docs/4.2` | Frozen 4.2 docs | `mike deploy 4.2` |

### Local Preview with Versions

```bash
# Deploy versions locally (does NOT push to remote)
mike deploy 4.2
mike deploy 5.0 latest
mike set-default latest

# Serve with version dropdown
mike serve
# Visit http://localhost:8000
```

### Editing a Specific Version

1. Switch to that version's branch
2. Edit the markdown files in `docs/`
3. Redeploy that version

```bash
# Edit 4.2 docs
git checkout docs/4.2
# make edits to docs/...
mike deploy 4.2

# Edit 5.0 docs
git checkout main
# make edits to docs/...
mike deploy 5.0 latest
```

### Publishing to GitHub Pages

Add `--push` to deploy commands to push to remote:

```bash
mike deploy 5.0 latest --push
mike set-default latest --push
```

### Creating a New Version

When releasing a new major version (e.g., 6.0):

```bash
# 1. Create branch to preserve current docs
git checkout main
git checkout -b docs/5.0
git push origin docs/5.0

# 2. Continue main as new version
git checkout main
# Update docs for 6.0...

# 3. Deploy new version
mike deploy 6.0 latest --push
mike set-default latest --push
```

### mike Commands Reference

| Command | Description |
|---------|-------------|
| `mike deploy <version>` | Build and commit to gh-pages |
| `mike deploy <version> <alias>` | Deploy with alias (e.g., `latest`) |
| `mike deploy <version> --push` | Deploy and push to remote |
| `mike set-default <version>` | Set `/` redirect target |
| `mike list` | List deployed versions |
| `mike delete <version>` | Remove a version |
| `mike serve` | Serve gh-pages locally |

## GitHub Actions

The workflow in `.github/workflows/deploy.yml` automatically:

1. **Push to main** → Deploys as `dev` version
2. **Version tag (v5.0.0)** → Deploys as `5.0` with `latest` alias
3. **Manual dispatch** → Deploy any version on demand

## File Structure

```
forge-documentation/
├── mkdocs.yml              # MkDocs configuration
├── requirements.txt        # Python dependencies (mkdocs, material, mike)
├── docs/
│   ├── index.md            # Home page
│   ├── getting-started/
│   ├── building-models/
│   ├── running-models/
│   ├── testing-chapter/
│   ├── forge-standard-library/
│   ├── electrum/           # Temporal Forge
│   ├── sterling/           # Custom visualizations
│   ├── dsl/
│   ├── images/
│   ├── example-models/
│   ├── stylesheets/
│   ├── javascripts/
│   └── glossary.md
└── .github/
    └── workflows/
        └── deploy.yml      # CI/CD
```

## Contributing

1. Edit markdown files in `docs/`
2. Preview with `mkdocs serve`
3. Submit pull request

For questions, contact Tim Nelson (Tim_Nelson@brown.edu).
