# ULD-04 - SSH само с ключ, без root и пароли

## Заглавна част

| Поле | Стойност |
| --- | --- |
| Фирма | GameStudio Neo OOD |
| Реализира контрол | ULD-04 - предложен допълнителен контрол по план R02, не GSN-04 |
| Annex A | 8.5, 8.2; подпомага 8.9 и 5.37 |
| R-ID | R02 (16), R03 (15), R12 (8) |
| Изпълнява се от | DevOps инженер |
| Честота | При първоначална настройка и SSH промяна; преглед месечно до 5-о число |
| Версия и дата | 1.0, 29.10.2026 |
| Статус | Подготвена; изпълнение и проверка предстоят |

## Обхват и предусловия

Лабораторен Debian srv с ssh.service и нерутов sudo акаунт. Не внедрява firewall, Jenkins, MFA или fail2ban. Не променя продукционната DMZ. Преди всяка промяна: snapshot pre-h06 на srv от VirtualBox и достъп до конзолата. На srv установи реалните акаунт и IP с whoami и ip -br addr. На gw въведи реалните стойности:

```bash
read -r -p 'IP/hostname на srv: ' SRV
read -r -p 'Нерутов sudo акаунт на srv: ' ADMIN
export SRV ADMIN
```

Ако SSH липсва, от конзолата на srv инсталирай openssh-server и стартирай ssh.service. Ако няма ключ на gw, създай отделен ключ с ssh-keygen -t ed25519 -f ~/.ssh/mis_h06 и копирай само публичния ключ чрез ssh-copy-id -i ~/.ssh/mis_h06.pub "$ADMIN@$SRV". Не презаписвай съществуващ ключ. Частният ключ не се качва в Git. Провери fingerprint на сървъра през конзолата преди първо доверяване. Подготовката на ключа се извършва ПРЕДИ забрана на паролите.

## 1. Положителен тест преди промяната - от gw

```bash
ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no "$ADMIN@$SRV" true
printf 'rc=%s\n' "$?"
```

Очаквано: код 0; допуска се локална passphrase за ключа, но не парола за отдалечения акаунт. При друг код спри. Отвори две отделни SSH сесии от gw към srv с този ключ. В първата провери sudo -v и я остави отворена до края като резервна. Във втората изпълни следващите стъпки.

## 2. Състояние преди - на srv

```bash
export D="$HOME/artefakti/s06-soa-procedura"
mkdir -p "$D"
test "$(id -u)" -ne 0 && sudo -v
sudo systemctl is-active ssh
sudo test ! -e /etc/ssh/sshd_config.d/10-uld04.conf
```

Очаквано: нерутов акаунт, active услуга, последният test връща 0. Ако файлът съществува, спри - не го презаписвай.

```bash
sudo tar -czf /root/ssh-pre-h06.tgz /etc/ssh
sudo chmod 600 /root/ssh-pre-h06.tgz
{ date -Is; hostname; id; sudo /usr/sbin/sshd -T; sudo systemctl status ssh --no-pager; } > "$D/predi.txt" 2>&1
sudo grep -nE '^[[:space:]]*(Include|Match|PermitRootLogin|PasswordAuthentication|KbdInteractiveAuthentication|PubkeyAuthentication)' /etc/ssh/sshd_config
sudo ls -l /etc/ssh/sshd_config.d/
```

