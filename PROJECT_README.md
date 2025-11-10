# Sen Protocol Documentation

This repository contains the official documentation for **Sen Protocol** - a privacy-first, agent-to-agent payment protocol built on Solana with TEE security.

## What's Documented

This GitBook covers:

- **Privacy Architecture** - TEE, PER, and encryption systems
- **X.402 Protocol** - Agent-to-agent negotiation protocol
- **Bazaar Marketplace** - Privacy-preserving deal platform
- **API Reference** - Complete SDK documentation
- **Security** - Threat model and best practices
- **Deployment** - Infrastructure and operations

## Viewing the Documentation

### Online

Published at: **[docs.sen.fi](https://docs.sen.fi)** (coming soon)

### Local Development

```bash
# Install GitBook CLI
npm install -g gitbook-cli

# Install dependencies
cd gitbook
gitbook install

# Serve locally
gitbook serve

# Build static site
gitbook build
```

## Documentation Structure

```
.
├── README.md              # Landing page
├── SUMMARY.md             # Table of contents
├── architecture/          # System architecture
│   ├── privacy.md
│   ├── agents.md
│   └── payments.md
├── protocol/              # X.402 protocol
│   ├── overview.md
│   ├── messages.md
│   └── payment-flow.md
├── marketplace/           # Bazaar marketplace
│   ├── bazaar.md
│   ├── discovery.md
│   └── matching.md
├── api/                   # API reference
│   ├── client-sdk.md
│   ├── agent-api.md
│   └── vendor-api.md
├── security/              # Security documentation
│   ├── threat-model.md
│   └── privacy-guarantees.md
├── deployment/            # Deployment guides
│   ├── aws-nitro.md
│   └── secrets.md
└── examples/              # Code examples
    ├── p2p-transfer.md
    └── negotiation.md
```

## Contributing

We welcome contributions! Please:

1. Fork this repository
2. Create a feature branch (`git checkout -b docs/new-section`)
3. Make your changes
4. Submit a pull request

### Documentation Guidelines

- Use clear, concise language
- Include code examples where relevant
- Add diagrams for complex concepts
- Link to related sections
- Keep line length reasonable (~80-100 chars)
- Use proper Markdown formatting

### Code Examples

All code examples should:
- Be tested and working
- Include necessary imports
- Show error handling
- Be properly commented
- Use TypeScript for type safety

## Building the Documentation

### Prerequisites

- Node.js >= 16
- GitBook CLI

### Commands

```bash
# Install dependencies
gitbook install

# Serve locally (with live reload)
gitbook serve

# Build static site
gitbook build

# Build PDF (requires calibre)
gitbook pdf . ./sen-protocol-docs.pdf

# Build ePub
gitbook epub . ./sen-protocol-docs.epub

# Build Mobi
gitbook mobi . ./sen-protocol-docs.mobi
```

## Publishing

Documentation is automatically published to [docs.sen.fi](https://docs.sen.fi) on push to `main` branch via GitHub Actions.

To publish manually:

```bash
# Build the site
gitbook build

# Deploy to GitHub Pages
cd _book
git init
git add .
git commit -m "Deploy documentation"
git remote add origin https://github.com/sen-fi/gitbook.git
git push -f origin main:gh-pages
```

## License

Documentation is licensed under MIT License - see [LICENSE](LICENSE) for details.

Code examples are released under the same license as the main Sen Protocol repository.

## Support

- **GitHub Issues**: [github.com/sen-fi/gitbook/issues](https://github.com/sen-fi/gitbook/issues)
- **Discord**: Join our community
- **Email**: docs@sen.fi

## Related Repositories

- **Sen Protocol**: [github.com/sen-fi/sen-protocol](https://github.com/sen-fi/sen-protocol)
- **SDK**: [github.com/sen-fi/sdk](https://github.com/sen-fi/sdk)
- **Examples**: [github.com/sen-fi/examples](https://github.com/sen-fi/examples)
