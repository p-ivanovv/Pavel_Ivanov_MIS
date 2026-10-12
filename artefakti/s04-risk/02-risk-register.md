# Разширен регистър на риска

Оценката използва скала от 1 до 5 за вероятност и въздействие.

`Риск = Вероятност x Въздействие`

## Обобщена таблица

| ID | Риск | Засегнати активи | MITRE ATT&CK техника | Тактика | Вероятност | Въздействие | Риск | Стратегия |
|---|---|---|---|---|---:|---:|---:|---|
| R01 | Exploit на публична уеб услуга на `srv` | A05, A06, A08 | T1190 - Exploit Public-Facing Application | Initial Access | 4 | 5 | 20 | Mitigate |
| R02 | Brute force срещу SSH на `gw` или `srv` | A07, A09 | T1110 - Brute Force | Credential Access | 4 | 4 | 16 | Mitigate |
| R03 | Използване на компрометирани или слаби акаунти | A07, A08, A09 | T1078 - Valid Accounts | Initial Access, Persistence, Privilege Escalation | 3 | 5 | 15 | Mitigate |
| R04 | Сканиране и откриване на услуги в LAN | A03, A04, A05 | T1046 - Network Service Discovery | Discovery | 5 | 3 | 15 | Mitigate |
| R05 | Exploit на remote service за движение към `srv` | A02, A05 | T1210 - Exploitation of Remote Services | Lateral Movement | 3 | 5 | 15 | Mitigate |
| R06 | Изпълнение на злонамерен shell или скрипт | A05, A08 | T1059 - Command and Scripting Interpreter | Execution | 3 | 5 | 15 | Mitigate |
| R07 | Създаване на нов привилегирован акаунт | A09 | T1136 - Create Account | Persistence, Privilege Escalation | 3 | 4 | 12 | Mitigate |
| R08 | Persistence чрез cron или systemd задача | A05, A08 | T1053 - Scheduled Task/Job | Execution, Persistence, Privilege Escalation | 2 | 5 | 10 | Mitigate |
| R09 | Изтичане на конфигурации или ключове чрез Git | A08, A11 | T1552 - Unsecured Credentials | Credential Access | 3 | 4 | 12 | Mitigate |
| R10 | DNS или outbound C2 трафик от `srv` | A01, A05 | T1071 - Application Layer Protocol | Command and Control | 2 | 5 | 10 | Mitigate |
| R11 | Denial of service срещу уеб услугата | A06 | T1498 - Network Denial of Service | Impact | 3 | 4 | 12 | Mitigate |
| R12 | Повреда или загуба на конфигурация | A02, A05, A10 | T1565 - Data Manipulation | Impact | 2 | 4 | 8 | Mitigate |

## R01 - Exploit на публична уеб услуга

