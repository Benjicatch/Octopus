# 🐙 Octopus

> Deploy a distributed voting application across 5 machines using Ansible — no containers allowed.

## Overview

**Octopus** is a DevOps project focused on **configuration management** with [Ansible](https://www.ansible.com/). The goal is to deploy the a poll application onto 5 separate virtual machines, configuring each one automatically and idempotently — without any containerization.

The poll application is composed of 4 interconnected services:

| Service | Technology | Role |
|---------|-----------|------|
| **poll** | Python / Flask | Web client that collects votes and pushes them to Redis |
| **redis** | Redis | Queue that stores incoming votes |
| **worker** | Java | Consumes votes from Redis and stores them in PostgreSQL |
| **postgresql** | PostgreSQL 16 | Persistent database for vote results |
| **result** | Node.js | Web client that reads from PostgreSQL and displays results |

## Features

### 6 Ansible Roles

- **`base`** — Applied to all machines. Installs essential packages and configures each instance properly.
- **`redis`** — Installs and configures the Redis queue.
- **`postgresql`** — Installs PostgreSQL 16, creates the `paul` user and database with the correct schema.
- **`poll`** — Uploads, installs dependencies, and runs the Flask poll client on port 80.
- **`worker`** — Uploads, builds, and runs the Java worker service.
- **`result`** — Uploads, installs dependencies, and runs the Node.js result client on port 80.

All services are managed by **systemd** and start automatically on boot. Configuration is passed via environment variables (host, port, user, password, database name) in line with [12-factor app](https://12factor.net/) best practices.

### Idempotence

Running the playbook multiple times produces no unintended changes.

### Security

All sensitive values (passwords, secrets) are encrypted with **Ansible Vault**. No clear-text passwords are present anywhere in the repository.

## Repository Structure

```
.
├── G-DOP-400_octopus.pdf
├── group_vars
│   └── all.yml
├── playbook.yml
├── poll.tar
├── README.md
├── result.tar
├── roles
│   ├── base
│   │   ├── files
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   ├── poll
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── poll.service.j2
│   ├── postgresql
│   │   ├── files
│   │   │   ├── pg_hba.conf
│   │   │   └── schema.sql
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   ├── redis
│   │   ├── files
│   │   │   └── redis.conf
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   ├── result
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── result.service.j2
│   └── worker
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── worker.service.j2
└── worker.tar

```

## Infrastructure

The inventory must define **5 groups**, each with one instance:

| Group | Instance |
|-------|----------|
| `redis` | `redis-1` |
| `postgresql` | `postgresql-1` |
| `poll` | `poll-1` |
| `result` | `result-1` |
| `worker` | `worker-1` |

All machines run **Debian 13**. A cloud provider is recommended.

## Usage

```bash
# Set up the Vault password
export ANSIBLE_VAULT_PASSWORD_FILE=/tmp/.vault_pass
echo verySecretPassword > /tmp/.vault_pass

# Run the playbook against your inventory
ansible-playbook -i production playbook.yml
```
