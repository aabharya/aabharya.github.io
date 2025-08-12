---
author: Ali Abharya
pubDatetime: 2025-07-10T16:38:17.169Z
modDatetime: 2025-07-10T16:28:17.169Z
title: سوکت‌ها در سطح کرنل و پایتون
title_seo: Socket API in Kernel and Python
slug: socket-api
featured: true
draft: false
tags:
  - socket
  - tcp
  - kernel
  - python
  - http

description: سوکت‌ها در سطح کرنل و پایتون
---

# سوکت‌ها در سطح کرنل و پایتون

از چند روز پیش که دور دوم مطالعه کتاب **HTTP: The Definitive Guide** رو شروع کردم یاد جمله حسین‌ناصر افتادم که میگه:

> من با هیچ ابزاری کار نمی‌کنم مگر اینکه ۱۰۰ درصد جزییات داخلیش رو عمیقا درک کرده باشم.

همین شد که به بخش سوکت‌ها که رسیدیم تصمیم گرفتم یکبار دستور اجرای سرور HTTP و همینطور هندل کردن یک درخواست HTTP رو توی پایتون دیباگ کنم. هدفم هم این بود internal کتابخونه استاندارد پایتون و همینطور اینترفیس WSGI رو بیشتر درک کنم به علاوه برام جالب بود که بدونم از سمت سیستم عامل هم این وسط چه اتفاقاتی میوفته؟

![http_dfg](@assets/images/http_dfg.jpeg)

خیلی خلاصه بخوام بگم اون بازیگر اصلی این وسط یه مفهومی هست به نام سوکت که هم سیستم‌عامل و هم کتابخونه استاندارد پایتون یه API برای اون دارن و در حقیقت اون چرخه request - response بین کلاینت و سرور از طریق همین سوکت هندل میشه.

این نوشته دو بخش داره:‌

- اول اینکه سوکت چی هست و در سطح کرنل چه عملیاتی میشه باهاش انجام داد؟

- دوم اینکه وقتی تو پایتون یه سرور HTTP اجرا می‌کنیم و یک درخواست از سمت کلاینت میاد تا پاسخ رو دریافت کنه چه فراخوانی‌هایی در پایتون انجام میشه و به تبع اون‌ها چه فراخوانی‌های سیستمی در سطح کرنل انجام میشه؟

---

## سوکت در سطح سیستم‌عامل

### 1️⃣ سوکت چی هست؟!

API سوکت‌ها در سطح کرنل ساختار داده لازم برای اندپوینت‌های TCPرو ایجاد می‌کنه و امکان میده که جریان داده‌ها خوانده و نوشته بشن. این API تمام جزئیات مربوط به فرآیند `handshaking` و سایر لایه‌های زیرساختی شبکه و جریان داده‌ها مربوط به بسته‌های IP رو پنهان می‌کنه.

در سطح سیستم‌عامل، سوکت رو میشه شبیه یک فایل در نظر گرفت که میشه روی اون عملیات خواندن و نوشتن انجام داد. وقتی یک سرور راه‌اندازی میشه کرنل ابتدا یک سوکت شنونده `(Listening Socket)` ایجاد می‌کنه که روی یک آدرس IP و پورت خاص (مثلاً 0.0.0.0:8000) آماده دریافت درخواست‌هاست.

این سوکت صرفاً مسئول دریافت درخواست‌های اتصال `(SYN)` هست و دادهٔ واقعی رو منتقل نمی‌کنه. به محض اینکه کلاینت درخواست اتصال میده، کرنل سیستم‌عامل یک سوکت جدید به نام سوکت اتصال `(Connection Socket)` می‌سازه که مختص همون کلاینت است. بنابراین هر کلاینت سوکت مستقل خودش یا `File Descriptor` خودش رو داره و داده‌هایش با سایر کلاینت‌ها تداخل پیدا نمی‌کنه.

![sct_act](@assets/images/socket_actions.png)

---

### 2️⃣ عملیات قابل انجام با سوکت در سطح کرنل

در چرخه درخواست پاسخ بین کلاینت و سرور bind و listen و accept و read و write پنج عملیات اصلی در این فرآیند هستند. در جدول زیر همه عملیات قابل انجام توسط API سوکت در سطح کرنل خلاصه شده‌اند:

| API سوکت در سطح کرنل          | توضیحات                                            |
| ----------------------------- | -------------------------------------------------- |
| (< parameters >)socket        | ایجاد یک سوکت جدید بدون نام و بدون اتصال           |
| (s, local IP : port )bind     | اختصاص یک شماره پورت و اینترفیس به سوکت            |
| (s, remote IP : port )connect | ایجاد یک اتصال TCP به سوکت و یک host و پورت remote |
| (..., s)listen                | تغییر وضعیت سوکت برای دریافت درخواست               |
| (s)accept                     | منتظر ماندن برای ایجاد یک اتصال به پورت            |
| (s, buffer, n)read            | خواندن n بایت از سوکت به داخل بافر                 |
| (s, buffer, n)write           | نوشتن n بایت از بافر به داخل سوکت                  |
| (s)close                      | بستن کامل اتصال TCP                                |
| (s, side)shutdown             | بستن فقط ورودی یا خروجی اتصال TCP                  |
| (..., s)getsockopt            | خواندن مقدار یک پیکربندی داخلی سوکت                |
| (..., s)setsockopt            | تغییر مقدار یک پیکربندی داخلی سوکت                 |

