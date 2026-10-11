# 🅰️ TOPIC: ANSIBLE

> Architecture, inventory, playbooks & roles, modules, idempotency, handlers, variables, Vault, templates, rolling deployments, troubleshooting and AWX.

<br>

---

### Q1. What is Ansible and how does it work? Why is it called agentless?

#### 📌 Short Explanation
The opening Ansible question. Interviewers check your understanding of its architecture and why teams adopt it.

#### ✅ Answer
**Ansible** is an open-source automation tool for configuration management, application deployment, and orchestration. It is **agentless**: the control node connects to managed nodes over **SSH** (Linux) or **WinRM** (Windows) and runs small modules there, so nothing needs to be installed on the targets (apart from Python on Linux for most modules).

#### 📖 Detailed Explanation

**🔹 Architecture**
```text
Control Node (ansible, playbooks, inventory)
      │  SSH / WinRM (push)
      ▼
Managed Nodes (web1, web2, db1 ...)  ← modules copied, executed, removed
```

**🔹 Key components**

| Component | Description |
|-----------|-------------|
| **Control node** | Machine where Ansible runs |
| **Inventory** | List of managed hosts and groups |
| **Playbook** | YAML file describing desired state |
| **Module** | Unit of work (`apt`, `service`, `copy`) |
| **Role** | Reusable, structured collection of tasks/vars/templates |
| **Facts** | Auto-gathered system information (`ansible_os_family`) |

**🔹 Why teams like it**
- No agents to install or maintain · simple YAML · **idempotent** modules · large module ecosystem · works for servers, network devices, and cloud APIs

**🔹 Ad-hoc vs playbook**
```bash
ansible all -i inventory.ini -m ping                              # ad-hoc
ansible webservers -m apt -a "name=nginx state=present" -b        # ad-hoc with sudo
ansible-playbook -i inventory.ini site.yml                        # playbook
```

<br>

---

### Q2. What is the difference between Ansible and Terraform? Can they be used together?

#### 📌 Short Explanation
A very common comparison question. Tests whether you know provisioning vs configuration management.

#### ✅ Answer
- **Terraform** — **provisions infrastructure** (VPC, EC2, RDS) declaratively and tracks it in **state**.
- **Ansible** — **configures** what runs on servers (packages, files, services) and orchestrates deployments; it is **stateless**.

They complement each other: Terraform creates the servers, Ansible configures them.

#### 📖 Detailed Explanation

| Aspect | Terraform | Ansible |
|--------|-----------|---------|
| **Primary purpose** | Infrastructure provisioning | Configuration management and orchestration |
| **Approach** | Declarative | Mostly declarative modules, procedural task order |
| **State** | Maintains a state file | No state (checks the current system each run) |
| **Language** | HCL | YAML |
| **Agent** | None (API calls) | None (SSH/WinRM) |
| **Strength** | Lifecycle of cloud resources, dependency graph | Server setup, app deployment, rolling updates |
| **Immutable vs mutable** | Favors immutable infrastructure | Typically mutable servers |

**🔹 Combined workflow**
1. Terraform creates EC2/VPC/RDS and outputs IPs or tags
2. Ansible uses a **dynamic inventory** (`aws_ec2` plugin) to discover instances by tag
3. Ansible installs packages, deploys configs, and starts services

> 💡 Many teams reduce Ansible's role by baking **golden AMIs** (Packer, with Ansible as the provisioner) and using containers.

<br>

---

### Q3. What is the difference between Ansible, Chef, and Puppet?

#### 📌 Short Explanation
Tests your awareness of the configuration management landscape.

#### ✅ Answer
Ansible is **agentless and push-based** using YAML. Chef and Puppet are traditionally **agent-based and pull-based**, using Ruby-based DSLs, with a master server.

#### 📖 Detailed Explanation