Очаквано: непразен predi.txt и глобален Include за sshd_config.d/*.conf преди конфликтни настройки. При липсващ Include, неразгледан Match или конфликтни по-ранни стойности спри и актуализирай процедурата с преподавателя. Архивът в /root съдържа чувствителни ключове и не се качва в Git.

## 3. Настройка и синтаксис - на srv

```bash
printf '%s\n' 'PermitRootLogin no' 'PasswordAuthentication no' 'KbdInteractiveAuthentication no' 'PubkeyAuthentication yes' | sudo tee /etc/ssh/sshd_config.d/10-uld04.conf >/dev/null
sudo chmod 644 /etc/ssh/sshd_config.d/10-uld04.conf
sudo /usr/sbin/sshd -t
```

Очаквано: sshd -t няма изход и връща 0. При грешка не прилагай и изпълни rollback.

## 4. Ефективни стойности и прилагане - на srv

```bash
read -r CLIENT_IP CLIENT_PORT SERVER_IP SERVER_PORT <<< "$SSH_CONNECTION"
export CLIENT_IP SERVER_IP SERVER_PORT
sudo /usr/sbin/sshd -T -C "user=$(id -un),addr=$CLIENT_IP,host=$CLIENT_IP,laddr=$SERVER_IP,lport=$SERVER_PORT" | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication) '
sudo /usr/sbin/sshd -T -C "user=root,addr=$CLIENT_IP,host=$CLIENT_IP,laddr=$SERVER_IP,lport=$SERVER_PORT" | grep -E '^(permitrootlogin|passwordauthentication|kbdinteractiveauthentication|pubkeyauthentication) '
```

Очаквано и за двата профила: permitrootlogin no; passwordauthentication no; kbdinteractiveauthentication no; pubkeyauthentication yes. При друго спри. При Match Host IP в host не замества анализа на реалното разрешено име на клиента.

```bash
sudo systemctl reload ssh
sudo systemctl is-active ssh
{ date -Is; hostname; sudo /usr/sbin/sshd -T; sudo /usr/sbin/sshd -T -C "user=$(id -un),addr=$CLIENT_IP,host=$CLIENT_IP,laddr=$SERVER_IP,lport=$SERVER_PORT"; sudo /usr/sbin/sshd -T -C "user=root,addr=$CLIENT_IP,host=$CLIENT_IP,laddr=$SERVER_IP,lport=$SERVER_PORT"; sudo systemctl status ssh --no-pager; } > "$D/sled.txt" 2>&1
```

Очаквано: reload е успешен, услугата е active, резервната сесия работи. sshd -T прочита конфигурацията от файловете, не паметта на работещия демон: изискват се и новите връзки по-долу.

## 5. Нови тестове - от gw

Отвори НОВ терминал на gw и въведи отново SRV и ADMIN чрез read командите. Не използвай терминал, който в момента е SSH сесия върху srv.

```bash
export G="$HOME/artefakti/s06-soa-procedura"
mkdir -p "$G"
{ date -Is; ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no "$ADMIN@$SRV" true; printf 'positive_rc=%s\n' "$?"; } > "$G/positive.txt" 2>&1
cat "$G/positive.txt"
```

Очаквано: positive_rc=0. Провери и sudo -v в нов интерактивен вход с ключ.

```bash
{
 date -Is
 echo 'TEST password'
 ssh -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PubkeyAuthentication=no -o KbdInteractiveAuthentication=no -o PreferredAuthentications=password "$ADMIN@$SRV" true
 printf 'password_rc=%s\n' "$?"
 echo 'TEST keyboard-interactive'
 ssh -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PubkeyAuthentication=no -o PasswordAuthentication=no -o PreferredAuthentications=keyboard-interactive "$ADMIN@$SRV" true
 printf 'keyboard_rc=%s\n' "$?"
 echo 'TEST root'
 ssh -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes -o ControlMaster=no -o ControlPath=none -o ConnectTimeout=10 -o PreferredAuthentications=publickey -o PasswordAuthentication=no -o KbdInteractiveAuthentication=no root@"$SRV" true
 printf 'root_rc=%s\n' "$?"
} > "$G/otkaz.txt" 2>&1
cat "$G/otkaz.txt"
```

Очаквано: отказ за трите теста, обикновено Permission denied (publickey), код 255. Timeout, connection refused и host-key грешка не доказват забранена автентикация. Ако се поиска отдалечена парола, прекрати и запиши отклонение. Не изключвай проверката на host ключа.

Отказ с ключ, който никога не е бил разрешен за root, сам по себе си не доказва PermitRootLogin no. Съпоставя се с root профила от sshd -T -C и журнала. Процедурата не добавя нов root ключ само за теста.

## 6. Журнал и копиране

На srv след тестовете:

```bash
sudo journalctl -u ssh --since '30 min ago' --no-pager > "$D/journal.txt"
```

На gw:

```bash
scp -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes "$ADMIN@$SRV:artefakti/s06-soa-procedura/predi.txt" "$G/"
scp -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes "$ADMIN@$SRV:artefakti/s06-soa-procedura/sled.txt" "$G/"
scp -i ~/.ssh/mis_h06 -o IdentitiesOnly=yes "$ADMIN@$SRV:artefakti/s06-soa-procedura/journal.txt" "$G/"
```

Очаквано: пет непразни файла за същия srv и тестов период. При празен журнал провери logging backend с преподавателя.

## При отклонение

При грешка, различни стойности, неуспешен разрешен вход или успешен забранен вход спри веднага. Запази резултата в deviation.txt. Уведоми DevOps инженера/преподавателя до 15 минути. Върни промяната от резервната сесия. Статусът не става внедрен преди успешни тестове.

## Връщане назад

От резервната сесия или конзолата на srv:

```bash
sudo rm -- /etc/ssh/sshd_config.d/10-uld04.conf
sudo /usr/sbin/sshd -t && sudo systemctl reload ssh
sudo systemctl is-active ssh
```

Очаквано: active и предишното поведение на новите връзки. Ако няма достъп, използвай VirtualBox конзолата; при невъзможност върни snapshot pre-h06. Връщането на snapshot отменя и други по-късни VM промени. Запази доказателствата извън VM предварително.

## Запис и приемане

Реалните predi.txt, sled.txt, otkaz.txt, positive.txt и journal.txt се поставят в artefakti/s06-soa-procedura/ на клонираното хранилище. Предложено съхранение: 12 месеца; очаква одобрение. Не качвай частни ключове, пароли или /root/ssh-pre-h06.tgz. След peer review отрази само доказания обхват в SoA: SSH не доказва цялостна фирмена реализация на 8.2, 8.9 и 5.37.

## Проверка от съученик

| Поле | Стойност |
| --- | --- |
| Изпълнил и дата | Предстои |
| Засякъл на стъпка | Не е проверено |
| Неясно | Не е проверено |
| Положителен тест | Няма реален резултат |
| Отрицателен тест | Няма реален резултат |
| Поправено след проверката | Предстои |
