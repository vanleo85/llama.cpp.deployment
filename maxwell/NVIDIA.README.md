
## Удаление nvidia drivers
apt update
apt upgrade
apt purge nvidia* -y
apt autoremove -y
sudo reboot
apt update

## Поиск доступных версий
ubuntu-drivers devices

## Установка драйвера и утилит (headless, без X11, если не нужна графика)
sudo apt install --no-install-recommends nvidia-driver-580 nvidia-utils-580
sudo reboot
nvidia-smi

## Установка nvidia-container-toolkit

### Порядок установки ниже по ссылке
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

### Обязательно надо перезапустить docker
sudo systemctl stop docker
sudo systemctl start docker

## Установка nvidia-cuda-toolkit
sudo apt install nvidia-cuda-toolkit


## Настройка gpu сервера

### Включаем режим постоянного управления (Persistence Mode)
sudo nvidia-smi -pm 1

### Ограничиваем максимальное энергопотребление (например, до 180W вместо 250W)
### Это снизит температуру на 10-15 градусов при потере производительности всего 5-7%
sudo nvidia-smi -pl 180
