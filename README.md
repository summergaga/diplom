# Описание

Это мой дипломный проект по проверке IP-адресов в различных TI-фидах на базе Logstash  
Для запуска требуется установить `logstash-filter-public_ip` командой

    .\logstash-plugin install logstash-filter-public_ip

в директории bin.

Дополнительно требуется установить [`logstash-filter-teamcymru`](https://github.com/summergaga/logstash-filter-teamcymru).
# Структурная схема
```mermaid
graph TD;
    A[stdin input] -->|Filter part| B[Grok IP parser];
    B --> C{Public IP filter};
    C --> |Not public IP| K[Output];
    C --> |Public IP| D{IP Version};
    D --> |IPv4| E[Spamhaus check];
    E --> F[Alienvault check];
    F --> G[Virustotal check];
    G --> H[AbuseIPDB check];
    H --> I[TeamCymru check];
    I --> J[Total result V4];
    J --> K;
    D --> |IPv6| L[Alienvault check];
    L --> M[AbuseIPDB check];
    M --> N[Total result V6];
    N --> K;
```

# Механизм работы
## Проверка версии и типа IP-адреса
Используется плагин `logstash-filter-public_ip`
## Spamhaus
Если DNS-запрос вернул 127.0.0.4 - адрес присутствует в спам-листах Spamhaus. [Коды](https://www.spamhaus.org/zen/) и их [описание](https://www.spamhaus.org/faq/section/DNSBL%20Usage#200).
## Рейтинг Alienvault
Если возвращает 0, то IP чист, если 1 - то вредоносный.
Работает через [API](https://otx.alienvault.com/api).
[Ссылка на списки вредоносных IP](https://gist.github.com/bsmartt13/efa02c40ea12c09d9c3a).
## Проверка Virustotal
Получаем оценку от Virustotal в интервале [-100, 100], где -100 - "абсолютно вредоносный", а 100 - "абсолютно безвредный". [Подробнее](https://support.virustotal.com/hc/en-us/articles/115002146769-Comments).

## Проверка AbuseIPDB
Интервал оценки - [0, 100]. 0 - безвреден, 100 - вредоносный. [Подробнее](https://www.abuseipdb.com/faq.html#confidence).

## Проверка Teamcymru
Рейтинг от 0 до 100, аналогично AbuseIPDB.

## Вывод общего результата
Простое сравнение каждого параметра с порогом.
# Примеры вывода
```
1.1.1.1 - Cloudflare DNS one.one.one.one
{
                  "ip" => "1.1.1.1",
     "abuseipdb_score" => 0,
    "virustotal_score" => 69,
    "spamhaus_score" => 0,
    "teamcymru_score" => 56,
    "alienvault_score" => 0,
    "Total match" => 2,
}
```
```
104.21.56.234 - rutracker.org
{
                  "ip" => "104.21.56.234",
     "abuseipdb_score" => 0,
    "alienvault_score" => 0,
    "virustotal_score" => 0,
    "teamcymru_score" => 0,
    "spamhaus_score" => 0,
    "Total match" => 0,
}
```
```
2606:4700:3036::6815:38ea - rutracker.org IPv6
{
    "Total match" => 0,
    "abuseipdb_score" => 0,
    "alienvault_score" => 0,
                 "ip" => "2606:4700:3036::6815:38ea"
}
```
```
91.238.229.134 - IP from AS58042 (СПбГУТ)
{
    "Total match" => 1,
    "alienvault_score" => 0,
    "abuseipdb_score" => 100,
    "spamhaus_score" => 0,
    "virustotal_score" => 0,
    "teamcymru_score" => 1,
                 "ip" => "91.238.229.134"
}
```
```
203.248.175.71 - IP from AS3786 (LG DACOM Corporation)
{
    "virustotal_score" => -6,
                  "ip" => "203.248.175.71",
      "spamhaus_score" => 0,
     "abuseipdb_score" => 100,
     "teamcymru_score" => 100,
    "alienvault_score" => 1,
    "Total match" => 4,
}
```