| Feature | Ansible | Puppet | Chef |
|---------|---------|--------|------|
| **Architecture** | Agentless (SSH/WinRM) | Agent + Puppet Server | Agent (chef-client) + Chef Server |
| **Model** | Push (pull possible with `ansible-pull`) | Pull | Pull |
| **Language** | YAML | Puppet DSL | Ruby DSL |
| **Learning curve** | Easy | Moderate | Steeper |
| **Setup effort** | Minimal | Server + agents | Server + agents |
| **Best for** | Quick adoption, orchestration, ad-hoc tasks | Large, long-lived fleets with strict drift control | Developer-centric, code-heavy environments |

> **Key takeaway:** Ansible wins on simplicity and low overhead; pull-based tools constantly enforce drift correction on large fleets.

<br>

---

### Q4. Explain Ansible Inventory. What is the difference between static and dynamic inventory?

#### 📌 Short Explanation
Tests how Ansible knows which hosts to manage, especially in cloud environments where IPs change.

#### ✅ Answer
- **Static inventory** — a manually maintained file listing hosts/groups (INI or YAML).
- **Dynamic inventory** — generated automatically from a source (AWS EC2, Azure, GCP, Kubernetes) using plugins.

#### 📖 Detailed Explanation

**🔹 Static inventory (`inventory.ini`)**
```ini
[webservers]
web1 ansible_host=10.0.1.10
web2 ansible_host=10.0.1.11

[dbservers]
db1 ansible_host=10.0.2.10 ansible_user=ubuntu

[prod:children]
webservers
dbservers

[prod:vars]
ansible_ssh_private_key_file=~/.ssh/prod.pem
```

**🔹 Dynamic inventory (AWS) — `aws_ec2.yml`**
```yaml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1
filters:
  tag:Environment: prod
keyed_groups:
  - key: tags.Role
    prefix: role
```
```bash
ansible-inventory -i aws_ec2.yml --graph
ansible-playbook -i aws_ec2.yml site.yml
```

**🔹 Comparison**

| Aspect | Static | Dynamic |
|--------|--------|---------|
| **Maintenance** | Manual | Automatic |
| **Best for** | Small, stable environments | Auto Scaling, cloud, containers |
| **Grouping** | Manual groups | Tags/attributes → groups |

**🔹 Variables:** place per-group/host values in `group_vars/` and `host_vars/` directories.

<br>

---

### Q5. Explain the structure of an Ansible playbook and roles. How does Ansible Galaxy help?

#### 📌 Short Explanation
Tests your ability to write clean, reusable automation.

#### ✅ Answer
A **playbook** contains one or more **plays**; each play maps **hosts** to **tasks**. **Roles** package tasks, handlers, templates, files, and variables into reusable units, and **Galaxy** is the hub for sharing roles/collections.

#### 📖 Detailed Explanation

**🔹 Simple playbook**
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars:
    http_port: 80

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Ensure nginx is running and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true
```

**🔹 Role structure (`ansible-galaxy init nginx`)**
```text
roles/nginx/
├── tasks/main.yml
├── handlers/main.yml
├── templates/nginx.conf.j2
├── files/
├── vars/main.yml          # high precedence
├── defaults/main.yml      # lowest precedence (easy to override)
└── meta/main.yml          # dependencies
```

**🔹 Using a role**
```yaml
- hosts: webservers
  roles:
    - role: nginx
      vars:
        http_port: 8080
```

**🔹 Galaxy and collections**
```bash
ansible-galaxy install geerlingguy.docker
ansible-galaxy collection install amazon.aws
ansible-galaxy install -r requirements.yml
```

**🔹 Best practices:** one role per concern, use FQCN module names (`ansible.builtin.copy`), pin role/collection versions in `requirements.yml`, lint with **ansible-lint**.

<br>

---

### Q6. What is the difference between the `command`, `shell`, `raw`, and `script` modules?

#### 📌 Short Explanation
A frequently asked modules question that tests safe automation practices.

#### ✅ Answer
Prefer purpose-built modules (`apt`, `service`, `copy`) first. If you must run commands: `command` (safe, no shell), `shell` (uses shell features), `raw` (no Python needed), `script` (run a local script remotely).

#### 📖 Detailed Explanation

| Module | Uses shell? | Needs Python on target? | Pipes / redirects / env vars | Use case |
|--------|-------------|-------------------------|------------------------------|----------|
| `command` | ❌ No | ✅ Yes | ❌ Not supported | Simple commands (default and safer) |
| `shell` | ✅ Yes (`/bin/sh`) | ✅ Yes | ✅ Supported | Pipes, `&&`, redirects |
| `raw` | Direct SSH | ❌ No | ✅ | Bootstrap hosts without Python, network devices |
| `script` | Runs a local script on remote | ❌ Not for execution | ✅ (inside script) | Run existing scripts |

**🔹 Examples**
```yaml
- name: Run a simple command
  ansible.builtin.command: uptime

