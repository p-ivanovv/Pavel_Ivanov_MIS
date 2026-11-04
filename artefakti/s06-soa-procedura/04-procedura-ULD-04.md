МРЕЖОВА И ИНФОРМАЦИОННА СИГУРНОСТ · шаблон 04-procedura

Процедура: ULD-04 - SSH само с ключ, без root и пароли


| Поле | Стойност |
| --- | --- |
| Реализира контрол | ULD-04 - лабораторна реализация |
| Annex A | 8.5, 8.2; подпомага 8.9 и 5.37 |
| R-ID | R-GSN-03 (12), активи A-GSN-05 и A-GSN-09 от artefakti/s04-risk/. Лабораторният srv е заместител, не фирмено внедряване. |
| Изпълнява се от | DevOps инженер |
| Честота | Първоначална настройка, след SSH промяна; месечен преглед до 5-о число (предложен срок) |
| Версия, дата | 1.2, 04.11.2026 г. |

Предусловия

| Какво | Стойност |
| --- | --- |
| Достъп | Нерутов vboxuser със sudo на srv; локална VirtualBox конзола. |
| Инструменти | Debian srv 10.202.0.10 с openssh-server; gw 10.202.0.1 с SSH клиент и ключ ~/.ssh/mis_h06. |
| Снапшот преди започване | pre-h06 или реално използваното pre-h06-key-working; името се проверява във VirtualBox. |
| Втора сесия | От gw: ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes vboxuser@10.202.0.10; провери sudo -v и остави отворена до успешни тестове. |

Стъпки


| # | Действие | Команда | Очакван резултат |
| --- | --- | --- | --- |
| 1 | Провери клиента | На gw: hostname; ip -br -4 addr | gw; адрес 10.202.0.1. При srv спри и отвори нов локален терминал. |
| 2 | Провери ключа преди промяната | На gw: ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no vboxuser@10.202.0.10 true; echo $? | 0. Passphrase на ключа е допустима; парола за srv не е. |
| 3 | Запиши преди | На srv: mkdir -p ~/artefakti/s06-soa-procedura<br>{ date -Is; hostname; sudo /usr/sbin/sshd -T; systemctl is-active ssh; } > ~/artefakti/s06-soa-procedura/predi.txt 2>&1 | Непразен файл от srv. Не презаписвай историческото predi.txt при повторен тест; използвай нова папка. |
| 4 | Провери включване и конфликти | На srv: sudo grep -nE "^[[:space:]]*(Include\|Match\|PermitRootLogin\|PasswordAuthentication\|KbdInteractiveAuthentication\|PubkeyAuthentication)" /etc/ssh/sshd_config<br>sudo ls -l /etc/ssh/sshd_config.d/ | Include за *.conf преди конфликтни настройки. При Match/по-ранна конфликтна стойност или съществуващ 10-uld04.conf спри за преглед; не презаписвай. |
| 5 | Създай drop-in | На srv: printf "%s\n" "PermitRootLogin no" "PasswordAuthentication no" "KbdInteractiveAuthentication no" "PubkeyAuthentication yes" \| sudo tee /etc/ssh/sshd_config.d/10-uld04.conf >/dev/null<br>sudo chmod 644 /etc/ssh/sshd_config.d/10-uld04.conf | Файл с четири директиви. |
| 6 | Синтаксис и стойности | На srv: sudo /usr/sbin/sshd -t<br>sudo /usr/sbin/sshd -T \| grep -E "^(permitrootlogin\|passwordauthentication\|kbdinteractiveauthentication\|pubkeyauthentication) " | Без синтактична грешка; no, no, no и yes за pubkey. При отклонение спри без reload. |
| 7 | Приложи | На srv: sudo systemctl reload ssh<br>systemctl is-active ssh | active; резервната сесия остава достъпна. |
| 8 | Запиши след | На srv: { date -Is; hostname; sudo /usr/sbin/sshd -T; systemctl is-active ssh; } > ~/artefakti/s06-soa-procedura/sled.txt 2>&1 | Стойностите след промяната са записани; sshd -T чете конфигурацията, затова следват реални нови връзки. |

