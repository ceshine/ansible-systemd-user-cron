# ansible-systemd-user-cron

Manage user-level `systemd` timers/services on localhost with Ansible.

## Run

- Syntax check: `uvx --from ansible-core ansible-playbook --syntax-check playbook.yml`
- Dry run: `uvx --from ansible-core ansible-playbook --check --diff playbook.yml`
- Apply: `uvx --from ansible-core ansible-playbook playbook.yml`

## Config

- Edit `configs/regular.yml`:
  - `task_prefix`
  - `scheduled_tasks`

### Alternative or multiple config files

- Use one alternative file:
  - `uvx --from ansible-core ansible-playbook -e @configs/work.yml playbook.yml`
- Layer multiple files (later `-e` wins for overlapping keys):
  - `uvx --from ansible-core ansible-playbook -e @configs/base.yml -e @configs/work.yml playbook.yml`
- Combine files with direct overrides:
  - `uvx --from ansible-core ansible-playbook -e @configs/work.yml -e 'task_prefix=dev-' playbook.yml`

## Templates

- Service unit template: `templates/service.j2`
- Timer unit template: `templates/timer.j2`

`playbook.yml` renders those templates into `~/.config/systemd/user/` and enables the timers.
