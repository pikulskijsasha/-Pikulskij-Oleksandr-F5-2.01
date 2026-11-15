<!--
Бланк звіту до лабораторної роботи «TLS та сертифікати».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ | Пікульський Олександр |
| Група | F5 2.01 |
| Ідентифікатор | cs-201-16 |
| Дата виконання | 15.11.2026 |
| Середовище | GitHub Codespaces (Docker) |
| Версія OpenSSL | OpenSSL 3.0.13 30 Jan 2024 (Library: OpenSSL 3.0.13 30 Jan 2024) |

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Віддалений сервер | front.cs-201-16.lab  |
| Імена для сертифіката | www.cs-201-16.lab, app.cs-201-16.lab |
| Ваш сервер (IP) | 10.53.1.1 |

Стенд під час виконання завдань не перезапускався.

## Завдання 1. HTTP і HTTPS у дампі трафіку

Вивід кроку 1.1:

```text
server: web
Host: front.cs-201-16.lab
SNI:
protocol: HTTP/1.1
client: 10.53.1.1

10 packets captured
10 packets received by filter
0 packets dropped by kernel

GET / HTTP/1.1
Host: front.cs-201-16.lab
server: web
Host: front.cs-201-16.lab
```

Вивід кроку 1.2:

```text
server: web
Host: front.cs-201-16.lab
SNI: front.cs-201-16.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1

23 packets captured
23 packets received by filter
0 packets dropped by kernel

grep -a -c -E 'GET /|server: web' t1-https.txt
0

grep -a -o 'front.cs-201-16.lab' t1-https.txt | wc -l
1
```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` | Так | Ні |
| заголовок `Host` | Так | Ні |
| тіло відповіді (`server: web`) | Так | Ні |
| ім'я віддаленого сервера (кількість входжень) | 2 | 1 |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?

Ім'я сервера передається в повідомленні ClientHello, у розширенні server_name (SNI). Воно дозволяє серверу визначити, до якого сайту звертається клієнт.

2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?

Тому що HTTP не використовує TLS і не передає SNI. HTTPS використовує TLS, тому ім'я сервера передається через SNI під час встановлення захищеного з'єднання.

3. Рядок `protocol:` показує HTTP/1.1 для запиту по HTTP і HTTP/2.0 для запиту по HTTPS, хоча обидва запити виконано однією командою `curl` без додаткових ключів. Що визначило версію HTTP у другому випадку?

Версію HTTP визначило розширення ALPN під час TLS-рукостискання. Клієнт і сервер узгодили використання HTTP/2, тому HTTPS-запит працював за цим протоколом.

## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```text
Sent Record
    Content Type = Handshake (22)
    ClientHello, Length=338
Received Record
    Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
    Content Type = ChangeCipherSpec (20)
Received Record
    Content Type = ApplicationData (23)
    Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
    Content Type = ApplicationData (23)
    Inner Content Type = Handshake (22)
    Certificate, Length=986
Received Record
    Content Type = ApplicationData (23)
    Inner Content Type = Handshake (22)
    CertificateVerify, Length=74
Received Record
    Content Type = ApplicationData (23)
    Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
    Content Type = ChangeCipherSpec (20)
Sent Record
    Content Type = ApplicationData (23)
    Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
    Content Type = ApplicationData (23)
    Inner Content Type = Alert (21)

New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

Вивід кроку 2.2 (TLS 1.2):

