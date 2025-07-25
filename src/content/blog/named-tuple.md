---
author: علی ابهریا
pubDatetime: 2024-02-15T09:13:38.039Z
modDatetime: 2024-02-15T09:13:38.039Z
title: Named Tuple در پایتون
slug: named-tuple
featured: false
draft: false
tags:
  - python
  - clean_code
description: مزیت‌های استفاده از named tuple
---

# Named Tuple در پایتون

## مقدمه

`namedtuple` یه ساختار داده‌ی سبک و کاربردیه که در ماژول `collections` از کتابخانه استاندارد پایتون در دسترس هست. این ساختار در واقع یه زیرکلاس از `tuple` هست که فیلدهاش نام‌گذاری‌ شدن و به همین خاطر نسبت به `tuple`های معمولی خواناتر هست. `namedtuple‌`مانند `immutabile` `tuple` هستن.

## نحوه استفاده

برای ایجاد یک `named tuple`، باید از تابع `namedtuple` در ماژول `collections` استفاده کرد. این تابع دو آرگومان دریافت می‌کنه:
`typename`: نام نوع داده به‌صورت رشته و `field names`: نام فیلدها که یک `iterable` از رشته‌ها می‌تونه باشه.

```python
  from collections import namedtuple

  # Define a named tuple named 'Person' with fields 'name' and 'age'
  Person = namedtuple('Person', ['name', 'age'])

  # Create an instance of the named tuple
  person_instance = Person(name='John', age=25)

  # Access fields using dot notation
  print(person_instance.name)  # Output: John
  print(person_instance.age)   # Output: 25
```

## مزایا

- خوانایی: استفاده از `namedtuple` خوانایی کد رو زیاد می‌کنه چون که فیلدها دارای نام‌های معنادار میشن.

- `immutability`: همانند `tuple`های معمولی، `namedtuple`ها هم تغییرناپذیر هستند. این ویژگی باعث حفظ یکپارچگی داده‌ها میشه و از تغییرات ناخواسته جلوگیری می‌کند.

- کارایی حافظه: `namedtuple`ها از حافظه به‌صورت بهینه استفاده می‌کنند، چون با زبان C پیاده‌سازی شدن و نسبت به بقیه کلاس‌های custom یا دیکشنری‌ها مصرف حافظه کمتری دارند.

- سازگاری: `namedtuple`ها با `tuple`های معمولی سازگارن یا به عبارت دیگه اینترفیس مشابهی دارن، بنابراین میشه اون‌ها را بدون مشکل در کدهایی که انتظار ساختار `tuple` دارن هم استفاده کرد.

## جمع‌بندی

`namedtuple`ها زمانی کاربرد دارن که به یک ساختار داده‌ ساده و سبک با مجموعه‌ای ثابت از فیلدها نیاز داریم. این ساختار برای مواردی مانند تنظیمات پیکربندی، نمایش رکوردهای یک مجموعه داده، یا هر موقعیتی که در آن `immutability` و دسترسی به فیلدها با نام مزیت محسوب میشه مناسب هست. `namedtuple‌`ها راهی تمیز، مختصر و خوانا برای تعریف ساختارهای داده‌ای در پایتون فراهم می‌کنن که در کنار مزایای `tuple`ها قابلیت نام‌گذاری‌ فیلدهاش رو هم داره.
