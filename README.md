# askpc.cc — Knowledge Base

Documentation for [askpc.cc](https://askpc.cc) — a technical knowledge base and forum about computers and software.

Content here is automatically pulled to the site on push to `main` (via webhook).

## Who can contribute

Anyone. This is a community-driven knowledge base: from beginners to pros, Windows to FreeBSD.

## Languages

The site supports **only two languages**: English and Russian. Articles in other languages will not be accepted.

## Article format

Each article is a plain `.md` file with YAML frontmatter at the top:

```markdown
---
title: Setting up Zapret on Arch Linux
tags: [network, censorship, arch]
lang: en
---

Article body in regular markdown.
```

### Frontmatter fields

| Field   | Required | Description                          |
|---------|----------|----------------------------------------|
| `title` | yes      | Article title                          |
| `tags`  | yes      | List of tags in square brackets        |
| `lang`  | yes      | `en` or `ru`                           |

## Content guidelines

- Computers and software only. No politics.
- Get to the point — this is a knowledge base, not a blog.
- Make sure the information is accurate at time of publishing.
- Don't use Obsidian-specific syntax (`[[wiki-links]]`, `![[embeds]]`) — the site renders plain markdown, these won't work.
- Wrap code in fenced blocks with the language specified (```` ```bash ````, ```` ```python ````, etc.)

## How to submit an article

1. Fork the repository
2. Create a `.md` file in the appropriate directory
3. Open a Pull Request with a short description of what you're adding/changing
4. Wait for moderation review — checked for accuracy, topic fit, and language
5. Once approved, it's merged and appears on the site via webhook

## License

CC BY-SA 4.0