```text
Sent Record
    Content Type = Handshake (22)
    ClientHello, Length=225
Received Record
    Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
    Content Type = Handshake (22)
    Certificate, Length=981
Received Record
    Content Type = Handshake (22)
    ServerKeyExchange, Length=112
Received Record
    Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
    Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
    Content Type = ChangeCipherSpec (20)
Sent Record
    Content Type = Handshake (22)
    Finished, Length=12
Received Record
    Content Type = Handshake (22)
    NewSessionTicket, Length=214
Received Record
    Content Type = ChangeCipherSpec (20)
Received Record
    Content Type = Handshake (22)
    Finished, Length=12
Received Record
    Content Type = ApplicationData (23)
Sent Record
    Content Type = Alert (21)

New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

<!-- Один рядок на кожен запис з виводу 2.1, у тому самому порядку. Напрям: К→С або С→К. -->

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
|---|---|---|---|---|---|
| 1 | К→С | Handshake (22) | — | ClientHello | 334 |
| 2 | С→К | Handshake (22) | — | ServerHello | 122 |
| 3 | С→К | ChangeCipherSpec (20) | — | ChangeCipherSpec | 1 |
| 4 | С→К | ApplicationData (23) | Handshake (22) | EncryptedExtensions | 36 |
| 5 | С→К | ApplicationData (23) | Handshake (22) | Certificate | 1007 |
| 6 | С→К | ApplicationData (23) | Handshake (22) | CertificateVerify | 95 |
| 7 | С→К | ApplicationData (23) | Handshake (22) | Finished | 69 |
| 8 | К→С | ChangeCipherSpec (20) | — | ChangeCipherSpec | 1 |
| 9 | К→С | ApplicationData (23) | Handshake (22) | Finished | 69 |
| 10 | К→С | ApplicationData (23) | Alert (21) | Alert | 19 |

Вивід кроків 2.3 і 2.4:

```text
extension_type=server_name(0), length=24
    0000 - 00 16 00 00 13 66 72 6f-6e 74 2e 63 73 2d 32
    000f - 30 31 2d 31 36 2e 6c 61-62
extension_type=ec_point_formats(11), length=4

TLS 1.3:
24 packets captured
24 packets received by filter
0 packets dropped by kernel
TLS 1.3: 0

TLS 1.2:
21 packets captured
21 packets received by filter
0 packets dropped by kernel
TLS 1.2: 2
```

| Показник | TLS 1.3 | TLS 1.2 |
|---|---|---|
| Довжина ClientHello, байт | 338 | 225 |
| Номер першого запису, тип якого спостерігач не бачить | № 4 | Немає: зовнішній тип запису видно, але вміст зашифрований після ChangeCipherSpec |
| Кількість RTT від ClientHello до першого HTTP-запиту | 1 RTT | 2 RTT |
| Чи видно сертифікат у дампі (результат 2.4) | Ні, 0 входжень | Так, 2 входження |
| Обраний протокол ALPN | h2 (HTTP/2) | h2 (HTTP/2) |


1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?

Шифрування починається із запису № 4. Це видно за рядками `Content Type = ApplicationData (23)` та `Inner Content Type = Handshake (22)`. Зовнішній спостерігач бачить тип ApplicationData, але не може визначити справжній тип зашифрованого повідомлення.

2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.

ClientHello у TLS 1.3 довший, тому що містить додаткові дані для узгодження захищеного з'єднання. Зокрема, розширення `key_share` дозволяє клієнту відразу передати дані для обміну ключами. Завдяки цьому рукостискання TLS 1.3 потребує на один RTT менше.

3. Довжина розширення `server_name` у виводі 2.3 становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?

Розширення складається з 2 байтів довжини списку імен, 1 байта типу імені, 2 байтів довжини самого імені та символів доменного імені. У моєму випадку `front.cs-201-16.lab` містить 19 символів. Тому довжина становить 2 + 1 + 2 + 19 = 24 байти. Це підтверджується рядком `extension_type=server_name(0), length=24`.

4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?

Запис ChangeCipherSpec у TLS 1.3 використовується для сумісності зі старими мережевими пристроями. Він не змінює ключі шифрування і не впливає на саме рукостискання, але допомагає уникнути проблем із пристроями, які очікують таке повідомлення.

## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```text
збережено /lab/tls/acme-root.crt

subject=CN = Pebble Root CA 4d3dc5
notBefore=Nov 15 13:14:47 2026 GMT
notAfter=Nov 15 13:14:47 2056 GMT

curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Вивід кроку 3.3 (останні рядки `lego` і рядки журналу nginx):

```text
2026/11/15 13:52:07 [INFO] [app.cs-201-16.lab] acme: use http-01 solver
2026/11/15 13:52:07 [INFO] [www.cs-201-16.lab] acme: use http-01 solver
2026/11/15 13:52:07 [INFO] [app.cs-201-16.lab] acme: Trying to solve HTTP-01
2026/11/15 13:52:13 [INFO] [app.cs-201-16.lab] The server validated our request
2026/11/15 13:52:13 [INFO] [www.cs-201-16.lab] acme: Trying to solve HTTP-01
2026/11/15 13:52:18 [INFO] [www.cs-201-16.lab] The server validated our request
2026/11/15 13:52:18 [INFO] [www.cs-201-16.lab, app.cs-201-16.lab] acme: Validations succeeded; requesting certificates
2026/11/15 13:52:18 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/15 13:52:18 [INFO] [www.cs-201-16.lab] Server responded with a certificate.
```

