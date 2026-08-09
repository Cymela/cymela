# Security Policy

## Supported versions

Only the latest published version of Cymela receives fixes. Please reproduce
against the current release before reporting:

```bash
npm install -g cymela@latest
cymela --version
```

## Reporting a vulnerability

**Do not open a public issue for a security problem.**

Email **contact@cymela.com** with:

- what the problem is, and what an attacker gains from it
- the version (`cymela --version`), your OS and your terminal
- steps to reproduce, or a proof of concept
- anything you think we would get wrong on a first read

You will get an acknowledgement. Cymela is a small project with no security
team and no bug bounty, so please do not expect a same-day reply — but reports
are taken seriously and fixed in order of severity.

Please give us a reasonable chance to ship a fix before publishing details.

### Please redact before sending

Cymela handles provider API keys and reads your source. When attaching logs,
transcripts or session files, remove API keys, tokens and any code you are not
free to share. We do not want them and cannot store them safely on your behalf.

## What is in scope

- Anything that lets a repository you merely *open* run code, read credentials,
  or escape the permission prompts — the trust model around project settings,
  hooks and `CYMELA.md` is the area we care most about
- API keys leaking out of `~/.cymela/`, into a project directory, into a log, or
  to any host other than the provider you configured
- Bypasses of Plan mode, deny rules, or the permission engine
- Anything in the published npm package that behaves differently from what the
  documentation says it does

## What is out of scope

- The model doing something unhelpful, wrong, or unsafe when you approved it —
  Cymela asks before it acts, and an approved action is your decision
- Vulnerabilities in your chosen model provider's service or in third-party
  dependencies, unless Cymela's use of them is what creates the problem
- Missing hardening that has no demonstrated impact
- Reports produced by a scanner with no working reproduction
