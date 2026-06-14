# T2HASH CORE Analysis

## فارسی

خب، این ریپو برای این ساخته شده که ببینیم پشت این پروژه‌ای که با اسم‌های سنگین مثل **Layer-0 Assembly Core**، **UDP Tunnel**، **Anti-DPI** و خلاصه «پدر شبکه» معرفی شده، واقعاً چی خوابیده.

نتیجه؟
اون چیزی که به اسم core اسمبلی منتشر شده بود، در عمل یک فایل ELF مخفی‌کاری‌شده بود که آخرش می‌رسید به یک Bash installer.

یعنی خلاصه‌اش اینه:

> ادعا: Layer-0 Assembly Core
> واقعیت: Bash script قایم‌شده داخل ELF

خیلی هم شاعرانه.

## خلاصه ماجرا

فایل `t2hash-deploy` بررسی شد؛ هم static analysis، هم dynamic analysis، هم dump گرفتن موقع اجرا.

چیزی که در اومد این بود:

* فایل اصلی یک ELF stripped بود.
* داخلش `execvp` استفاده شده بود.
* فایل در نهایت `/bin/bash` را اجرا می‌کرد
* 
* payload واقعی یک Bash installer بود.
* این Bash installer بعداً خودش فایل Assembly داخل `/root` می‌سازد.
* یعنی Assembly وجود دارد، ولی نه اون‌جوری که تبلیغ شده بود.
* خود فایل منتشرشده یک wrapper/installer مخفی‌شده است، نه یک core شفاف و قابل بررسی.

## یافته‌های اصلی

موارد مهمی که از تحلیل بیرون آمد:

* فایل اصلی سورس شفاف ندارد.

* باینری obfuscated / packed شده بود.

* با `/bin/bash` اجرا می‌شود.

* اسکریپت مخفی‌شده داخلش بعد از dump قابل خواندن شد.

* اسکریپت recovered شده داخل مسیر زیر قرار دارد:

  recovered/recovered_t2hash.sh

* این اسکریپت پکیج‌های زیر را نصب می‌کند:

  nasm
  build-essential
  wget
  gzip

* از GitHub فایل `gost` را دانلود می‌کند.

* دانلود `gost` بدون checksum verification انجام می‌شود.

* داخل `/root` فایل‌های Assembly می‌سازد.

* با `nasm` و `ld` آن‌ها را build می‌کند.

* systemd service می‌سازد.

* سرویس‌ها را با دسترسی root اجرا می‌کند.

* مقدارهای network buffer را با `sysctl` تغییر می‌دهد.

* داخل Assembly از XOR ساده با `0x5A` استفاده شده.

## نکته خنده‌دار ولی مهم

اسم پروژه طوری چیده شده که انگار با یک شاهکار low-level طرفیم؛ ولی چیزی که کاربر دانلود می‌کند، یک ELF مخفی‌شده است که Bash installer را decode و اجرا می‌کند.

یعنی مسیر واقعی این است:

```
ELF wrapper
    ↓
decoded Bash installer
    ↓
writes Assembly files
    ↓
builds binaries
    ↓
creates systemd services
```

پس بله، یک جایی وسط کار Assembly هم هست، ولی اینکه یک Bash installer را داخل ELF قایم کنیم و اسمش را بگذاریم Layer-0 Core، بیشتر شبیه مارکتینگ فضایی است تا مهندسی شفاف.

## نگرانی‌های امنیتی

این فایل را نباید کورکورانه با root اجرا کرد، چون:

* payload واقعی را مخفی کرده بود.
* سورس قابل بررسی در نگاه اول ارائه نمی‌داد.
* بدون تأیید کاربر سرویس‌های systemd می‌سازد.
* فایل‌هایی داخل `/root` می‌نویسد.
* `gost` را بدون بررسی checksum دانلود می‌کند.
* سرویس‌ها را با root اجرا می‌کند.
* روی سرور ایران SOCKS5 روی پورت `8081` باز می‌کند.
* سرویس‌ها و فایل‌های قبلی با اسم مشابه را پاک می‌کند.
* هیچ احراز هویت جدی یا رمزنگاری واقعی در بخش UDP دیده نمی‌شود.
* XOR با `0x5A` رمزنگاری حساب نمی‌شود؛ بیشتر شبیه این است که به پکت‌ها عینک آفتابی زده باشند.

## آیا این بدافزار است؟