Журнал nginx (довгі токени запитів тут скорочено):

```text
10.53.0.10 - - [15/Nov/2026:13:52:07 +0000] "GET /.well-known/acme-challenge/… HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [15/Nov/2026:13:52:13 +0000] "GET /.well-known/acme-challenge/… HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
```
 
Вивід кроку 3.4:

```text
subject=CN = www.cs-201-16.lab
issuer=CN = Pebble Intermediate CA 7fbd3f
notBefore=Nov 15 13:52:18 2026 GMT
notAfter=Nov 15 13:52:17 2031 GMT
serial=27437E330A99685C
X509v3 Subject Alternative Name:
    DNS:www.cs-201-16.lab, DNS:app.cs-201-16.lab

subject=CN = Pebble Intermediate CA 7fbd3f
issuer=CN = Pebble Root CA 4d3dc5
```

Вивід кроку 3.5:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/15 13:55:20 [notice] 112#112: signal process started

curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.

Certificate chain
 0 s:CN = www.cs-201-16.lab
   i:CN = Pebble Intermediate CA 7fbd3f
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 15 13:52:18 2026 GMT; NotAfter: Nov 15 13:52:17 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)
```

Вивід кроку 3.6:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/15 13:56:44 [notice] 120#120: signal process started

server: cs-201-16 (https)
Host: www.cs-201-16.lab
SNI: www.cs-201-16.lab
protocol: TLSv1.3

Certificate chain
 0 s:CN = www.cs-201-16.lab
   i:CN = Pebble Intermediate CA 7fbd3f
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 15 13:52:18 2026 GMT; NotAfter: Nov 15 13:52:17 2031 GMT
 1 s:CN = Pebble Intermediate CA 7fbd3f
   i:CN = Pebble Root CA 4d3dc5
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 15 13:14:47 2026 GMT; NotAfter: Nov 15 13:14:47 2056 GMT
---
Verify return code: 0 (ok)

server: cs-201-16 (https)
Host: app.cs-201-16.lab
SNI: app.cs-201-16.lab
protocol: TLSv1.3
```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера | Pebble Intermediate CA 7fbd3f |
| Видавець сертифіката видавця | Pebble Root CA 4d3dc5 |
| Імена в SAN | www.cs-201-16.lab, app.cs-201-16.lab |
| Строк дії (від – до) | 15.11.2026 13:52:18 GMT – 15.11.2031 13:52:17 GMT |
| Тип ключа сервера і алгоритм підпису | EC, 256 біт (id-ecPublicKey), RSA-SHA256 |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 | 3.5 — 1 сертифікат; 3.6 — 2 сертифікати |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.

Контроль над обома іменами перевірив тестовий центр сертифікації Pebble за допомогою HTTP-01. Він звернувся до спеціальних файлів на моєму сервері через HTTP.

У виводі є повідомлення `The server validated our request` для обох імен, а також `Validations succeeded; requesting certificates`. У журналі nginx запити до `/.well-known/acme-challenge/` отримали код `200`, що підтверджує успішну перевірку.

2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?

Тому що сервер передавав лише свій сертифікат без проміжного сертифіката центру Pebble Intermediate CA. Клієнт не зміг побудувати повний ланцюжок довіри до кореневого сертифіката.

Це підтверджує помилка `Verify return code: 21 (unable to verify the first certificate)`. Після додавання проміжного сертифіката перевірка завершилася успішно — `Verify return code: 0 (ok)`.

3. У виводі ланцюжка для сертифіката сервера вказано `id-ecPublicKey` і `RSA-SHA256`. Що означає кожне з цих значень і чому вони не суперечать одне одному?

`id-ecPublicKey` означає, що сервер використовує відкритий ключ на основі еліптичних кривих (EC). `RSA-SHA256` означає, що центр сертифікації підписав сертифікат алгоритмом RSA із хешуванням SHA-256.