- name: Use a pipe (needs shell)
  ansible.builtin.shell: "ps aux | grep nginx | wc -l"

- name: Install Python on a bare host
  ansible.builtin.raw: apt-get install -y python3
```

**🔹 Important:** `command`/`shell` are **not idempotent** by default — use `creates:`, `removes:`, `changed_when`, and `when` to make them safe.

<br>

---

### Q7. What is idempotency in Ansible? Scenario: a playbook always shows "changed" — how do you fix it?

#### 📌 Short Explanation
Idempotency is Ansible's core principle. Tests how well you write safe, repeatable automation.

#### ✅ Answer
An operation is **idempotent** if running it multiple times produces the same result as running it once. Ansible modules check the current state and only change what is needed. Tasks using `command`/`shell` need explicit conditions to behave idempotently.

#### 📖 Detailed Explanation

**🔹 Idempotent vs non-idempotent**
```yaml
# ✅ Idempotent — no change on second run
- ansible.builtin.apt:
    name: nginx
    state: present

# ❌ Not idempotent — appends every time
- ansible.builtin.shell: echo "export PATH=/opt/bin:$PATH" >> /etc/profile
```

**🔹 Fix with the right module**
```yaml
- ansible.builtin.lineinfile:
    path: /etc/profile
    line: 'export PATH=/opt/bin:$PATH'
```

**🔹 Fix a command task with `creates` / `changed_when`**
```yaml
- name: Initialize database once
  ansible.builtin.command: /opt/app/init_db.sh
  args:
    creates: /var/lib/app/.initialized

- name: Check config validity (read-only)
  ansible.builtin.command: nginx -t
  register: nginx_test
  changed_when: false
```

**🔹 Verify idempotency**
```bash
ansible-playbook site.yml           # first run: changed
ansible-playbook site.yml           # second run should show changed=0
ansible-playbook site.yml --check --diff   # dry run
```

<br>

---

### Q8. What are Handlers? Scenario: restart nginx only when its configuration changes.

#### 📌 Short Explanation
Tests event-driven task execution and avoiding unnecessary restarts.

#### ✅ Answer
**Handlers** are tasks that run **only when notified** by another task that reports `changed`. They run **once at the end of the play** even if notified multiple times.

#### 📖 Detailed Explanation

**🔹 Example**
```yaml
- name: Configure nginx
  hosts: webservers
  become: true
  tasks:
    - name: Deploy nginx config
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
        validate: nginx -t -c %s
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

**🔹 Behavior**
- If the template didn't change → handler **does not run** (no needless restart)
- Multiple notifications → handler runs **once**
- Handlers run at the end of the play; force earlier execution with `meta: flush_handlers`
- If a task fails, pending handlers don't run unless `--force-handlers`

**🔹 Use `reload` over `restart`** for services that support it, to avoid downtime.

<br>

---

### Q9. What is the order of variable precedence in Ansible? What is the difference between `vars`, `group_vars`, `defaults`, and `extra-vars`?

#### 📌 Short Explanation
Tests your understanding of where variables can live and which one wins.

#### ✅ Answer
Ansible has many variable sources. In practice, remember: **role `defaults` are the lowest**, inventory and `group_vars`/`host_vars` are in the middle, play/task `vars` are higher, and **`--extra-vars` (`-e`) always wins**.

#### 📖 Detailed Explanation