بر اساس اسکریپت recovered شده، نمی‌شود با قطعیت گفت این یک malware کلاسیک است.
چیزی مثل steal کردن SSH key، reverse shell واضح، miner یا cron backdoor مستقیم در اسکریپت دیده نشد.

اما این هم واقعیت دارد:

> این یک installer شفاف و قابل اعتماد هم نیست.

برچسب درست‌تر:

```
Suspicious obfuscated root installer
```

نه باید بی‌دلیل داد زد «ویروسه»، نه باید ساده‌لوحانه گفت «داداش اسمبلیه، اجرا کن بره».

## فایل‌های مدرک

مسیر فایل‌های evidence:

```
evidence/execvp-output.txt
evidence/findings-short.txt
evidence/sha256.txt
```

اسکریپت decode شده:

```
recovered/recovered_t2hash.sh
```

## جمع‌بندی 

اگر قرار است پروژه واقعاً فنی باشد، سورس را شفاف منتشر کن.
اگر Assembly نوشته‌ای، همان Assembly را داخل ریپو بگذار.
اگر installer داری، بگو installer است.
اگر Bash script است، اسمش را Bash script بگذار.

قایم کردن Bash installer داخل ELF و بعد با عنوان Layer-0 Assembly Core، مهندسی نیست؛ بیشتر شبیه این است که روی پراید، لوگوی سفینه ناسا بچسبانی.

---

## English

This repository documents a static and dynamic analysis of the published `t2hash-deploy` binary.

The project was advertised with heavy words like **Layer-0**, **Assembly Core**, **UDP tunneling**, and **Anti-DPI**.

After analysis, the reality was much simpler:

> Claim: Layer-0 Assembly Core
> Reality: An obfuscated ELF wrapper that decodes and runs a Bash installer

Poetic, but not exactly transparent engineering.

## Summary

The published binary was analyzed using static inspection, GDB runtime tracing, memory dumping, and string extraction.

The recovered payload shows that the distributed ELF is not a transparent Assembly core.
It is mainly an obfuscated wrapper that launches `/bin/bash` and executes a hidden Bash installer.

That Bash installer then writes Assembly source files, builds them, creates systemd services, downloads `gost`, and changes network settings.

## Main Findings

* The original file is a stripped ELF.
* It imports `execvp`.
* It launches `/bin/bash`.
* Runtime memory dumping revealed a hidden Bash installer.
* The recovered Bash script writes Assembly source files to `/root`.
* It installs `nasm`, `build-essential`, `wget`, and `gzip`.
* It downloads `gost` from GitHub without checksum verification.
* It creates systemd services as root.
* It applies network buffer sysctl changes.
* It uses a simple XOR `0x5A` operation in the generated Assembly code.

## Important Finding

The binary itself is not a transparent Assembly core.

The real flow is:

```
ELF wrapper
    ↓
decoded Bash installer
    ↓
generated Assembly files
    ↓
compiled binaries
    ↓
systemd services
```

So yes, Assembly exists somewhere in the chain.
But calling an obfuscated Bash installer a “Layer-0 Assembly Core” is a bit too much marketing sauce.

## Security Concerns

This installer should not be run blindly as root because it:

* Is obfuscated.
* Hides its real Bash payload.
* Downloads a binary without checksum verification.
* Creates root-level systemd services.
* Writes files into `/root`.
* Opens network services.
* Removes previous files and services with matching names.
* Does not provide strong authentication.
* Does not provide serious cryptographic protection.
* Uses XOR `0x5A`, which is not real encryption.

## Is it malware?

Based on the recovered script, this is not confirmed classic malware.
No obvious SSH key theft, miner, reverse shell, or cron backdoor was found in the recovered Bash script.

However, it is still not a clean or trustworthy installer.

A more accurate label would be:

```
Suspicious obfuscated root installer
```

## Recovered Files

Recovered Bash script:

```
recovered/recovered_t2hash.sh
```

Evidence files:

```
evidence/execvp-output.txt
evidence/findings-short.txt
evidence/sha256.txt
```

## Conclusion

If this project is supposed to be serious, publish the real source code clearly.

If it is Assembly, publish the Assembly.
If it is an installer, call it an installer.
If it is a Bash script, do not hide it inside an obfuscated ELF and call it a Layer-0 core.

Calling an obfuscated Bash installer “Layer-0 Assembly Core” is not engineering; it is marketing with a space helmet.