Вони не суперечать одне одному, тому що перше значення описує ключ сервера, а друге — алгоритм, яким центр сертифікації підписав його сертифікат.

4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?

Після перезапуску стенда центр Pebble створює нові кореневий і проміжний ключі. Через це раніше виданий сертифікат сервера більше не буде перевірятися за новим кореневим сертифікатом.

Для відновлення роботи потрібно повторно отримати та встановити сертифікат.

У реальній інфраструктурі подібна ситуація може виникнути після заміни центру сертифікації або зміни кореня довіри, коли старі сертифікати більше не підтверджуються.

## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```text
=== f1.cs-201-16.lab
curl: (60) SSL certificate problem: certificate has expired
Certificate chain
 0 s:CN = f1.cs-201-16.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Apr 29 13:14:46 2026 GMT; NotAfter: Oct 26 13:14:46 2026 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 15 13:14:45 2026 GMT; NotAfter: Nov 14 13:14:45 2031 GMT
---
Verify return code: 10 (certificate has expired)

=== f2.cs-201-16.lab
curl: (60) SSL: no alternative certificate subject name matches target host name 'f2.cs-201-16.lab'
Certificate chain
 0 s:CN = intra.cs-201-16.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 15 13:14:46 2026 GMT; NotAfter: Feb 13 13:14:46 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 15 13:14:45 2026 GMT; NotAfter: Nov 14 13:14:45 2031 GMT
---
Verify return code: 0 (ok)

=== f3.cs-201-16.lab
server: web, f3
Certificate chain
 0 s:CN = f3.cs-201-16.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 15 13:14:46 2026 GMT; NotAfter: Feb 13 13:14:46 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 15 13:14:45 2026 GMT; NotAfter: Nov 14 13:14:45 2031 GMT
---
Verify return code: 0 (ok)

=== f4.cs-201-16.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f4.cs-201-16.lab
   i:O = Foreign Org, CN = Foreign Intermediate CA
   v:NotBefore: Nov 15 13:14:46 2026 GMT; NotAfter: Feb 13 13:14:46 2027 GMT
 1 s:O = Foreign Org, CN = Foreign Intermediate CA
   i:O = Foreign Org, CN = Foreign Root CA
   v:NotBefore: Nov 15 13:14:46 2026 GMT; NotAfter: Nov 14 13:14:46 2031 GMT
---
Verify return code: 20 (unable to get local issuer certificate)

=== f5.cs-201-16.lab
curl: (60) SSL certificate problem: self-signed certificate
Certificate chain
 0 s:CN = f5.cs-201-16.lab
   i:CN = f5.cs-201-16.lab
   v:NotBefore: Nov 15 13:14:46 2026 GMT; NotAfter: Feb 13 13:14:46 2027 GMT
---
Verify return code: 18 (self-signed certificate)
```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 | certificate has expired | 10 (certificate has expired) | Перевірка строку дії сертифіката | `NotAfter: Oct 26 13:14:46 2026 GMT` | Оновити прострочений сертифікат |
| f2 | no alternative certificate subject name matches target host name | 0 (ok) | Перевірка відповідності імені сервера сертифікату | `no alternative certificate subject name matches target host name 'f2.cs-201-16.lab'` | Отримати сертифікат із правильним іменем у SAN |
| f3 | Немає, сервер відповів успішно | 0 (ok) | Усі перевірки пройдені | `server: web, f3` та `Verify return code: 0 (ok)` | Нічого змінювати не потрібно |
| f4 | unable to get local issuer certificate | 20 (unable to get local issuer certificate) | Перевірка довіри до центру сертифікації | `i:O = Foreign Org, CN = Foreign Root CA` та `Verify return code: 20` | Отримати сертифікат від довіреного центру сертифікації |
| f5 | self-signed certificate | 18 (self-signed certificate) | Перевірка довіри до сертифіката | `Verify return code: 18 (self-signed certificate)` | Замінити самопідписаний сертифікат на сертифікат від довіреного центру |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність: сервер не надсилає проміжний сертифікат чи сертифікат виданий центром, якому клієнт не довіряє. Чим ці дві несправності відрізняються у виводі `openssl s_client`?

У моєму варіанті цю помилку має сервер f4. Його сертифікат виданий центром, якому клієнт не довіряє.

У виводі `openssl s_client` присутній проміжний сертифікат `Foreign Intermediate CA`, який підписаний `Foreign Root CA`. Але цей кореневий центр не входить до довірених для клієнта.

Якщо сервер не надсилає проміжний сертифікат, його не буде в ланцюжку `Certificate chain`. Якщо центр недовірений, проміжний сертифікат може бути присутнім, але перевірка все одно завершиться помилкою.

2. Якщо серед ваших серверів є сервер з невідповідним ім'ям, чому `openssl s_client` повертає для нього `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку? Якщо такого сервера немає, дайте відповідь для загального випадку.

