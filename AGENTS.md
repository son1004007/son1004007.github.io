# AGENTS.md

## Global AI Control

When GitHub access is available, before substantive work read `son1004007/ai-agent-workflow-playbook/CONTROL.md`, then return here and follow this repository's public-blog rules. The global control provides cross-repository discovery and shared verification rules. **This repository remains the source of truth for public blog structure, publication policy, and sanitized public content.**

## Repository Purpose

`son1004007/son1004007.github.io` is a GitHub Pages blog for two kinds of durable public records:

1. technical/work records that may also support a resume or portfolio;
2. Personal Notes about life, decisions, plans, learning, family activities, housing, devices, or other subjects worth keeping publicly.

Do **not** decide that a post is unnecessary merely because it is unrelated to IT or portfolio use. Instead, classify it correctly and keep technical/work content and Personal Notes separated in navigation and listing pages.

## Public Web Screenshot / Blogshot

When a credential-free public web screenshot would materially improve a post and GitHub access is available, do not repeatedly ask the owner to capture it manually.

Read the current private `son1004007/device-control/docs/BLOGSHOT.md` and use its verified `[blogshot]` Issue contract. That private document is authoritative for runtime behavior. Do not copy runner, SSH, NAS, credential, or other private infrastructure details into this public repository.

Current handoff principle:

```text
AI creates bounded [blogshot] request
  -> Synology captures and optimizes the image
  -> Synology stages the WebP directly in a blogshot/* branch of this repository
  -> AI receives branch/path/commit/SHA metadata
  -> AI reviews and merges the staged Git blob
```

Rules:

- Use automatic Blogshot only for credential-free public HTTP(S) pages. Do not use it for login-required pages, authenticated sessions, internal/LAN/admin pages, or private customer systems.
- `blogshot=READY` means the Synology runtime created, optimized, pushed, and remote-SHA-verified the image. It is **not** publication approval.
- Never download the Blogshot image and re-upload its binary through ChatGPT/Codex when the verified staged Git branch/blob already exists. Reuse the staged Git blob/branch directly.
- Before publication, inspect the actual image and confirm direct relevance, complete framing, low visual noise, readability, and safe public content. A technically successful but irrelevant or cropped capture must be rejected or recaptured.
- Prefer `selector` for a stable component, a deliberately sized `viewport` for a complete visible state, and `full-page` only when the entire page is meaningful.
- Do not add screenshots as decoration. Prefer text, code, Mermaid, or another visual form when it communicates the point better.
- Published images need meaningful `alt` text and, when useful, a caption explaining what the reader should notice.
- After merging, verify the GitHub Pages build and inspect the built/deployed result. For image changes, confirm the built image bytes/SHA and rendered readability rather than relying only on a successful workflow status.
- Never publish customer data, credentials, private/internal information, or unintended personal/session details visible in the pixels.
- If private `device-control` is unavailable, do not guess or reproduce its execution boundary. Continue without the screenshot when possible, or ask only for the minimum owner action that is actually required.

## Read Order

Before editing the blog, read as applicable:

1. `README.md`
2. `AGENTS.md`
3. `docs/category-guide.md`
4. `docs/blog-writing-guide.md`
5. `docs/post-template.md`
6. `_config.yml`
7. relevant recent `_posts/`

## Content Classification

### Technical / work posts

Use one of the seven standard categories:

| Category | Use |
|---|---|
| `backend` | Java, Spring Boot, API, authentication/authorization, web application structure |
| `database` | SQL, DB design, Oracle, PostgreSQL, Tibero, data modeling |
| `infrastructure` | Linux, Nginx, Apache, Tomcat, Podman/Docker, deployment, incident handling |
| `data-systemization` | connecting analysis results to DB/API/UI/operational systems |
| `security-audit` | security, access control, logging, internal control, CISA, ISMS-P |
| `project-management` | PMP, requirements, work logs, reporting, customer communication |
| `career` | career direction, portfolio, technical blogging, AI development workflow |

For technical/work posts, prefer a reusable problem-solving structure such as:

```text
problem -> cause -> decision/evidence -> solution -> execution -> verification -> prevention/improvement
```

### Personal Notes

Personal/life content is first-class blog content but should not be mixed into the technical portfolio feed.

- Add the `Personal` tag for new personal/life posts.
- Existing `Housing` posts are also treated as Personal Notes for compatibility.
- Do not invent `personal` or `housing` as technical categories merely to display these posts.
- Personal Notes may use a narrative, decision log, checklist, plan, comparison, or retrospective structure instead of the technical troubleshooting template.
- Follow `docs/category-guide.md` and keep Personal Notes under `/notes/` rather than the technical Home/Blog/Topics flow.
- Avoid unnecessarily detailed family, financial, account, location, or other private information even when the subject itself is appropriate to publish.

## Frontmatter

Technical post example:

```yaml
---
layout: post
title: "글 제목"
date: YYYY-MM-DD
categories: [infrastructure]
tags: [Linux, Troubleshooting]
---
```

Personal Notes should follow the repository's current Personal Notes policy in `docs/category-guide.md` and include the `Personal` tag where applicable.

Post filenames use:

```text
_posts/YYYY-MM-DD-kebab-case-title.md
```

## Writing and Evidence

- Prefer specific titles that explain the problem, decision, or useful outcome.
- Distinguish observed facts, tested results, assumptions, and recommendations.
- Do not publish AI output merely because it sounds plausible; verify claims that can be checked with code, tests, logs, Git history, official documentation, or the actual deployed result.
- ChatGPT/Codex conversation is working context, not automatically publishable source text. Convert useful outcomes into concise reusable records.
- Internal/customer-specific experience should be generalized without fabricating details.

## Public Safety Rules

Never publish:

- customer/internal IP addresses or private endpoints;
- usernames, passwords, tokens, keys, OAuth/session material, OTP/recovery information;
- non-public meeting minutes, contracts, proprietary customer documents, or exact internal paths that create access/security risk;
- customer identity directly tied to sensitive incident details unless explicitly public and appropriate;
- unnecessary personal data;
- proprietary certification exam questions, answers, or explanations.

For CISA work, the private `son1004007/cisa-playbook` repository remains the source of truth for application code and learner data. Publish only sanitized aggregate analysis; never copy proprietary question text, answers, rationales, credentials, private infrastructure details, or personal data.

## Do Not Do

- Do not revert to the theme's original sample README/content structure.
- Do not classify content as useless simply because it is not technical.
- Do not force Personal Notes into the technical category/feed structure.
- Do not expose private infrastructure or customer-identifying information.
- Do not paste raw ChatGPT conversations as finished posts.
- Do not exaggerate experience or claim verification that was not performed.
- Do not re-upload Blogshot binaries through AI when Synology has already provided a verified staged Git blob.

## Done Criteria

A change is complete only when the criteria relevant to that change are satisfied:

1. Frontmatter and file naming are valid for posts.
2. Technical/work posts use the standard technical classification; Personal Notes follow the Personal Notes policy instead of being rejected for non-technical subject matter.
3. Public/private boundaries are checked.
4. Technical claims are supported by appropriate evidence or clearly identified as assumptions/recommendations.
5. If an image is used, its actual pixels were inspected for relevance, complete framing, readability, noise, and public safety.
6. Blogshot images use the Synology-staged Git blob/branch directly; binary handoff through AI is not part of the normal workflow.
7. GitHub Pages build/deploy succeeds when the rendered site is affected.
8. The final built/deployed page is checked at the level needed to catch broken images, bad paths, cropping, unreadable scale, or mismatch with the surrounding text.
