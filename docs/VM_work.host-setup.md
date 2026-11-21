# Создаём и настроаиваем ВМ на рабочем хосте 

## Гипервизор - VMware Workstation Pro v25.0.0.24995812

**VM_work.host — Ubuntu 26.04 server, RAM 8 GB, SSDHard disk (SCSI) 30 GB**

**Подключение к VM по SSH**
`ssh-copy-id <user>@*.*.*.*`
`ssh <user>@*.*.*.*`

**Настройка ВМ**

`sudo apt update && sudo apt upgrade -y`
`sudo timedatectl set-timezone Europe/Moscow`

**Установка docker**

``