У моєму варіанті це сервер f2.

Команда `openssl s_client` без параметра `-verify_hostname` перевіряє ланцюжок сертифікатів, але не перевіряє відповідність імені сервера. Тому вона показує `Verify return code: 0 (ok)`.

Програма `curl` додатково перевіряє, чи відповідає ім'я сервера іменам у сертифікаті. Для f2 ця перевірка не пройшла, тому `curl` повідомив про помилку.

3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?

На запит `curl -k` відповіли б усі п'ять серверів: f1, f2, f3, f4 і f5, оскільки параметр `-k` вимикає перевірку сертифіката та відповідності імені сервера.

При цьому шифрування з'єднання зберігається, але клієнт більше не може надійно підтвердити справжність сервера. Це створює ризик підключення до підробленого сервера.

Параметр `--cacert`, навпаки, дозволяє вказати довірений кореневий сертифікат без вимкнення перевірок безпеки.

## Завдання 5. Ціна нового з'єднання

**Прогноз** (записати до вимірювання). Скільки RTT мине до завершення кожного етапу:

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | 2 RTT | 2 RTT | 0 RTT |
| `time_appconnect` | 3 RTT | 4 RTT | 0 RTT |
| `time_starttransfer` | 4 RTT | 5 RTT | 1 RTT |

Вивід кроків 5.2 і 5.3:

Крок 5.2. Вимірювання нових з'єднань TLS 1.3:

```text
0.146115 0.236492 0.290697 0.336257 0.336316 2 1
0.045451 0.090595 0.139148 0.184644 0.184703 2 1
0.045491 0.090637 0.139530 0.185043 0.185101 2 1
```

Крок 5.2. Вимірювання нових з'єднань TLS 1.2:

```text
0.045481 0.090629 0.184638 0.229998 0.230066 2 1
0.045866 0.090998 0.185087 0.230433 0.230489 2 1
0.045417 0.090554 0.184275 0.229731 0.229799 2 1
```

Крок 5.3. Три запити з повторним використанням з'єднання:

```text
0.090778 0.135927 0.184598 0.229957 0.230015 2 1
0.000018 0.000000 0.000000 0.045308 0.045630 2 0
0.000037 0.000000 0.000000 0.045442 0.045509 2 0
```

Значення в мілісекундах. Для нових з'єднань використано медіану трьох вимірювань.

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове | 45.491 | 90.637 | 139.530 | 185.043 | 185.101 | 1 |
| TLS 1.2, нове | 45.481 | 90.629 | 184.638 | 229.998 | 230.066 | 1 |
| 5.3, перший запит | 90.778 | 135.927 | 184.598 | 229.957 | 230.015 | 1 |
| 5.3, другий запит | 0.018 | 0 | 0 | 45.308 | 45.630 | 0 |
| 5.3, третій запит | 0.037 | 0 | 0 | 45.442 | 45.509 | 0 |

RTT, визначений вимірюванням, і з якого значення його визначено: RTT ≈ 45 мс (0.045 с).

Це значення визначено за `time_starttransfer` другого запиту в кроці 5.3, який становив 0.045308 с. Оскільки з'єднання вже було встановлене, цей час приблизно відповідає одному RTT.

Результат також підтверджується різницею між `time_connect` і `time_namelookup` для TLS 1.3:

90.637 - 45.491 = 45.146 мс ≈ 1 RTT.

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?

Для TLS 1.3 отримано:

- `time_namelookup` ≈ 1 RTT;
- `time_connect` ≈ 2 RTT;
- `time_appconnect` ≈ 3 RTT;
- `time_starttransfer` ≈ 4 RTT;
- `time_total` ≈ 4 RTT.