**🔹 Simplified precedence (low → high)**
1. Role `defaults/main.yml`
2. Inventory `group_vars` / `host_vars`
3. Facts and registered variables
4. Play `vars`, `vars_files`
5. Role `vars/main.yml`
6. Task `vars`, `include_vars`, `set_fact`
7. **Extra vars (`-e`)** — highest

| Location | Typical use |
|----------|-------------|
| `defaults/` | Safe overridable role defaults |
| `group_vars/<group>.yml` | Environment-level values (prod, staging) |
| `host_vars/<host>.yml` | Host-specific overrides |
| `vars:` in play | Playbook-specific values |
| `-e` | One-off overrides from CLI/CI |

```bash
ansible-playbook deploy.yml -e "app_version=2.4.1 env=prod"
```

**🔹 Facts vs variables:** facts come from the `setup` module (`ansible_distribution`, `ansible_default_ipv4.address`); disable with `gather_facts: false` to speed up large runs.

<br>

---

### Q10. What is Ansible Vault? Scenario: how do you keep secrets out of Git?

#### 📌 Short Explanation
Tests secure handling of sensitive data in automation.

#### ✅ Answer
**Ansible Vault** encrypts variables or whole files (AES-256) so secrets can be safely stored in version control and decrypted at runtime with a password or password file.

#### 📖 Detailed Explanation

**🔹 Commands**
```bash
ansible-vault create group_vars/prod/vault.yml
ansible-vault encrypt secrets.yml
ansible-vault edit group_vars/prod/vault.yml
ansible-vault view secrets.yml
ansible-vault rekey secrets.yml
ansible-vault encrypt_string 'S3cr3t!' --name 'db_password'
```

**🔹 Running with Vault**
```bash
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
ansible-playbook site.yml --vault-id prod@prompt
```

**🔹 Best-practice layout**
```text
group_vars/prod/
├── vars.yml      # db_password: "{{ vault_db_password }}"
└── vault.yml     # encrypted: vault_db_password: ...
```
This keeps variable names visible/greppable while values stay encrypted.

**🔹 Best practices**
- Never commit the vault password file; store it in a **CI secret store**
- Use **separate vault IDs** per environment
- Use `no_log: true` on tasks handling secrets to avoid leaking them in logs
- For enterprise setups, integrate with **HashiCorp Vault / AWS Secrets Manager** lookups

<br>

---

### Q11. What is the difference between the `template` and `copy` modules? What is Jinja2?

#### 📌 Short Explanation
Tests how you manage configuration files across environments.

#### ✅ Answer
- **`copy`** copies a file **as-is**.
- **`template`** renders a **Jinja2** template (with variables, loops, conditionals) and then copies the result.

#### 📖 Detailed Explanation

| Aspect | `copy` | `template` |
|--------|--------|-----------|
| **Content** | Static | Dynamic (variables processed) |
| **Source location** | `files/` | `templates/` (`.j2`) |
| **Use case** | Binaries, static configs | Configs that differ per host/env |

**🔹 Template example — `nginx.conf.j2`**
```jinja
server {
    listen {{ http_port | default(80) }};
    server_name {{ inventory_hostname }};

{% for app in apps %}
    location /{{ app.name }} {
        proxy_pass http://127.0.0.1:{{ app.port }};
    }
{% endfor %}
}
```

**🔹 Task**
```yaml
- ansible.builtin.template:
    src: nginx.conf.j2
    dest: /etc/nginx/conf.d/app.conf
    owner: root
    mode: '0644'
    backup: true
  notify: Reload nginx
```

**🔹 Common Jinja2 filters:** `default()`, `join`, `upper`, `to_json`, `regex_replace`, `int`.

<br>

---

### Q12. How do you use conditionals, loops, `register`, and `block/rescue` in Ansible?

#### 📌 Short Explanation
Tests control-flow skills for real playbooks.

#### ✅ Answer
Use **`when`** for conditions, **`loop`** for iteration, **`register`** to capture task results, and **`block/rescue/always`** for error handling.

#### 📖 Detailed Explanation

