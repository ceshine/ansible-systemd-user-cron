# AGENTS.md

## Purpose

This repository manages user-level `systemd` service and timer units with Ansible.

## Ansible via uvx

Use `uvx` to run Ansible commands without requiring a global install.

### Core commands

- Syntax check:
  - `uvx --from ansible-core ansible-playbook --syntax-check playbook.yml`
- Dry run:
  - `uvx --from ansible-core ansible-playbook --check --diff playbook.yml`
- Apply changes:
  - `uvx --from ansible-core ansible-playbook playbook.yml`

### Notes

- This playbook targets `localhost` with `connection: local`.
- Inventory warnings about implicit localhost are expected unless an explicit inventory is provided.

## Project conventions

- Keep variable data in `configs/*.yml` (for example, `configs/regular.yml`).
- Keep rendered unit definitions in Jinja2 templates under `templates/`.
- Keep orchestration and validation logic in `playbook.yml`.

## Ansible best practices

- Prefer `ansible.builtin.*` module names for clarity and portability.
- Keep tasks idempotent; repeated runs should converge with no drift.
- Validate user input early with `ansible.builtin.assert` before file generation.
- Use `ansible.builtin.template` for non-trivial file content; avoid large inline `content` blocks.
- Use handlers for daemon reloads and flush handlers before enabling units when needed.
- Avoid `shell`/`command` in tasks unless necessary; if shell behavior is required, make it explicit.
- Keep naming stable via `task_prefix` so orphan detection remains reliable.
- Prefer explicit timer fields (`on_calendar`, `on_boot_sec`, etc.) over ambiguous free-form values.

## Verification checklist

Before finalizing changes:

1. Run `--syntax-check`.
2. Run `--check --diff` and inspect proposed changes.
3. Confirm no unrelated files were modified.