- **ATT&CK:** [T1190 - Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- **Тактика:** Initial Access
- **Засегнати активи:** `srv`, уеб услуга, конфигурации и данни
- **Релевантност:** `srv` изпълнява уеб или приложна услуга. Уязвимост, слаба конфигурация или непачен компонент може да позволи на нападателя да получи първоначален достъп до VM-а.
- **APT примери:** APT28, APT29, APT41 и Volt Typhoon са сред групите, които ATT&CK описва като използващи exploit на публично достъпни приложения за първоначален достъп.
- **Mitigations:** M1051 Update Software, M1016 Vulnerability Scanning, M1050 Exploit Protection, M1030 Network Segmentation, M1035 Limit Access to Resource Over Network, M1026 Privileged Account Management
- **Обосновка на оценката:** Вероятността е висока, защото уеб услугите често са първата атакувана повърхност. Въздействието е критично, защото компрометирането на `srv` може да разкрие данни и конфигурации.
- **Доказателство от lab:** Резултатите от `nmap` и `ffuf` трябва да покажат кои портове и web пътища са достъпни на `srv`.

## R02 - Brute force срещу SSH

- **ATT&CK:** [T1110 - Brute Force](https://attack.mitre.org/techniques/T1110/)
- **Тактика:** Credential Access
- **Засегнати активи:** SSH достъп, потребителски акаунти, `gw` и `srv`
- **Релевантност:** SSH е основният административен канал към двете Debian VM-и. Слаба парола, разрешен password authentication или липса на rate limiting правят brute force възможен.
- **APT примери:** APT28, APT38, APT39, APT41, Lazarus Group и Turla са свързани с brute force дейност в ATT&CK.
- **Mitigations:** M1036 Account Use Policies, M1032 Multi-factor Authentication, M1027 Password Policies, M1026 Privileged Account Management, M1018 User Account Management
- **Обосновка на оценката:** Вероятността е висока, защото SSH е често сканирана услуга. Въздействието е високо, защото успешен вход дава директен контрол над системата.
- **Доказателство от lab:** Логове от `/var/log/auth.log`, неуспешни SSH входове и конфигурацията на `sshd_config`.

## R03 - Използване на компрометирани акаунти

- **ATT&CK:** [T1078 - Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- **Тактика:** Initial Access, Persistence, Privilege Escalation
- **Засегнати активи:** Потребителски акаунти, SSH, конфигурации и данни
- **Релевантност:** Нападател може да използва валидни локални или SSH credentials, за да влезе като нормален потребител. Това затруднява откриването, защото трафикът изглежда легитимен.
- **APT примери:** APT28, APT29, APT33, APT39, APT41, Lazarus Group, Sandworm Team и Volt Typhoon са описани от ATT&CK като използващи валидни акаунти за достъп, персистентност или латерално движение.
- **Mitigations:** M1036 Account Use Policies, M1032 Multi-factor Authentication, M1027 Password Policies, M1026 Privileged Account Management, M1018 User Account Management
- **Обосновка на оценката:** Вероятността е средна до висока, защото един компрометиран акаунт може да се използва многократно. Въздействието е критично, защото валидните credentials заобикалят много от обичайните защиты.
- **Доказателство от lab:** SSH логове, `sudo` употреба, създадени акаунти и промени в `/etc/passwd`.

## R04 - Сканиране и откриване на услуги

- **ATT&CK:** [T1046 - Network Service Discovery](https://attack.mitre.org/techniques/T1046/)
- **Тактика:** Discovery
- **Засегнати активи:** LAN1, LAN2, `srv`, мрежови услуги
- **Релевантност:** Нападател, който вече има достъп до вътрешна мрежа, може да използва `nmap` или подобен инструмент, за да открие активни host-ове, отворени портове и уязвими услуги.
- **APT примери:** APT32, APT39, APT41, FIN13 и Volt Typhoon са сред групите, които ATT&CK свързва с мрежово откриване и сканиране. ATT&CK изрично посочва `nmap` като инструмент, използван от атакуващи групи.
- **Mitigations:** M1042 Disable or Remove Feature or Program, M1031 Network Intrusion Prevention, M1030 Network Segmentation
- **Обосновка на оценката:** Вероятността е много висока, защото сканирането е лесно за изпълнение. Въздействието е средно, защото само по себе си не компрометира системата, но дава информация за следваща атака.
- **Доказателство от lab:** `nmap` scan на LAN1/LAN2, отворени портове на `srv` и firewall правила на `gw`.

## R05 - Exploit на remote service

- **ATT&CK:** [T1210 - Exploitation of Remote Services](https://attack.mitre.org/techniques/T1210/)
- **Тактика:** Lateral Movement
- **Засегнати активи:** `gw`, `srv`, LAN1, LAN2
- **Релевантност:** След първоначален достъп нападателят може да exploit-не remote service в мрежата, за да получи достъп до `srv` или до други системи зад `gw`.
- **APT примери:** APT28, Fox Kitten, FIN7, MuddyWater, ShinyHunters и Wizard Spider са сред групите, които ATT&CK описва като exploit-ващи remote services за латерално движение.
- **Mitigations:** M1048 Application Isolation and Sandboxing, M1042 Disable or Remove Feature or Program, M1050 Exploit Protection, M1030 Network Segmentation, M1026 Privileged Account Management, M1051 Update Software, M1016 Vulnerability Scanning
- **Обосновка на оценката:** Вероятността е средна, защото изисква вече наличен достъп до мрежата. Въздействието е критично, защото води до латерално движение и разширяване на компромета.
- **Доказателство от lab:** Firewall правила между LAN1 и LAN2, отворени услуги на `srv` и резултати от vulnerability scan.

## R06 - Изпълнение на shell или скрипт

- **ATT&CK:** [T1059 - Command and Scripting Interpreter](https://attack.mitre.org/techniques/T1059/)
- **Тактика:** Execution
- **Засегнати активи:** `srv`, конфигурации, данни, потребителски акаунти
- **Релевантност:** След exploit или SSH достъп нападателят може да изпълнява bash, Python или други скриптове за разузнаване, персистентност и exfiltration.
- **APT примери:** APT19, APT32, APT37, APT39, Dragonfly, FIN6, Fox Kitten и OilRig са сред групите, които ATT&CK описва като използващи command и script interpreters.
- **Mitigations:** M1049 Antivirus/Antimalware, M1047 Audit, M1040 Behavior Prevention on Endpoint, M1045 Code Signing, M1042 Disable or Remove Feature or Program, M1038 Execution Prevention, M1033 Limit Software Installation, M1026 Privileged Account Management
- **Обосновка на оценката:** Вероятността е средна до висока, защото shell достъп е почти неизбежен след компрометация. Въздействието е критично, защото позволява пълно управление над системата.
- **Доказателство от lab:** auditd логове, необичайни child процеси на web service, reverse shell или изпълнение на `curl`/`wget` от `srv`.

## R07 - Създаване на привилегирован акаунт

- **ATT&CK:** [T1136 - Create Account](https://attack.mitre.org/techniques/T1136/)
- **Тактика:** Persistence, Privilege Escalation
- **Засегнати активи:** Потребителски акаунти, `srv`, `gw`
- **Релевантност:** Нападател с root или sudo достъп може да създаде нов локален акаунт, за да запази достъп дори след премахване на първоначалния exploit.
- **APT примери:** Създаване на акаунти е често използвана техника за персистентност от множество APT и ransomware групи; за точни примери се използват Procedure Examples в официалната ATT&CK страница на T1136.
- **Mitigations:** M1026 Privileged Account Management, M1018 User Account Management, M1027 Password Policies, M1032 Multi-factor Authentication
- **Обосновка на оценката:** Вероятността е средна, защото изисква вече получени привилегии. Въздействието е високо, защото новият акаунт осигурява трайна точка за достъп.
- **Доказателство от lab:** Промени в `/etc/passwd`, `/etc/shadow`, `sudo` групи и системни логове за `useradd`.

## R08 - Persistence чрез cron или systemd

- **ATT&CK:** [T1053 - Scheduled Task/Job](https://attack.mitre.org/techniques/T1053/)
- **Тактика:** Execution, Persistence, Privilege Escalation
- **Засегнати активи:** `srv`, конфигурации, услуги
- **Релевантност:** Нападател може да добави cron job или systemd timer, който периодично изпълнява malicious script или поддържа reverse shell.
- **APT примери:** APT29, APT41 и други групи са описани в ATT&CK като използващи scheduled tasks за персистентност; точните примери се взимат от Procedure Examples на T1053.
- **Mitigations:** M1047 Audit, M1018 User Account Management, M1026 Privileged Account Management, M1033 Limit Software Installation
- **Обосновка на оценката:** Вероятността е средна, защото изисква вече компрометирана система. Въздействието е критично, защото осигурява автоматично повторно изпълнение на злонамерен код.
- **Доказателство от lab:** `/etc/cron*`, `crontab -l`, `systemctl list-timers` и файлови промени в systemd unit файлове.

## R09 - Изтичане на secrets чрез Git

- **ATT&CK:** [T1552 - Unsecured Credentials](https://attack.mitre.org/techniques/T1552/)
- **Тактика:** Credential Access
- **Засегнати активи:** Git repository, конфигурации, ключове, данни
- **Релевантност:** Ако SSH private key, парола, API token или database credential попадне в repo, всеки с достъп до него може да използва credentials за достъп до `srv` или `gw`.
- **APT примери:** ATT&CK описва T1552 като търсене на credentials в файлове, histories, configuration files и други места; конкретните Procedure Examples на страницата показват реални случаи на злоупотреба.
- **Mitigations:** M1013 Application Developer Guidance, M1027 Password Policies, M1026 Privileged Account Management, M1047 Audit
- **Обосновка на оценката:** Вероятността е средна до висока, защото secrets често попадат случайно в Git. Въздействието е високо, защото изтичането може да даде директен достъп без exploit.
- **Доказателство от lab:** `git log`, `git grep`, `.gitignore`, наличие на `.env`, private keys или пароли в repository.

## R10 - C2 или outbound трафик от `srv`

- **ATT&CK:** [T1071 - Application Layer Protocol](https://attack.mitre.org/techniques/T1071/)
- **Тактика:** Command and Control
- **Засегнати активи:** `srv`, WAN връзка, данни
- **Релевантност:** Компрометиран `srv` може да използва HTTP, HTTPS или DNS за комуникация с attacker-controlled server, изтегляне на payload или exfiltration на данни.
- **APT примери:** T1071 е една от най-често използваните C2 техники и е свързана с множество APT групи в ATT&CK; конкретните примери се взимат от Procedure Examples на официалната страница.
- **Mitigations:** M1030 Network Segmentation, M1031 Network Intrusion Prevention, M1037 Filter Network Traffic, M1032 Multi-factor Authentication
- **Обосновка на оценката:** Вероятността е средна, защото изисква вече компрометиран `srv`. Въздействието е критично, защото C2 позволява дистанционно управление и exfiltration.
- **Доказателство от lab:** Outbound връзки от `srv`, DNS заявки към необичайни домейни и firewall/egress правила.

## R11 - Denial of service срещу уеб услугата

- **ATT&CK:** [T1498 - Network Denial of Service](https://attack.mitre.org/techniques/T1498/)
- **Тактика:** Impact
- **Засегнати активи:** Уеб услуга, `srv`, достъпност на услугата
- **Релевантност:** Атакуващ може да претовари уеб услугата с заявки или трафик, което да я направи недостъпна за легитимни потребители.
- **APT примери:** T1498 се използва основно от threat actors с цел disrupt; за конкретни кампании и групи се използват Procedure Examples в официалната ATT&CK страница.
- **Mitigations:** M1030 Network Segmentation, M1031 Network Intrusion Prevention, M1037 Filter Network Traffic, M1053 Data Backup
- **Обосновка на оценката:** Вероятността е средна, защото DoS не изисква компрометация. Въздействието е високо за достъпността, но обикновено не води до загуба на confidential данни.
- **Доказателство от lab:** Rate limiting, resource usage на `srv`, access logs и поведение при голям брой заявки.

## R12 - Повреда или загуба на конфигурация

- **ATT&CK:** [T1565 - Data Manipulation](https://attack.mitre.org/techniques/T1565/)
- **Тактика:** Impact
- **Засегнати активи:** `gw`, `srv`, VirtualBox snapshots, конфигурации
- **Релевантност:** Нападател или грешка при администриране може да промени firewall правила, systemd услуги, cron задачи или мрежови конфигурации, което да наруши работата на lab средата.
- **APT примери:** Data manipulation е използвана за sabotage и disruption от различни threat actors; конкретни примери се взимат от Procedure Examples на T1565 в ATT&CK.
- **Mitigations:** M1053 Data Backup, M1047 Audit, M1026 Privileged Account Management, M1018 User Account Management
- **Обосновка на оценката:** Вероятността е средна, защото включва и human error. Въздействието е високо, защото повредена конфигурация може да прекъсне цялата lab среда.
- **Доказателство от lab:** Snapshot преди промяна, Git history на конфигурационните файлове и restore тест.
