# Получаем информацию о хосте, создаем точку восстановление
# Work_host — рабочий ноутбук (Win11 Pro, RAM 32 GB, SSD 512)

winver: 26H2 сбоока:26300.9974

Диск под ВМ: 30 ГБ, свободно 240 ГБ

Get-Volume | Select DriveLetter, FileSystemLabel, Size, SizeRemaining

Checkpoint-Computer -Description "Checkpoint (date)" -RestorePointType MODIFY_SETTINGS

# Включаем необходимые компоненты на Windows хосте для wsl2, гипервизора и запуска ВМ внутри Windows

Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux -NoRestart

Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart

Enable-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform -All -NoRestart

# Проверка доступности используемых конечных точек на хосте

curl.exe -I https://github.com

curl.exe -I https://ghcr.io/v2/

curl.exe -I https://download.docker.com

curl.exe -I https://dockerhub.timeweb.cloud/v2/

curl.exe -I https://s3.cloud.ru

curl.exe -I https://go.dev/dl/

# Установка WSL 2

`wsl --install -d Ubuntu-24.04`

# Лимит ресурсов для ВМ wsl

```bash
`%userprofile%\.wslconfig

[wsl2]
processors=4
memory=6442450944
```

`wsl --shutdown`


# Установка тулчейнов

`sudo apt install -y git curl unzip python3-venv pipx`

`sudo snap install --classic opentofu`

`sudo pipx install --include-deps ansible`

`sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip && sudo unzip awscliv2.zip && sudo ./aws/install`

# Клонирование репозитория с github

`git config --global user.name "lettache"`

`git config --global user.mail "<ID+lettache@users.noreply.github.com>"`

`ssh-keygen -t ed25519 -C "lettachev@work-laptop"`

`git clone git@github.com:lettache/NextCloud.git ~/NextCloud`

# Проверка доступности используемых конечных точек на wsl: Ubuntu-24.04

`curl -I https://github.com https://ghcr.io/v2/ https://download.docker.com https://dockerhub.timeweb.cloud/v2/ https://s3.cloud.ru https://go.dev/dl/`

# Создание профиля aws и проверка подключения к S3 cloud.ru

```bash
aws configure --profile s3_cloudru
chmod 600 ~/.aws/credentials 
EP=https://s3.cloud.ru
aws --profile s3_cloudru --endpoint-url $EP s3 ls

```