---
tags: [redis, administration, security, acl]
---

# Security and ACLs

> [!summary] Summary
> Redis's security model has evolved from a single shared password to a full Access Control List (ACL) system supporting multiple users, per-command permissions, and per-key patterns — essential for any deployment beyond a fully trusted, isolated network.

## 1. Legacy authentication: requirepass

```
requirepass "a-strong-random-password"
```
```
AUTH a-strong-random-password
```

Simple but coarse: one shared password grants full access to everything — no way to give one application read-only access and another full access.

## 2. ACLs (Redis 6+)

ACLs replace the single-password model with **named users**, each with their own password(s), command permissions, and key-pattern permissions.

```
ACL SETUSER app_readonly on >mypassword ~cache:* +get +mget -@all
ACL SETUSER app_full on >otherpassword allkeys allcommands
```

| Directive | Meaning |
|---|---|
| `on` / `off` | enable/disable the user |
| `>password` | set a password (can specify multiple for rotation) |
| `~pattern` | grant access to keys matching a glob pattern (`allkeys` = all) |
| `+command` | allow a specific command |
| `-command` | deny a specific command |
| `+@category` | allow a whole command category (e.g., `+@read`, `+@write`, `+@dangerous`) |
| `nocommands` / `allcommands` | deny/allow everything as a baseline before adding specific overrides |

> [!example] Principle of least privilege in practice
> A reporting service that only needs to read cached data should get a user like `~report:* +@read -@dangerous`, never the full-access credentials used by the main application — limiting the blast radius if that service's credentials are ever compromised.

## 3. Dangerous commands

Certain commands are risky enough in production that many deployments explicitly restrict or rename them:

| Command | Risk |
|---|---|
| `FLUSHALL` / `FLUSHDB` | wipes all data instantly |
| `KEYS` | can block the event loop on a large keyspace (see [[Redis Architecture and Event Loop]]) |
| `CONFIG` | can change security-relevant settings at runtime |
| `SHUTDOWN` | stops the server |
| `DEBUG` | can expose internals or degrade performance |

The `@dangerous` ACL category groups many of these together for easy blanket restriction: `-@dangerous` on an application user, reserved only for admin users.

## 4. Network-level protections

- **`bind`**: restrict which network interfaces Redis listens on — never bind to a public interface without additional protections.
- **`protected-mode yes`** (default): refuses external connections when no password is set and no explicit `bind` is configured, guarding against the classic "Redis exposed to the internet with no auth" misconfiguration that has led to real-world breaches and cryptomining infections.
- **TLS**: Redis supports TLS-encrypted client connections and replication links (`tls-port`, certificate configuration) for traffic that traverses untrusted networks.
- **Firewalling**: in practice, Redis should typically sit behind a firewall/security group allowing access only from application servers, treating `requirepass`/ACLs as defense-in-depth rather than the sole perimeter.

## 5. Command renaming (legacy hardening technique)

```
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_9f8a2b"
```

Disables or obscures dangerous commands at the config level. Largely superseded by ACLs' more granular `+`/`-` command permissions, but still seen in older deployments and defense-in-depth setups.

## See also
- [[Configuration]]
- [[Monitoring and Observability]]

#redis #administration #security
