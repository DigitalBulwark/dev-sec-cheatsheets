# Contributing to Dev-Sec-Cheatsheets

Thank you for your interest in contributing to **DigitalBulwark**'s repository. We enforce a high standard for our documentation to ensure that all snippets and guides remain technically accurate, safe to execute, and highly pragmatic.

By participating in this project, you agree to abide by these core contribution guidelines. Please take a moment to review them before submitting a Pull Request.

## 1. Governance & Quality Standard

We are not building a random list of links. Every cheat sheet added should contain **high-value, battle-tested snippets** that a mid-level or senior engineer would use. 
- Avoid basic, trivial commands (e.g., `ls` or `ping`) unless coupled with advanced, unexpected flags.
- Clearly annotate edge cases or risks associated with a command syntax (e.g., "WARNING: this docker command removes all stopped containers irretrievably").

## 2. Setting Up Your Local Environment

1. Fork the repository on GitHub.
2. Clone your fork locally:
   ```bash
   git clone https://github.com/YourUsername/dev-sec-cheatsheets.git
   ```
3. Add the upstream remote to sync your fork:
   ```bash
   git remote add upstream https://github.com/DigitalBulwark/dev-sec-cheatsheets.git
   ```

## 3. Creating a Branch
Never commit directly to the `main` branch. Always work on a feature branch.
Name your branch cleanly:
- `docs/add-k8s-security`
- `fix/typo-in-nmap`
- `feature/new-aws-category`

## 4. Markdown Formatting Requirements
All contributions must adhere to the following Markdown structures:
- Use Level 2 Headings `##` for core categories within a file.
- Wrap all code commands in backticks (` ` `) and define the language syntax (`bash`, `yaml`, `python`).
- Use GitHub Flavored Markdown Alerts (`> [!WARNING]`, `> [!TIP]`) where necessary.

### Example Markdown Format:
```markdown
## Docker Hardening

> [!TIP]
> Always enforce the `--read-only` flag when spinning up untrusted containers to prevent rogue script execution.

```bash
docker run --read-only --rm -it alpine sh
```
```

## 5. Commit Standards
This repository adheres to the [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) standard. Your Pull Request might be rejected if your commit history is messy.

Acceptable prefixes:
- `docs: add nmap firewall bypass tricks`
- `fix: correct typo in docker compose example`
- `style: format markdown spacing in git-recovery.md`

## 6. Submitting the Pull Request
1. Push your branch to your forked repository.
2. Go to the original DigitalBulwark repository and open a Pull Request.
3. Fill out the **Pull Request Template** completely.
4. Wait for a Maintainer to review your code. We try to review within 48 hours.

We value your expertise. Welcome to the Arena.
