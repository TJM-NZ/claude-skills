# Claude Code Skills

A collection of professional skills for [Claude Code](https://claude.com/claude-code) - AI-powered CLI tool for software development.

## Skills Included

### Audit & Review Skills
- **technical-review** - Comprehensive technical code review identifying design flaws, code quality issues, and architectural problems.
- **security-audit** - Comprehensive security audit framework covering authentication, injection, cryptography, APIs, and more.
- **seo-audit** - Technical SEO audit analyzing performance, structured data, mobile responsiveness, and site architecture.
- **geo-audit** - Generative Engine Optimization (GEO) audit for AI discoverability (ChatGPT, Perplexity, Gemini).
- **perf-check** - Performance analysis finding bottlenecks, N+1 queries, missing indexes, and inefficient algorithms.
- **refactor** - Scan for refactoring opportunities and technical debt.
- **data-validation** - Data validation framework for schema validation, data quality, constraints, and business rules (finance/AI focus).

### Copywriting Skills
- **copywrite** - Professional copywriting framework for hero sections, emails, UX microcopy, sales pages, SEO content, and brand tone guides.

### Design Skills
- **design** - Design system and component architecture guidance.
- **design-interactions** - Interaction design patterns and micro-interactions.
- **design-system** - Design system implementation and best practices.

### Utility Skills
- **logic-check** - Logic and reasoning validation.

## Installation

1. Clone this repository:
```bash
git clone https://github.com/TJM-NZ/claude-skills.git ~/claude-skills
```

2. Create symlinks in your Claude skills directory:
```bash
cd ~/.claude/skills
ln -s ~/claude-skills/technical-review technical-review
ln -s ~/claude-skills/copywrite copywrite
ln -s ~/claude-skills/data-validation data-validation
ln -s ~/claude-skills/design design
ln -s ~/claude-skills/design-interactions design-interactions
ln -s ~/claude-skills/design-system design-system
ln -s ~/claude-skills/geo-audit geo-audit
ln -s ~/claude-skills/perf-check perf-check
ln -s ~/claude-skills/refactor refactor
ln -s ~/claude-skills/security-audit security-audit
ln -s ~/claude-skills/seo-audit seo-audit
ln -s ~/claude-skills/logic-check.skill.md logic-check.skill.md
```

## Usage

Skills are invoked in Claude Code using the `/skill-name` syntax:

```bash
/technical-review  # Review your codebase
/security-audit    # Run security audit
/seo-audit         # Check SEO optimization
/data-validation   # Validate data quality and integrity
/copywrite         # Generate marketing copy
/perf-check        # Analyze performance
```

## Skill Structure

Each skill follows the Claude Code skill format:
- **SKILL.md** - Main skill definition with metadata and instructions
- **references/** - Optional subdirectory for modular sub-skills (e.g., seo-audit, copywrite)

## Contributing

This repository is public for reference, but contributions are limited. Feel free to fork and adapt for your own use.

## License

MIT License - See LICENSE file for details

## About

Created by [Teo McArthur](https://github.com/TJM-NZ) for use with Claude Code.
