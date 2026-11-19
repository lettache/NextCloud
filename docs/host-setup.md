# Получаем информацию о хосте, создаем точку восстановление
# Host setup — рабочий ноутбук (Win11 Pro, 32 GB)

winver: 26H2 сбоока:26300.9974 | диск под ВМ: 30 ГБ, свободно 240 ГБ

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

'curl.exe -I https://go.dev/dl/'
