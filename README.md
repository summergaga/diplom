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