Проверка

Положителна (разрешеното работи):

На gw, нов терминал:
mkdir -p ~/artefakti/s06-soa-procedura
{ date -Is; hostname; ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no vboxuser@10.202.0.10 "hostname; whoami"; echo "positive_rc=$?"; } > ~/artefakti/s06-soa-procedura/positive.txt 2>&1
Очаквано: gw, srv, vboxuser, positive_rc=0.

Отрицателна (забраненото се отказва):

На gw, нови връзки, не резервната сесия:
{ date -Is; hostname;
 echo "TEST password"; ssh -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PubkeyAuthentication=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=password vboxuser@10.202.0.10 true; echo "password_rc=$?";
 echo "TEST keyboard-interactive"; ssh -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PubkeyAuthentication=no -o PasswordAuthentication=no -o PreferredAuthentications=keyboard-interactive vboxuser@10.202.0.10 true; echo "keyboard_rc=$?";
 echo "TEST root"; ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no root@10.202.0.10 true; echo "root_rc=$?";
} > ~/artefakti/s06-soa-procedura/otkaz.txt 2>&1
Очаквано: Permission denied (publickey), код 255 за всеки тест. Timeout, refused или host-key грешка не са доказателство. При поискана отдалечена парола прекрати с Ctrl+C и запиши отклонение.
На srv: sudo /usr/sbin/sshd -T -C user=root,addr=10.202.0.1,host=gw,laddr=10.202.0.10,lport=22 | grep -E "^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication) "
Запиши с date -Is и hostname в root-settings.txt. Root отказът сам не доказва забрана, защото ключът не е разрешен за root; съпостави с permitrootlogin no и журнала.

При отклонение

При неочакван резултат спри веднага. Запиши отклонението и уведоми преподавателя/отговорния DevOps до 15 минути (предложен срок). При неуспешен разрешен вход или успешен забранен вход не отчитай контрол като успешно проверен.

Възстановяване (Rollback)

От резервната сесия или конзолата на srv:
sudo rm -- /etc/ssh/sshd_config.d/10-uld04.conf
sudo /usr/sbin/sshd -t && sudo systemctl reload ssh
systemctl is-active ssh
Очаквано active и старо поведение на новите връзки. Ако достъпът не може да се възстанови през конзолата, върни pre-h06. Snapshot restore отменя и по-късни промени; запази доказателствата извън VM предварително.

Запис

На srv: sudo journalctl -u ssh --since "30 min ago" --no-pager > ~/artefakti/s06-soa-procedura/journal.txt
Съхранявай predi.txt, sled.txt, positive.txt, otkaz.txt, root-settings.txt и journal.txt в artefakti/s06-soa-procedura/ на репото след преглед за тайни. Предложен срок: 12 месеца, не е одобрен. Частни ключове и пароли не се качват. Реалните тестове от 04.11.2026 са успешни; peer review не е извършен. Това доказва лабораторния srv, не цялостно фирмено внедряване.

Проверка от съученик

| Поле | Стойност |
| --- | --- |
| Изпълнил, дата | Не е извършена проверка от съученик; не се попълва сега. |
| Засякъл на стъпка | Не е извършена проверка от съученик; не се попълва сега. |
| Неясно | Не е извършена проверка от съученик; не се попълва сега. |
| Отрицателният тест даде | Не е извършена проверка от съученик; не се попълва сега. |
| Поправено след проверката | Не е извършена проверка от съученик; не се попълва сега. |


Автор на документа: Павел Иванов

Статус след съгласуване: лабораторният ULD-04 е технически проверен; фирмената мярка за R-GSN-03 остава планирана. Проверка от съученик не е извършена. Името и датата на автора са запазени от предоставения документ.

