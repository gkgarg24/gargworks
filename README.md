# GargWorks

Personal website and portfolio — [gargworks.com](https://gargworks.com)

## Live Projects

| Subdomain | Project | Stack |
|-----------|---------|-------|
| [endax.gargworks.com](https://endax.gargworks.com) | Energy Data Explorer | Go · DuckDB · AWS EC2 ARM64 |
| [patscape.gargworks.com](https://patscape.gargworks.com) | U.S. Patent Explorer | Go · DuckDB · OCI ARM64 |
| [signum.gargworks.com](https://signum.gargworks.com) | OIDC Identity Provider | Java 26 · Helidon SE · DuckDB · AWS EC2 ARM64 |

## Structure

```
├── hugo.toml           # Site config (colors, menu, metadata)
├── assets/css/         # Stylesheet (processed via css.Build + hugo:vars)
├── content/
│   ├── _index.md       # Home page
│   ├── about/          # About/profile
│   └── projects/       # Project pages (personal + work)
├── layouts/
│   ├── _default/       # baseof, list, single templates
│   ├── partials/       # header, footer
│   └── index.html      # Home page template
└── public/             # Generated output
```

## Tech Stack

- Hugo 0.163+ (static site generator)
- CSS via Hugo's `css.Build` pipeline with `hugo:vars` injection
- Dark mode (OS preference + manual toggle, colors from hugo.toml)
- Inter + Lora fonts
- Cloudflare Pages (hosting + CDN)
- Cloudflare DNS

## Development

```bash
hugo server -D    # Local dev server with drafts
hugo              # Build to public/
```

## Deployment

Cloudflare Pages with automatic deployments from the main branch.
