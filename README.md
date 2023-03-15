# Subspace-Genemi3-c

Требования по железу:
Официальные требования: 4 CPU x 4 GB RAM x 50 GB SSD - Ubuntu 20.04.

# обновляем базу данных, обновляем дистрибутив

`sudo apt-get update && sudo apt-get upgrade -y`

# скачиваем необходимые зависимости одной командой

sudo apt-get install wget jq ocl-icd-opencl-dev \ 

libopencl-clang-dev libgomp1 ocl-icd-libopencl1 -y

# скачиваем исполняемые файлы и выводим их версии одной командой

mkdir $HOME/subspace >/dev/null 2>&1 && \ 

cd $HOME/subspace && \ 

VER=$(wget -qO- https://api.github.com/repos/subspace/subspace-cli/releases | jq '.[] | select(.prerelease==false) | select(.draft==false) | .html_url' | grep -Eo "v[0-9]*.[0-9]*.[0-9]*-alpha" | head -n 1) && \ 

wget https://github.com/subspace/subspace-cli/releases/download/${VER}/subspace-cli-ubuntu-x86_64-${VER} -qO subspace; \ 
sudo chmod +x * && \ 

if [[ $(./subspace -h) == "" ]]; then
  echo -e "\n\ndat sh*t is broken, ping @cyberomanov.\n\n"
else
  sudo mv * /usr/local/bin/ && \ 

  echo -e "\n\nrelease >> ${VER}.\n\n"
fi && \ 

cd $HOME && \ 

rm -Rvf $HOME/subspace >/dev/null 2>&1

Теперь нам нужен кошелёк.
 Переходим на дашборд Subspace Gemini 3c

https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Feu-0.gemini-3c.subspace.network%2Fws#/accounts
 и копируем адрес нашего кошелька. 

Если по каким-то причинам у вас ещё нет кошелька полькусамы, нужно скачать polkadot.js расширение и сгенерировать кошелёк там

https://polkadot.js.org/extension/

На этот адрес мы будем фармить TSSC токены, как доказательство участия в тестнете.

Пример адреса кошелька  5HKd3GpsaTQsh8Y5LembrtQFvuurjrzsB4xDt5E68c8T8oAK

# инициализируемся
```subspace init```

Вводим кошелёк из прошлого пункта;

Вводим желаемый никнейм;

Остальное я оставил по дефолту (просто нажать Enter).


# фиксим журнал
`sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF`


# рестартим журнал
`sudo systemctl restart systemd-journald`

# создаём файл сервиса для запуска ноды

sudo tee <<EOF >/dev/null /etc/systemd/system/subspaced.service

[Unit]

Description=Subspace Node

After=network.target

[Service]

Type=simple

User=$USER

ExecStart=$(which subspace) farm -v

Restart=always

RestartSec=10

LimitNOFILE=65535

[Install]

WantedBy=multi-user.target

EOF




# запускаем ноду

sudo systemctl daemon-reload && \
sudo systemctl enable subspaced && \
sudo systemctl restart subspaced


# проверяем логи

sudo journalctl -fu subspaced --no-hostname -o cat


За синхронизацией можно следить в прямом эфире с помощью команды:

sudo journalctl -fu subspaced -o cat | grep -E "best"

Ещё можно наблюдать синхронизацию в телеметрии, но она часто отваливается.

https://telemetry.subspace.network/#list/0xab946a15b37f59c5f4f27c5de93acde9fe67a28e0b724a43a30e4fe0e87246b7

 "Идеальную" высоту можно найти в экслорере.
https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Feu-0.gemini-2a.subspace.network%2Fws#/explorer

## Полезные команды

# пезапуск фармера и ноды
sudo systemctl restart subspaced
# остановка фармера и ноды
sudo systemctl stop subspaced
# проверяем логи
sudo journalctl -fu subspaced --no-hostname -o cat


## Удаление

# останавливаем и отключаем сервис ноды одной командой
sudo systemctl stop subspaced && \
sudo systemctl disable subspaced
# удаляем остаточные файлы
rm -Rvf $HOME/.local/share/subspace*
# удаляем файл сервиса и перезагружаем демона одной командой
sudo rm -v /etc/systemd/system/subspaced.service \
/etc/systemd/system/farmerd.service && \
sudo systemctl daemon-reload