---

## سوکت در سطح کتابخانه استاندارد پایتون

### 1️⃣ اینترفیس WSGI

WSGI یا **Web Server Gateway Interface** یک استاندارد رسمی در پایتون برای برقراری ارتباط بین وب‌سرورها و فریم‌ورک‌ها یا وب‌اپلیکیشن‌های پایتونی هست. پیش از معرفی WSGI، هر وب‌سرور و فریم‌ورک روش خاص خودش رو برای تبادل داده پیاده‌سازی می‌کرد و این باعث ناسازگاری و پیچیدگی توسعه می‌شد. WSGI این مشکل رو با تعریف یک قرارداد ساده حل کرد:

وب‌سرور درخواست HTTP رو به شکل داده‌های ساخت‌یافته به اپلیکیشن منتقل می‌کنه و اپلیکیشن هم پاسخ HTTP رو به همان شکل استاندارد به سرور برمی‌گردونه.

این مدل، اپلیکیشن و سرور رو از هم تفکیک میکنه و به توسعه‌دهنده اجازه میده بدون وابستگی به نوع وب‌سرور، اپلیکیشن خودش رو پیاده‌سازی و اجرا کنه. مثلا وقتی با جنگو سرور لوکال رو راه‌اندازی می‌کنم کد زیر اجرا میشه که یه instance از کلاس `WSGIServer` رو ایجاد می‌کنه و بهش می‌گه که تمامی درخواست‌های کلاینت‌ها رو با `WSGIRequestHandler` پاسخ بده:

```python
# file: django/core/servers/basehttp.py
def run(addr, port, wsgi_handler, ipv6=False, threading=False, on_bind=None,server_cls=WSGIServer):
    server_address = (addr, port)
    if threading:
        httpd_cls = type("WSGIServer", (socketserver.ThreadingMixIn, server_cls), {})
    else:
        httpd_cls = server_cls
    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
    if on_bind is not None:
        on_bind(getattr(httpd, "server_port", port))
    if threading:
        httpd.daemon_threads = True
    httpd.set_app(wsgi_handler)
    httpd.serve_forever()

```

متدهایی که اینجا لازمه بیشتر درباره‌اش عمیق بشیم یکی همین `serve_forever` مربوط به `WSGIServer` هست و یکی دیگه هم داخل کلاس `WSGIRequestHandler` هست به نام `handle_one_request` ولی قبل این‌ها لازمه درباره سوکت‌ها در کتابخونه استاندارد پایتون یکمی بیشتر بدونیم .

### 2️⃣ سوکت‌ها در کتابخانه استاندارد پایتون

کتابخونه استاندارد `socket` پایتون تقریباً یه wrapper مستقیم روی همون API سطح کرنل برای سوکت‌هاست که عملا اسم متدهای پایتونی با اسم عملیات کرنلی مشابه و یکی هست. مثلا وقتی توی پایتون `()socket.socket` رو فراخوانی کنیم در اصل به سیستم‌عامل درخواست دادیم که یک سوکت جدید ساخته بشه که این کار معمولاً از طریق فراخوانی سیستمی `()socket` انجام میشه.

| API سوکت در سطح پایتون  | توضیحات                                 |
| ----------------------- | --------------------------------------- |
| ()bind                  | اتصال سوکت به یک آدرس و پورت            |
| (address)connect        | ایجاد یک برقراری اتصال TCP با سرور مقصد |
| (backlog)listen         | تغییر وضعیت سوکت برای دریافت درخواست    |
| ()accept                | ایجاد یک سوکت جدید برای هر کلاینت       |
| (buffersize, flags)recv | خواندن داده از سوکت                     |
| (data, flags)send       | ارسال داده به سوکت                      |
| ()close                 | بستن کامل اتصال TCP                     |
| (flag)shutdown          | بستن فقط ورودی یا خروجی اتصال TCP       |

```python
# file: django/core/servers/basehttp.py
def run(addr, port, wsgi_handler, ipv6=False, threading=False, on_bind=None,server_cls=WSGIServer):
    server_address = (addr, port)
    if threading:
        httpd_cls = type("WSGIServer", (socketserver.ThreadingMixIn, server_cls), {})
    else:
        httpd_cls = server_cls
    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
    if on_bind is not None:
        on_bind(getattr(httpd, "server_port", port))
    if threading:
        httpd.daemon_threads = True
    httpd.set_app(wsgi_handler)
    httpd.serve_forever()

```

---

### 3️⃣ نمونه سرور echo با کتابخانه استاندارد

کتابخونه استاندارد پایتون همون API کرنل رو در اختیار میگذاره یعنی همون عملکردها با همون اسامی. به صورت کلی میشه گفت فرایند مربوط به سوکت‌ها همیشه همین الگو رو داره که این الگو رو در مثال زیر توضیح می‌دم:

