# Описание

Это мой дипломный проект по проверке IP-адресов в различных TI-фидах на базе Logstash  
Для запуска требуется установить `logstash-filter-public_ip` командой

    .\logstash-plugin install logstash-filter-public_ip

в директории bin

Дополнительно требуется установить [`logstash-filter-teamcymru`](https://github.com/summergaga/logstash-filter-teamcymru)
# Структурная схема
```mermaid
graph TD;
    A[stdin input] -->|Filter part| B[Grok IP parser];
    B --> C{Public IP filter};
    C --> |Not public IP| I[Output];
    C --> |Public IP| D[Spamhaus check for IPv4];
    D --> E[RKN block checker];
    E --> F[Virustotal check];
    F --> G[AbuseIPDB check];
    G --> H[Team Cymru check];
    H --> I;
```

# Механизм работы
## Spamhaus
Если DNS-запрос вернул 172.0.0.2 - адрес присутствует в спам-листах Spamhaus. [Коды](https://www.spamhaus.org/zen/) и их [описание](https://www.spamhaus.org/faq/section/DNSBL%20Usage#200).
## Блокировки Роскомнадзора
Проверяем IP через HTTP-запрос к [реестру Роскомсвободы](https://reestr.rublacklist.net/). Если он там есть, возможно, наша система участвует в [атаке](https://www.usenix.org/system/files/sec21fall-bock.pdf).

## Проверка Virustotal
Получаем оценку от Virustotal в интервале [-100, 100], где -100 - "абсолютно вредоносный", а 100 - "абсолютно безвредный". [Подробнее](https://support.virustotal.com/hc/en-us/articles/115002146769-Comments).

## Проверка AbuseIPDB
Интервал оценки - [0, 100]. 0 - безвреден, 100 - вредоносный. [Подробнее](https://www.abuseipdb.com/faq.html#confidence).

## Проверка Teamcymru
Рейтинг от 0 до 100, аналогично AbuseIPDB.

# Примеры вывода
```
1.1.1.1 - Cloudflare DNS one.one.one.one
{
             "src_ipv" => "4",
       "src_public_ip" => true,
                  "ip" => "1.1.1.1",
     "abuseipdb_score" => 0,
    "virustotal_score" => 69,
    "teamcymru" => 56,
}
```
```
104.21.56.234 - rutracker.org
{
       "src_public_ip" => true,
             "src_ipv" => "4",
                  "ip" => "104.21.56.234",
     "abuseipdb_score" => 0,
                "tags" => [
        [0] "RKN blocklisted"
    ],
    "virustotal_score" => 0,
    "teamcymru" => 0,
}
```
```
2606:4700:3036::6815:38ea - rutracker.org IPv6
{
    "abuseipdb_score" => 0,
            "src_ipv" => "6",
               "tags" => [
        [0] "RKN blocklisted"
    ],
      "src_public_ip" => true,
                 "ip" => "2606:4700:3036::6815:38ea"
}
```
```
91.238.229.134 - IP from AS58042
{
            "src_ipv" => "4",
      "src_public_ip" => true,
    "abuseipdb_score" => 100,
    "virustotal_score" => 0,
    "teamcymru" => 17,
                 "ip" => "91.238.229.134"
}
```