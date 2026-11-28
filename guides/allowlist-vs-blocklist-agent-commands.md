# Allowlist vs Blocklist for What Your Agent Can Run

In PCI DSS, a good firewall rule set ends with "deny all". The same thinking works for a coding agent that may only run tests and `git status`.

## Side by side

| | Blocklist | Allowlist |
|---|---|---|
| Default | Allow everything | Deny everything |
| You write | Every bad command you can think of | The few commands you expect |
| New bad command | Gets through | Blocked |
| Small variation (`rm -rf` vs `rm -fr`) | Easy to miss | Not on the list, so blocked |
| Maintenance | Never ends | Short and easy to read |
| Failure mode | Silent | Agent asks, you decide |

## Why blocklists fail

- Same action can be written many ways, so one pattern rarely covers all of them
- The list grows every week and is never finished
- Wrapping a command in `bash -c` or a script hides it
- You only block what you already know about

## How the check runs

- Agent wants to run a command
- Exact match on the allowlist means it runs
- No match means it asks a human, and the default answer is no

## What a good allowlist looks like

```json
{
  "allow": [
    "pytest",
    "git status"
  ],
  "default": "ask"
}
```

- **Exact commands**, not wildcards like `git *`
- **Unknown command**: ask a human, never run silently
- **No shell wrappers** in the list

## The saved approval trap

Agents offer "always allow this command". If the command has a password in it, the password now sits in a settings file.

| Saved approval | Problem | Better |
|---|---|---|
| `psql postgres://admin:Pass123@db/prod` | Password stored in plain text | `psql` using an env variable or a secret manager |
| `curl -H "Authorization: Bearer sk-..."` | Token in settings and history | Read token from environment |
| `git *` | Allows push, reset, anything | `git status` |

## Decisions and trade-offs

| Decision | Why | Trade-off |
|---|---|---|
| Deny by default | Unknown things cannot run | More prompts at the start |
| Exact commands only | Small attack surface | Longer list |
| Review saved approvals monthly | Old approvals pile up | Takes some time |

## When to use it (and when not to)

- Use it when the agent can touch a shell, a database or a repo
- Use it when real credentials exist on the machine
- Use it when the agent reads files or web pages you do not control, since those can carry hidden instructions
- Skip strict lists for a throwaway sandbox with no secrets
- Skip it if nobody will ever review the prompts, since people then click yes to everything

## Checklist

- [ ] Agent default is deny or ask
- [ ] Only needed commands are listed
- [ ] No wildcards on git, shell or database tools
- [ ] No password or token inside any saved command
- [ ] Saved approvals reviewed on a schedule
- [ ] Secrets live in environment or a vault

## My takeaway

Basically, list what is allowed and deny the rest, the same way a firewall rule set ends. And never let an approved command carry a password inside it.