`socket → bind → listen → accept → send/recv → close`

البته قبلش بگم اینکه این مثال برای اجرا از `asyncio` استفاده کرده یا برای راه‌اندازی سرور `WSGI` از `ThreadingMixIn` استفاده میشه موضوعی هست که بعدا درباره‌اش توضیح میدم ولی مستقیما به موضوع سوکت مربوط نیست.

کد زیر یک سرور echo ساده هست و سرور هر چیزی که کلاینت براش می‌فرسته رو بهش بر‌می‌گردونه. بعد از اجراش میشه با telnet به لوکال‌هاست و پورت 9999 اون رو تست کرد.

```python

import asyncio
import socket
from asyncio import AbstractEventLoop


async def echo(connection: socket, loop: AbstractEventLoop) -> None:
    while data := await loop.sock_recv(connection, 1024):
        await loop.sock_sendall(connection, data)


async def listen_for_connection(server_socket: socket, loop: AbstractEventLoop):
    while True:
        connection, client_address = await loop.sock_accept(server_socket)
        connection.setblocking(False)
        print('Connection from', client_address)
        asyncio.create_task(echo(connection, loop))


async def main() -> None:
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server_address = ('127.0.0.1', 9999)
    server_socket.bind(server_address)
    server_socket.setblocking(False)
    server_socket.listen()
    await listen_for_connection(server_socket, asyncio.get_event_loop())


if __name__ == '__main__':
    asyncio.run(main())
```

با اجرای `socket.socket(socket.AF_INET, socket.SOCK_STREAM)` یه سوکت TCP ایجاد میشه و عملا `()socket` در سطح کرنل فراخوانی میشه. مقادیری که به عنوان آرگومان پاس داده میشن پارامترهای سوکت هستن: `AF_INET` یعنی آدرس‌ها از نوع `IPv4` باید باشن و `SOCK_STREAM` هم یعنی نوع سوکت از پروتکل TCP هست. بعد از اون سوکت به آدرس و پورت `bind` میشه و بعدش به حالت `listen` نغییر وضعیت میده که فراخوانی‌های کرنل و پایتون اون دقیقا هم نام هستن.

بعد از این مرحله برای اینکه درخواست‌های همزمان کلاینت‌ها همدیگه رو بلاک نکنن هندل کردن اون‌ها از طریق `event loop` کنترل میشه. متد `listen_for_connection` منتظر یه درخواست اتصال جدید می‌مونه و از طریق `loop.sock_accept` فراخوانی کرنل `accept` رو انجام میده. این کار به صورت `non-blocking` انجام میشه و سرور می‌تونه بلافاصله درخواست جدید یک کلاینت دیگه رو هم هندل کنه.

داخل متد `echo` یه حلقه بی‌نهایت هست که داده‌ها رو از کلاینت دریافت می‌کنه و همون‌داده‌ها رو به کلاینت بر‌میگردونه. متدهای `loop.sock_recv` و `loop.sock_sendall` عملا فراخوانی‌های مربوط به `recv` و `send` کرنل رو انجام میدن.

```shell
    telnet 127.0.0.1 9999
    Trying 127.0.0.1...
    Connected to 127.0.0.1.
    Escape character is '^]'.
    hello world!
    hello world!
```

### 4️⃣ فراخوانی‌های سمت پایتون در زمان پاسخ به درخواست HTTP

حالا می‌تونیم به سرور `WSGI` برگردیم. در زمان راه‌اندازی سرور `WSGI` در صورتی که آرگومان `threading` پاس داده شده باشه به سرور `WSGI` رفتار `socketserver.ThreadingMixIn` هم اضافه میشه. در حقیقت میشه اینطوری فرض کرد که برای هندل کردن هر درخواست یه `Thread` جدید درست میشه. ایده مشابهی که بالاتر مثالش رو با `asyncio` دیدیم البته اونجا `task` درست میشد. برای اینکه ساده‌تر بشه می‌تونیم فرض کنیم متد `()serve_forever` در سرور `WSGI` همون کاری رو میکنه که `listen_for_connection` در مثال سرور echo انجام میده یعنی با همون ایده که درخواست‌ها همدیگه رو بلاک نکنن.

---

## جمع‌بندی

درک جزئیات و فهم عمیق ابزارها و طریقه استفاده از اون‌ها دغدغه همیشگی یک **مهندس نرم‌افزار** هست. وقتی یک مفهوم را تا سطح عملکرد داخلی‌اش می‌شناسیم، ذهن ما می‌تونه اون را در یک چارچوب منطقی دسته‌بندی کنه: علت‌ها، فرآیندها و پیامدها.
اما اگه این لایه‌های زیرین ابزارها رو درک نکنیم در ارائه راه‌حل‌ها در نهایت وابستگی زیادی به فریمورک‌ها پیدا می‌کنیم و به اصطلاح نمی‌تونیم `out of the box` فکر کنیم چون اجازه دادیم فریمورک‌ها این جزییات رو از ما مخفی کنن و به جای ما فکر کنن و تصمیم بگیرند.