Для TLS 1.2 отримано:

- `time_namelookup` ≈ 1 RTT;
- `time_connect` ≈ 2 RTT;
- `time_appconnect` ≈ 4 RTT;
- `time_starttransfer` ≈ 5 RTT;
- `time_total` ≈ 5 RTT.

Для першого запиту в кроці 5.3 отримано приблизно 2, 3, 4, 5 і 5 RTT відповідно.

Для другого та третього запитів через уже встановлене з'єднання `time_connect` і `time_appconnect` дорівнюють нулю, а `time_starttransfer` і `time_total` становлять приблизно 1 RTT.

Невеликі відхилення від цілих RTT пояснюються обробкою запитів сервером, роботою операційної системи та мережевими затримками.

Перше вимірювання TLS 1.3 відрізнялося сильніше: `time_total` становив 0.336316 с, що помітно більше за наступні результати. Це могло бути пов'язано з додатковими затримками під час визначення адреси або встановлення з'єднання.

2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?

Тому що навіть за наявності запису в кеші клієнт повинен надіслати DNS-запит до резолвера та отримати відповідь. На це витрачається приблизно один RTT.

Час міг би бути приблизно вдвічі більшим, якби для визначення адреси знадобився ще один послідовний мережевий обмін, наприклад додатковий запит резолвера до іншого DNS-сервера.

3. Сервіс виконує 200 послідовних запитів до цього сервера. Розрахуйте, скільки часу займе лише очікування мережі, якщо кожен запит встановлює нове з'єднання TLS 1.3, нове з'єднання TLS 1.2 і якщо всі запити йдуть через одне з'єднання.

За вимірюваннями один RTT становить приблизно 45 мс.

**Варіант 1. Кожен запит встановлює нове з'єднання TLS 1.3.**

Один запит займає приблизно 4 RTT.

200 × 4 × 45 = 36000 мс = 36 с.

**Варіант 2. Кожен запит встановлює нове з'єднання TLS 1.2.**

Один запит займає приблизно 5 RTT.

200 × 5 × 45 = 45000 мс = 45 с.

**Варіант 3. Усі 200 запитів виконуються через одне з'єднання TLS 1.3.**

Перший запит потребує 4 RTT, а кожен із наступних 199 запитів — приблизно 1 RTT.

(4 + 199) × 45 = 9135 мс = 9.135 с.

Отже, повторне використання одного TLS-з'єднання значно скорочує час очікування. Найкращий результат серед розглянутих варіантів — приблизно 9.1 секунди замість 36 або 45 секунд.

## Висновки

**Що виявилося несподіваним. Результат, який не збігся з очікуванням, і рядок виводу, де це видно:**

Несподіваним виявилося те, що перше з'єднання TLS 1.3 було значно повільнішим за наступні.

У першому вимірюванні `time_total = 0.336316 с`, а в наступних двох — `0.184703 с` та `0.185101 с`.

Також перший `time_namelookup = 0.146115 с` був значно більшим за очікувані приблизно 0.045 с.

Це показало, що навіть за однакових умов окремі вимірювання можуть відрізнятися, тому для порівняння краще використовувати медіану.

**Яке питання залишилося без відповіді. Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено і що з'ясувати не вдалося:**

Не вдалося точно визначити причину великої затримки під час першого вимірювання TLS 1.3.

Це стосується рядка, де `time_namelookup = 0.146115 с`, хоча наступні результати становили `0.045451 с` та `0.045491 с`.

Я порівняв усі три вимірювання TLS 1.3 та результати TLS 1.2. Встановив, що збільшення часу не повторювалося в наступних запитах.

Однак без окремого аналізу DNS-трафіку під час першого вимірювання не вдалося з'ясувати, чи затримка виникла через роботу резолвера, мережі або самого середовища стенда.

## Декларація використання ШІ

Використовувалися інструменти ШІ: **так**.

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | ChatGPT (OpenAI) | поясни рядки виводів | Використано допомогу у поясненні результатів виводів рядків у консолі, поясненні помилок |
| 2 | ChatGPT (OpenAI) | все правильно поки бо зі скриншотів ? | Використано допомогу в перевірці результатів, поясненні помилок TLS і сертифікатів та аналізі вимірювань. Результати отримано самостійно в GitHub Codespaces. |
