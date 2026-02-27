# Ansible-based Management of User Systemd Periodic Tasks 

Replace manually managed user-level `systemd` timers and services with a structured, config-driven workflow that is friendly to AI agents. 

This repository lets you declare all periodic tasks in YAML files, render them into `systemd` unit files, and deploy them via Ansible. It makes auditing, versioning, and automation of user-level scheduling much simpler in environments that already rely on Ansible.

## Installation

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd ansible-systemd-user-cron
   ```

2. **Ensure `uvx` is installed:**
   - `uvx` is used to run Ansible without a global installation
   - Install from [astral-sh/uv](https://github.com/astral-sh/uv) if needed

3. **Set up your configuration:**
   - Copy `configs/example.yml` to `configs/regular.yml` (the default config file path)
   - Edit `configs/regular.yml` with your tasks and settings
   - See the [Config](#config) section for more details

## Running the Playbook

- Syntax check: `uvx --from ansible-core ansible-playbook --syntax-check playbook.yml`
- Dry run: `uvx --from ansible-core ansible-playbook --check --diff playbook.yml`
- Refresh the systemd units: `uvx --from ansible-core ansible-playbook playbook.yml`

## Configuring Scheduled Tasks

Configuration is driven by YAML files located in `configs/`. You can copy [configs/example.yml](./configs/example.yml) to get started.

- Default config file path: `configs/regular.yml`
- Override with `-e @path/to/config.yml`

### Top-level Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `task_prefix` | string | **Yes** | Prefix for generated systemd unit files (e.g., `my-cron-`). |
| `scheduled_tasks` | list | **Yes** | List of task definitions. |

### Task Definition

Each item in `scheduled_tasks` supports the following:

#### General

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `name` | string | **Yes** | Unique identifier for the task (appended to `task_prefix`). |
| `command` | string | **Yes** | The command to execute. |
| `working_directory` | string | No | Directory to execute the command in. |
| `use_shell` | boolean | No | If `true`, runs command in a shell (`/bin/bash -lc`). Default: `false`. |

#### Scheduling

You must provide either `schedule` OR `timer` (but not both).

**Option A: Simple Schedule**

| Field | Type | Description |
| :--- | :--- | :--- |
| `schedule` | string | Systemd time format (e.g., `daily`, `*-*-* 04:00:00`). Maps to `OnCalendar`. |

**Option B: Advanced Timer**

Use the `timer` dictionary for more granular control. At least one trigger (e.g., `on_calendar`, `on_boot_sec`) must be defined.

| Field | Type | Description |
| :--- | :--- | :--- |
| `timer.on_calendar` | string | Calendar event expression (e.g., `Mon..Fri 09:00`). |
| `timer.on_boot_sec` | string/int | Seconds after boot to run. |
| `timer.on_unit_active_sec` | string/int | Seconds after last activation to run. |
| `timer.on_unit_inactive_sec` | string/int | Seconds after last deactivation to run. |
| `timer.randomized_delay_sec` | int | Delay execution by random seconds (0 to value). Default: `30`. |
| `timer.persistent` | boolean | If `true`, run immediately if missed during downtime. Default: `true`. |

### Alternative or multiple config files

- Use one alternative file:
  - `uvx --from ansible-core ansible-playbook -e @configs/work.yml playbook.yml`
- Layer multiple files (later `-e` wins for overlapping keys):
  - `uvx --from ansible-core ansible-playbook -e @configs/base.yml -e @configs/work.yml playbook.yml`
- Combine files with direct overrides:
  - `uvx --from ansible-core ansible-playbook -e @configs/work.yml -e 'task_prefix=dev-' playbook.yml`

## Implementation Details

### Templates

- Service unit template: `templates/service.j2`
- Timer unit template: `templates/timer.j2`

`playbook.yml` renders those templates into `~/.config/systemd/user/` and enables the timers.
