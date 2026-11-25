# Создаём структуру ansible

`ansible/ansible.cfg`

```bash
[defaults]
inventory = ansible/inventory/hosts.yaml
roles_path = ansible/roles
vault_password_file = ansible/vault-pass.txt
```

```bash
ansible/
├── ansible.cfg
├── inventory/hosts.yaml        # уже есть
├── group_vars/all.yml          # обычные переменные (не секреты)
├── vault/secrets.yml           # секреты, зашифрованы
├── playbooks/site.yml          # «кнопка запуска»
└── roles/
    ├── docker_host/            # роль 1: Docker
    └── nextcloud_stack/        # роль 2: стек Nextcloud
```

`ansible-vault create ansible/vault/secrets.yml`

```bash
postgres_password: 
nextcloud_admin_password: 
s3_access_key:
s3_secret_key:
```

**Роль docker_host**

`roles/docker_host/tasks/main.yml`

```bash
- name: Ключ репозитория Docker
  ansible.builtin.apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Репозиторий Docker
  ansible.builtin.apt_repository:
    repo: "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu {{ ansible_distribution_release }} stable"
    state: present

- name: Пакеты Docker
  ansible.builtin.apt:
    name:
      - docker-ce
      - docker-ce-cli
      - containerd.io
      - docker-buildx-plugin
      - docker-compose-plugin
    state: present
    update_cache: yes

- name: daemon.json с зеркалами
  ansible.builtin.copy:
    dest: /etc/docker/daemon.json
    content: |
      { "registry-mirrors": ["https://dockerhub.timeweb.cloud", "https://mirror.gcr.io"] }
    notify: Перезапустить docker

- name: vlk в группе docker
  ansible.builtin.user:
    name: vlk
    groups: docker
    append: yes

- name: Docker запущен и включён в автостарт
  ansible.builtin.service:
    name: docker
    state: started
    enabled: yes
```

`handlers/main.yml`

```bash
- name: Перезапустить docker
  ansible.builtin.service:
    name: docker
    state: restarted
```

`ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/site.yml`
