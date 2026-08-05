# Security policy

## Reporting a vulnerability

Please report security issues privately. Do not open a public GitHub issue.

Use GitHub's **private vulnerability reporting**: navigate to the Security tab of this repository and click **Report a vulnerability**. This routes the report directly to the maintainers without making it public.
Direct link: [Security tab](https://github.com/Obviously-Not/patent-continuation-public/security).

## Scope

continuation-drafter is a local command-line tool and local web UI. It reads a
specification and a set of parent claims from your filesystem and sends them to
an inference backend you choose: a local Ollama instance (nothing leaves your
machine) or a remote OpenAI-compatible endpoint you configure. There is no
service operated on your behalf, so the attack surface is the binary and the
files it touches.

In scope:

- Anything that sends specification text to a destination the operator did not
  configure, or that widens the local-only guarantee of the Ollama path
- Disclosure of the API key or license keys held in
  `~/.continuation-drafter/config.json`, including through logs, error text,
  process arguments, or a loosening of that file's 0600 permissions
- Vulnerabilities in the local web UI's server: request handling that reaches
  files outside the working set, or that lets another local user or a web page
  in the operator's browser drive the tool
- Flaws in offline license verification, such as accepting a token that was not
  signed by the embedded key, or verifying with an attacker-supplied key
- Path traversal or symlink escape when reading spec and claim files
- Dependency vulnerabilities that materially affect any of the above

Out of scope:

- Issues in third-party services the tool can be pointed at, including Ollama
  itself and any remote inference provider
- The quality, correctness, or completeness of drafted claims. Output is
  R&D-grade drafting input for a licensed practitioner, never filing-ready and
  never a legal determination; a weak draft is a product limitation, not a
  vulnerability
- That all capabilities are currently available regardless of license tier.
  This is a documented, deliberate state: verification and reporting are
  implemented and feature gating is deferred, so an unlocked feature is not a
  bypass
- Social engineering, physical access, and theoretical issues without a
  reproducer

## What to expect

We will acknowledge a valid report within 5 business days and provide a
remediation plan within 14 days for in-scope issues. Disclosure timing is
coordinated with the reporter.