**🔹 `when` and facts**
```yaml
- name: Install Apache on Debian family
  ansible.builtin.apt:
    name: apache2
  when: ansible_os_family == "Debian"
```

**🔹 `loop`**
```yaml
- name: Create users
  ansible.builtin.user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
  loop:
    - { name: alice, groups: dev }
    - { name: bob,   groups: ops }
```

**🔹 `register` and use the result**
```yaml
- name: Check if app is listening
  ansible.builtin.command: ss -ltn sport = :8080
  register: port_check
  changed_when: false
  failed_when: "'8080' not in port_check.stdout"
```

**🔹 `block / rescue / always`**
```yaml
- block:
    - name: Deploy new version
      ansible.builtin.command: /opt/deploy.sh {{ version }}
  rescue:
    - name: Roll back
      ansible.builtin.command: /opt/rollback.sh
  always:
    - name: Send notification
      ansible.builtin.debug:
        msg: "Deployment finished (success or rollback)"
```

**🔹 Other useful keywords:** `ignore_errors`, `failed_when`, `changed_when`, `until/retries/delay`, `tags`, `delegate_to`, `run_once`.

<br>

---

### Q13. Scenario: Deploy an application to 100 web servers behind a load balancer without downtime. How do you do it with Ansible?

#### 📌 Short Explanation
A classic real-world scenario. Tests rolling updates, safety controls, and orchestration.

#### ✅ Answer
Use **`serial`** to update in small batches, **deregister each server from the load balancer**, deploy, run health checks, **re-register**, and stop on failure with **`max_fail_percentage`**.

#### 📖 Detailed Explanation

**🔹 Rolling update playbook**
```yaml
- name: Rolling deployment
  hosts: webservers
  become: true
  serial: "10%"                 # or 5, or [1, 5, "25%"] for canary-style batches
  max_fail_percentage: 0        # stop immediately on any failure

  pre_tasks:
    - name: Remove from load balancer
      community.aws.elb_target:
        target_group_arn: "{{ tg_arn }}"
        target_id: "{{ instance_id }}"
        state: absent
      delegate_to: localhost

  tasks:
    - name: Deploy application
      ansible.builtin.unarchive:
        src: "app-{{ version }}.tar.gz"
        dest: /opt/app
      notify: Restart app

    - name: Flush handlers so restart happens before health check
      ansible.builtin.meta: flush_handlers

    - name: Wait for health check
      ansible.builtin.uri:
        url: "http://localhost:8080/health"
        status_code: 200
      register: result
      until: result.status == 200
      retries: 10
      delay: 5

  post_tasks:
    - name: Add back to load balancer
      community.aws.elb_target:
        target_group_arn: "{{ tg_arn }}"
        target_id: "{{ instance_id }}"
        state: present
      delegate_to: localhost

  handlers:
    - name: Restart app
      ansible.builtin.service:
        name: myapp
        state: restarted
```

**🔹 Key controls**

| Feature | Purpose |
|---------|---------|
| `serial` | Batch size for rolling updates |
| `max_fail_percentage` | Abort when too many hosts fail |
| `delegate_to: localhost` | Run LB/API tasks from the control node |
| Health check with `until` | Don't proceed until the app is healthy |
| `run_once` | Execute a task once per play (e.g., DB migration) |

<br>

---

### Q14. Scenario: Ansible shows `UNREACHABLE!` for some hosts. How do you troubleshoot?

#### 📌 Short Explanation
Connection problems are the most common Ansible failure. Tests systematic troubleshooting.

#### ✅ Answer
Verify **network reachability, SSH access, user/key, inventory variables, and Python availability** — using `-vvv` for detailed connection output.

#### 📖 Detailed Explanation

**🔹 Typical error**
```text
web1 | UNREACHABLE! => {"msg": "Failed to connect to the host via ssh: Permission denied (publickey)."}
```

