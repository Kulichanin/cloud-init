# Подготовка к запуску
Инструмент от [Canonical](https://cloud-init.io/).
Позволяет выполнить настройки системы после развертывания из шаблона. (Устанавливать пакеты во время инициализации)
Запускается раньше сетевого стека и может настроить сетевой адаптер. (Например, задать шлюз и DNS)
Может заменить Packer. (но он все равно должен быть в шаблонном образе )

Позволяет автоматизировать:
- Установку имени хоста
- Создание пользователей
- Настройку SSH-ключей
- Монтирование дисков
- Запуск произвольных команд
  
## Как работает cloud-init?

Основная [дока](https://cloudinit.readthedocs.io/en/latest/index.html).

1. **Запускается при первой загрузке** ВМ (если в системе установлен `cloud-init`).
2. **Ищет конфигурацию** в одном из источников:
    - Метаданные от cloud-провайдера (в случае QEMU/KVM — через `NoCloud` datasource).
    - Файл `user-data` (обычно в формате YAML или cloud-config).
3. **Применяет настройки** и отключается (если не настроено иное).

## Работа с QUME
Для экспериментов будем использовать [QUME](https://www.qemu.org/) и образ image [Ubuntu Jammy](https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img)

```bash
sudo apt-get install -y qemu-system cloud-localds
mkdir ~/lab
wget https://cloud-images.ubuntu.com/jammy/current/jammy-server-cloudimg-amd64.img
```

## Создаём ISO с cloud-init данными

```bash
cloud-localds -v cloud-init.iso user-data meta-data
```

Теперь можно примонтировать этот ISO к виртуальной машине в QEMU/KVM.

## Запуск ВМ с образом

```bash
# Базовая команда для запуска ВМ
qemu-system-x86_64 \
  -name "lab-vm" \          # Имя виртуальной машины
  -enable-kvm \             # Использовать KVM для ускорения
  -cpu host \               # Пробросить CPU хоста
  -m 2048 \                 # Выделить 2 ГБ оперативной памяти
  -hda jammy-server-cloudimg-amd64.img \ # Основной диск, который мы загрузили предварительно
  -cdrom cloud-init.iso \   # Диск конфигурации cloud-init
  -netdev user,id=net0 \    # Настройка сети (режим "user")
  -device virtio-net-pci,netdev=net0 \
  -redir tcp:8080::80 \      # Проброс портов nginx
  -redir tcp:2222::22 \     #  Проброс портов ssh
  -nographic                # Запуск в консоли (опционально)
```

## Hot keys
Для работы с QUME из консоли
```bash
Ctrl-A C
```
Прибить процесс qume
```bash
Ctrl-A X
```

## Дополнительные материалы

[Ссылка](https://docs.google.com/presentation/d/1g-0pXzeB0ZSisv9CiwIQFzDd-qTGt_xsIbJdSBuibjI/edit?usp=sharing) на презентацию
