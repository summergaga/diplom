# diplom

This is my diplom project "Logstash-based IP checker"  
Run

    .\logstash-plugin install logstash-filter-virustotalthree logstash-filter-public_ip

in bin directory before using this plugin

Execution order
```mermaid
graph TD;
    A[stdin input] -->|Filter part| B[Grok IP parser];
    B --> C{Public IP filter};
    C --> |Not public IP| J[Output];
    C --> |Public IP| D[Spamhaus check for IPv4];
    D --> E[Geoip filter];
    E --> F[RKN block checker];
    F --> G[Virustotal check];
    G --> H[AbuseIPDB check];
    H --> I[Team Cymru check];
    I --> J;
```