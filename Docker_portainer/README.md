## Организация Docker с использованием Portainer на виртуальной машине Linux  

# Задача:  
Развернуть сервис docker на виртуальной машине (пр. ubuntu server 24.04 LTS) с использованием платформы Portainer.  
Удобный UI для управления Docker контейнерами прямо из браузера.

В данном примере будет использоваться:  
• Cреда виртуализации Proxmox  
• Дистрибутив ОС Ubuntu Server 24.04 LTS  
• Официальный дистрибутив Docker + Docker Compose  
• Контейнер платформы Portainer  

Краткая справка:  
**Portainer** — это инструмент управления контейнерами, который эффективно взаимодействует как с Docker, так и с Kubernetes

# Развёртка ВМ  

# Установка Docker и Docker Compose  
Установка GPG ключа Docker:  
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Добавление стабильного репозитория Docker:  
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```
Устанавливаем Docker + compose:  
```
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose
systemctl status docker
```
Важно, что при использовании ubuntu server версии 24.04 при использовании команды "**docker-compose**" возникает ряд ошибок связанных с библиотеками python. Верным решением будет использовать команду "**docker compose**", без ** - **.  
# Развёртка контейнера службы Portainer:
Обратим внимание, развёртывание контейнера происходит без compose файла напрямую через консоль. 
```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    portainer/portainer-ce:latest
```
Пояснение:  
Для работы Portainer использует 2 порта (в данном случае ими являются 8000 и 9433) для корректной работы  
атрибут **--restart-always** обозначает, что вне зависимости от возникших обстоятельств, контейнер будет перезагружаться всегда, пока не получит статус "running"  
атрибут **-v /var/run/docker.sock:/var/run/docker.sock** "соединяет" сокет контейнера с физическим сокетом докера, для работы системы  
атрибут **-v portainer_data:/data \** похож на атрибут выше, но соединяет папку data контейнера с физической папкой машины на которой развёрнут docker  (образно говоря создаёт ярлык, пусть это и неккоретное описание)  
**portainer/portainer-ce:latest** - репозиторий из которого будет осуществлятся pull файлов для установки сервиса Portainer.  

В результате установки, можно перейти по адресу машины (в данном примере 192.168.10.27) с указанием порта, который мы задавали выше при развёртывании контейнера. (в данном примере: https://192.168.10.27:9443)  

Если все этапы были выполнены корректно и развёртка прошла без ошибок, при первом переходе по ссылке будет видна форма, в которой необходимо ввести свои данные для входа (вы создаёте их самостоятельно):  
![image](https://github.com/NyashMan/LinuxSysa/assets/1348639/2923a2a6-4808-44c4-9757-bb77b8d6bd5d)  
Далее попадаем в административную панель Portainer:  
![image](https://github.com/NyashMan/LinuxSysa/assets/1348639/6be5037a-06d3-463e-bfb8-c2961a1ceb88)  

Установка завершена!  

