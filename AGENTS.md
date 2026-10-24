# AGENTS.md

Ansible role that installs and configures the BIND9 nameserver on Debian (bookworm, trixie).

## Layout

- `defaults/main.yml`: all role variables with defaults and examples
- `meta/argument_specs.yml`: variable validation; keep in sync with `defaults/main.yml`
- `tasks/`: `main.yml` and `create_ddns_keys.yml` (DDNS TSIG key generation)
- `templates/bind/`: `named.conf.options`, `named.conf.local`, dynamic zone template
- `molecule/default/`: integration test (Docker, Debian 13)

## Checks

- `ansible-lint .` must pass without warnings
- `molecule test` runs the role in a container; CI runs it on Debian 12 and 13
- `ansible-lint` creates `.ansible/`; it is git ignored, never commit it

## Conventions

- Use fully qualified module names, quoted octal `mode` values and `true`/`false` booleans
- Give every task a `name`, and loops a `loop_control.label`
- Mark read-only tasks with `changed_when: false` and `check_mode: false`
- Use BIND terminology primary/secondary, not master/slave (the older names are removed)
- New or changed role variables need an entry in `defaults/main.yml`, `meta/argument_specs.yml` and the README

## Commits and PRs

- Write in English, use conventional commit titles (`feat:`, `fix:`, `chore:`, `ci:`; `!` for breaking changes)
- The PR title sets the release-drafter label (`enhancement`, `bug`, `chore`, `breaking`); a breaking change bumps the major version
- Open PRs as draft and fill in `.github/pull_request_template.md`: what changed and why, plus a breaking changes section if needed
- Do not list changed files or claim test results in the description; CI covers that
