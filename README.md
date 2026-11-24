
## Готовые решения для гибридного каскада (AWG → РФ → VLESS Reality)

AWG Router (веб-панель для каскадной маршрутизации)
	• Как работает. Поднимает сервер AmneziaWG на российском VPS и дает веб-интерфейс, где для каждого клиента можно задать правило выхода: пустить трафик через другой AWG-сервер или завернуть его в локальный SOCKS5-выход (в роли которого выступает **xray-core** с настроенным VLESS Reality на зарубежный сервер).
	• Плюсы. Есть встроенная проверка «здоровья» VLESS-выхода (через отправку TLS ClientHello) и защита от утечки трафика (unreachable fallback), если зарубежный узел временно недоступен.

---

## Устанавливаем [AWG-Router](https://github.com/r-slashslash/AWG-Router/blob/main/README.ru.md)

```bash
curl -fsSL https://raw.githubusercontent.com/r-slashslash/AWG-Router/main/install.sh | bash
```

## Запускаем xray-core в Docker

1. Предварительно через клиент AmneziaWG или иным способом на конечном сервере устанавливаем контейнер XRay + VLESS Reality.
2. Полученый конфиг для xray **amnezia_for_xray.json** открываем и правим в блоке inbounds прописан "protocol": "socks", "listen": "0.0.0.0", "port": 10808 и "udp": true. Сохраняем этот файл на российский сервер в /opt/xray/config.json.
   Должно получится:
   
   ```json
    "inbounds": [
      {
        "listen": "0.0.0.0",
        "port": 10808,
        "protocol": "socks",
        "settings": { "udp": true }
      }
    ]
   ```
4. Запускаем контейнер с скопированным на сервер конфигом.

```bash
# Для удаления
docker rm -f xray-exit

# Для запуска контейнера 
docker run -d --name xray-exit \
  --restart unless-stopped \
  -p 172.17.0.1:10808:10808/tcp \
  -p 172.17.0.1:10808:10808/udp \
  -v /opt/xray/config.json:/etc/xray/config.json \
  ghcr.io/xtls/xray-core:latest run -c /etc/xray/config.json
```

5. После старта контейнера проверить можно командой
   ```bash
    curl -x socks5h://172.17.0.1:10808 https://ifconfig.me
   ```
   Команда должна показать адрес зарубежного сервера.

6. В панели **AWG Router** выход нужно задать в формате socks5://172.17.0.1:10808 (протокол в настройках AWG Router пишется как socks5://).