**🔹 Steps**
1. **Test the connection manually**
```bash
ssh -i ~/.ssh/key.pem ubuntu@10.0.1.10
ansible web1 -m ping -vvv
```
2. **Network:** Security Group / firewall allows port 22 from the control node; host is up; correct IP in inventory
3. **Authentication:** correct `ansible_user`, key file permissions (`chmod 400`), correct `ansible_ssh_private_key_file`, key present in `authorized_keys`
4. **Host key issues:** first connection prompts — set `host_key_checking` appropriately (understand the security trade-off), or pre-populate `known_hosts`
5. **Python missing:** `"/usr/bin/python: not found"` → set `ansible_python_interpreter=/usr/bin/python3` or bootstrap using the `raw` module
6. **Privilege escalation:** `become` failing → check sudoers and `become_method`
7. **Bastion/jump host:**
```ini
ansible_ssh_common_args='-o ProxyJump=ec2-user@bastion.example.com'
```
8. **Timeouts:** raise `timeout` in `ansible.cfg` or tune `ControlPersist` / `pipelining` for performance

| Error | Likely cause |
|-------|--------------|
| `Permission denied (publickey)` | Wrong user/key |
| `Connection timed out` | Firewall/SG/network |
| `Connection refused` | SSH service not running |
| `Missing sudo password` | `become` needs a password or NOPASSWD |

<br>

---

### Q15. Scenario: How do you safely test and run a playbook against a subset of hosts or specific tasks?

#### 📌 Short Explanation
Tests operational safety — essential for production automation.

#### ✅ Answer
Use **`--check --diff`** (dry run), **`--limit`** (host subset), **`--tags`** (specific tasks), and **`--start-at-task`/`--step`** for controlled execution.

#### 📖 Detailed Explanation

```bash
# Dry run: show what WOULD change
ansible-playbook site.yml --check --diff

# Only one group or host
ansible-playbook site.yml --limit web1
ansible-playbook site.yml --limit 'webservers:!web3'

# Only tagged tasks
ansible-playbook site.yml --tags "nginx,config"
ansible-playbook site.yml --skip-tags "slow"

# Validate syntax and lint
ansible-playbook site.yml --syntax-check
ansible-lint site.yml

# List what will run
ansible-playbook site.yml --list-hosts --list-tasks

# Step through interactively / resume
ansible-playbook site.yml --step
ansible-playbook site.yml --start-at-task "Deploy application"
```

**🔹 Tagging tasks**
```yaml
- name: Deploy config
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/app.conf
  tags: [config]
```

**🔹 Notes:** `--check` isn't perfect for `command`/`shell` tasks (they're skipped unless `check_mode: false`); test in **staging** and use **Molecule** to test roles.

<br>

---

### Q16. What are Ansible Tower / AWX, and what is the difference between push mode and `ansible-pull`?

#### 📌 Short Explanation
Tests enterprise usage and execution models.

#### ✅ Answer
**AWX** (upstream, open source) and **Red Hat Ansible Automation Platform / Tower** provide a **web UI, REST API, RBAC, scheduling, credentials management, and audit logs** on top of Ansible. **`ansible-pull`** flips the model: nodes pull playbooks from Git and apply them locally.

#### 📖 Detailed Explanation

**🔹 AWX / Tower features**
- Centralized **credential storage** (users never see SSH keys/passwords)
- **Job templates**, workflows, scheduling, notifications
- **RBAC**, audit trails, REST API for CI/CD integration
- Dynamic inventory sync, surveys for runtime inputs

**🔹 Push vs Pull**

| Aspect | Push (`ansible-playbook`) | Pull (`ansible-pull`) |
|--------|--------------------------|----------------------|
| **Initiated by** | Control node | Each managed node |
| **Scales to** | Hundreds (use forks) | Very large or unreachable/auto-scaling nodes |
| **Network** | Control node needs SSH access to targets | Nodes need access to Git only |
| **Use case** | Deployments, orchestration | Self-configuring nodes on boot, edge devices |

```bash
ansible-pull -U https://github.com/org/ansible-config.git -i localhost, site.yml
```

**🔹 Performance tuning:** increase `forks`, enable **SSH pipelining** and `ControlPersist`, disable fact gathering when not needed, use `strategy: free`, and use `async/poll` for long tasks.

<br>

---

<br>