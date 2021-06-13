
---
weight: 30
bookFlatSection: true
title: "بررسی سانسور در ارتباط با سرور در لایه‌ی کاربرد"
description: "بررسی وجود سانسور و یا امکان دسترسی به سرور در لایه ی کاربرد. معنای لاگ های curl و تشخیص تحریم از فیلتر"
tags: ["سانسور", "Application", "RST injection", "null route", "فیلتر", "فیلترنت", "curl", "صفحه‌ی پیوندها", "HTTP", "HTTPS", "Application layer", "اینترنت", "تحریم", "خطای 403", "خطای 503", "لایه ی کاربرد", "non-transparent proxy", "Host name blocking", "بررسی سانسور اینترنت", "سانسور اینترنت"]
images:
- "/images/docs/measure-internet-censorship/Application/HTTPS-curl-exampledotcom-with-dnsgoogle-ip.png"
---

# بررسی سانسور در ارتباط در لایه ی کاربرد

در ارتباط عادی، سومین مرحله از بازدید یک سایت و یا استفاده از سرویس، ارتباط با برنامه ی نصب شده در سرور است. به عنوان مثال Apache، IIS، و یا هر سرویس ساده ی دیگر.

بهترین ابزار بررسی ارتباط با پروتکل های HTTP، HTTPS، MQTT و FTP و غیره، [curl](https://curl.se/download.html) است که اکثر سیستم عامل ها را پشتیبانی می کند.

برای بررسی روش هایی که یک سایت در لایه ی بالاتر از IP مسدود شده، اولین قدم بررسی ارتباط HTTP است. 
اما توجه به این نکته مهم است که در این بررسی نیز باید از IP اصلی استفاده کنیم، نه چیزی که سیستم سانسور به ما می دهد.


## ارتباط HTTP

در این سطح ما می خواهیم که DNS Hijacking در تست ما دخیل نباشد و در قدم اول فقط می خواهیم ببینیم که اگر از IP ی درست استفاده کنیم، باز هم ارتباط HTTP سانسور می شود؟ : 
<div dir="ltr">

`> .\curl -v --doh-url "https://mozilla.cloudflare-dns.com/dns-query" http://twitter.com/`
<br/>

![HTTP curl twitter with doh](/images/docs/measure-internet-censorship/Application/HTTP-curl-twitter-with-doh.png)
</div>

(تصویر از انتهای لاگ -- تست شده در ویندوز)

در خط اول می بینید که درخواست DoH به درستی کامل شد. در نتیجه IP ی دریافت شده درست است.

در ادامه، به سایت توییتر با آدرس دریافت شده، درخواست فرستاده شد و چون ارتباط HTTP است، یعنی رمزنگاری نیست و به راحتی توسط شخص ناشناس قابل دستکاری است، خطای 403 در Header و iframe ای که صفحه ی سانسور جمهوری اسلامی را نمایش خواهد داد، دریافت شد.

برای آزمایش مشابه، اگر قصد وصل شدن به یک IP ی خاص داریم، می توانیم به این صورت از curl استفاده کنیم:

<div dir="ltr">

`> .\curl -v --resolve twitter.com:80:104.244.42.193 http://twitter.com/`
<br/>

![HTTP curl twitter with resolve cache ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-twitter-with-resolve-cache-ip.png)
</div>

مقدار 80 به معنای Port ای است که قصد وصل شدن به آن را داریم.

`--resolve` برای قرار دادن یک IP در DNS cache موقت curl است.

حال سوالی که پیش می آید این است که آیا این سانسور، بر مبنای مقدار Host در header است و یا بر مبنای محتوای درخواست، و یا هر دو؟

برای بررسی، می توانیم از آدرسی که می دانیم که سانسور نیست استفاده کنیم. مانند `example.com` :
<div dir="ltr">

`> .\curl -v http://example.com`
<br/>

![HTTP curl exampledotcom](/images/docs/measure-internet-censorship/Application/HTTP-curl-exampledotcom.png)
</div>

در اینجا می بینیم که HTTP 200 OK دریافت شده و مقدار title برابر با مقداری است که با اینترنت بدون سانسور به دست می آوریم.
 این بدان معناست که در این شبکه ارتباط با این دامنه و IP سانسور نمی شود.

حال که از مسدود نبودن آن مطمئن هستیم، می توانیم از این IP برای آزمایش استفاده کنیم. 
در curl، علاوه بر `--connect-to` و `--resolve` می توانیم از `-H` یا `--header` نیز استفاده کنیم. 
به این معنا که ما قصد اضافه کردن و یا باز نویسی یک header در HTTP داریم. به این صورت:
<div dir="ltr">

`> .\curl -v -H 'Host: twitter.com' http://93.184.216.34`
<br/>

![HTTP curl twitter with example.com ip.png](/images/docs/measure-internet-censorship/Application/HTTP-curl-twitter-with-exampledotcom-ip.png)
</div>

در اینجا آدرس توییتر در header به عنوان Host انتخاب شد اما خطای 503 دریافت کردیم. 

درخواست مشابه را از طریق سرور خارجی انجام می دهیم تا واکنش سرور را نسبت به این درخواست ببینیم و مطمئن شویم که این خطا، خطایی مربوط به سرور نیست:
<div dir="ltr">

![HTTP germany curl twitter with example.com ip](/images/docs/measure-internet-censorship/Application/HTTP-germany-curl-twitter-with-exampledotcom-ip.png)
</div>

همانطور که انتظار می رفت، این خطا خارج از سیستم سانسور جمهوری اسلامی مشاهده نشده است.

این خطایی است که از سمت سیستم سانسور جمهوری اسلامی، به دلیل ناتوانی در تحلیل به موقع درخواست باز می گرداند. 
[در اینجا می توانید ببینید که در یک تست کامل توسط OONI-probe تعداد 329 آدرس این خطا را دریافت کردند](https://github.com/net4people/bbs/issues/42). حدود 15٪ از کل لیست (شامل اکثریت پروتکل HTTPS).

حال برای بررسی دقیق زمان واکنش `--trace-time` را اضافه می کنیم:
<div dir="ltr">

`> .\curl -v --trace-time -H 'Host: twitter.com' http://93.184.216.34`
<br/>

![HTTP curl trace time twitter with example.com ip repeat](/images/docs/measure-internet-censorship/Application/HTTP-curl-trace-time-twitter-with-exampledotcom-ip-repeat.png)
</div>

زمان پاسخگویی با خطای 503، دقیقا 30 ثانیه تنظیم شده است. 
چند بار این تست را تکرار می کنیم و بعد از گذشت چند دقیقه از درخواست اولیه، سیستم سانسور جواب معمول خود را برای سانسور، ارسال می کند:
<div dir="ltr">

![HTTP curl trace time twitter with example.com ip repeat](/images/docs/measure-internet-censorship/Application/HTTP-curl-trace-time-twitter-with-exampledotcom-ip-repeat.png)
</div>


حال آزمایش می کنیم که سیستم سانسور جمهوری اسلامی نسبت به ارتباط با IP ی توییتر، اما دامنه ی `example.com` در header چه واکنشی نشان می دهد:

<div dir="ltr">

`> .\curl -v -H 'Host: example.com' http://104.244.42.193/`
<br/>

![HTTP curl example.com with twitter ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-exampledotcom-with-twitter-ip.png)
</div>

جواب تعجب برانگیزی دریافت شده است. 
در حالت عادی شما نمی توانید از IP یک سایت دیگر استفاده کنید و جواب یکسان بگیرید. 
مگر اینکه هر دو بر روی یک هاست اشتراکی باشند و یا هر دو از CDN خاصی استفاده کنند که از domain-fronting پشتیبانی می کند. 

برای اطمینان دوباره از سرور خارجی همین درخواست را تکرار می کنیم:
<div dir="ltr">

![HTTP germany curl example.com with twitter ip](/images/docs/measure-internet-censorship/Application/HTTP-germany-curl-exampledotcom-with-twitter-ip.png)
</div>

جواب متفاوت است. 
در نتیجه، پاسخ قبلی احتمالا به دلیل وجود سیستم Cache در اینترنت آن شبکه است.
در بخش «لایه ی شبکه و انتقال»، نشان داده شد که در دستور traceroute با Port `80` و به صورت TCP، بعد از فقط ۳ Hop، با تاخیر بسیار پایین، از سرور مقصد جواب دریافت می کردیم. 
این سیستم احتمالا فقط بر مبنای مقدار Host کار می کند و IP را برای درخواست نهایی مد نظر قرار نمی دهد. 

یک تست دیگر انجام می دهیم. با IP ی تلگرام و دامنه ی iran.ir :
<div dir="ltr">

`> .\curl -v -H 'Host: iran.ir' http://149.154.167.99/`
<br/>

![HTTP curl iran.ir with telegram ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-irandotir-with-telegram-ip.png)
</div>

در نتیجه، IP اگر مسدود باشد، از آن جلوگیری می شود، تفاوتی وجود ندارد. 
به جای تلگرام از IP ی Wikipedia استفاده می کنیم:
<div dir="ltr">

`> .\curl -v -H 'Host: iran.ir' http://91.198.174.192/`
<br/>

![HTTP curl iran.ir with wikipedia ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-irandotir-with-wikipedia-ip.png)
</div>

دوباره خطای 503 دریافت کردیم که به نظر سیستم Cache جمهوری اسلامی برای آدرس iran.ir آماده نبود.

این بار به جای آدرس iran.ir از google.com استفاده می کنیم که می دانیم که به تازگی حداقل یک بار توسط مردم درخواست داده شده است:
<div dir="ltr">

`> .\curl -v -H 'Host: google.com' http://91.198.174.192/`
<br/>

![HTTP curl google.com with wikipedia ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-googledotcom-with-wikipedia-ip.png)
</div>

همانطور که انتظار می رفت، جواب داده شد. همین درخواست را دوباره از سرور خارج انجام می دهیم:
<div dir="ltr">

![HTTP germany curl google.com with wikipedia ip](/images/docs/measure-internet-censorship/Application/HTTP-germany-curl-googledotcom-with-wikipedia-ip.png)
</div>

می بینیم که جواب متفاوت است. 
دوباره همین درخواست را از ایران انجام می دهیم و www. را اضافه می کنیم:
<div dir="ltr">

`> .\curl -v -H 'Host: www.google.com' http://91.198.174.192/ `
<br/>

![HTTP curl www.google.com with wikipedia ip](/images/docs/measure-internet-censorship/Application/HTTP-curl-wwwgoogledotcom-with-wikipedia-ip.png)
</div>

محتوای صفحه ی گوگل از طریق IP ی ویکی‌پدیا به صورت کامل دریافت شده است. با توجه به ساعت درخواست، به نظر درخواست در همان لحظه انجام شده و در نتیجه این یک middle box در سیستم سانسور جمهوری اسلامی است.

در این رابطه، در گذشته [گزارشی توسط Citizenlab](https://citizenlab.ca/2018/04/planet-netsweeper/) منتشر شده که رویکرد سیستم سانسور ای که توسط شرکت کانادایی Netsweeper ساخته شده را به صورت زیر به تصویر کشیده است:

<div dir="ltr">

![Citizenlab planet netsweeper](/images/docs/measure-internet-censorship/Application/Citizenlab-planet-netsweeper.png)
</div>


## ارتباط HTTPS

همانند HTTP ما نباید سانسور از طریق DNS hijacking را در این سطح از آزمایش ها دخیل کنیم. 
برای این کار، از همان روش های توضیح داده شده در ارتباط HTTP میتوانیم استفاده کنیم. 
بجز اینکه تمام header ها و آدرس ها رمزنگاری می شوند و در نتیجه توسط سیستم سانسور قابل مشاهده نیستند تا نسبت به آنها سانسور شدن یا نشدن تصمیم گیری شود.
اما موارد جدیدی وجود دارند که سیستم سانسور نسبت به آنها تصمیم گیری کند و در صورت تمایل، از ادامه ی ارتباط جلوگیری کند.

اولین مورد می تواند [SNI](https://en.wikipedia.org/wiki/Server_Name_Indication) باشد. 
بخشی در ارتباط TLS که به سرور این اطلاع را می دهد که کاربر قصد وصل شدن به چه host ای دارد.

SNI در Client Hello قرار دارد. 
اما بعد از این packet، سرور Server Hello را ارسال می کند که اگر زیر نسخه ی TLSv1.3 باشد، محتوای сerfiticate به صورت plain text ارسال می شود. 
در گذشته از این طریق شناسایی محتوای رمزنگاری نشده در این packet، [ارتباط سرورهای اینستاگرام در ایران مسدود شده بود](https://ooni.org/post/2018-iran-protests-pt2/). (تنها کشوری که تاکنون مشاهده شده که چنین کاری انجام داده است)

اما با پیشرفت تکنولوژی این ضعف دیگر در نسخه های جدید وجود ندارد. 
البته برای حل مشکل SNI نیز استانداردی در حال نوشته شدن است که extension های با اطلاعات حساس در Client Hello به صورت رمزنگاری شده در [extension جدیدی به نام ECH](https://blog.cloudflare.com/encrypted-client-hello/) قرار بگیرند. 

در حال حاضر ارتباط HTTPS به چند صورت مسدود می شوند:

- از طریق SNI
- از طریق fingerprint در TLS
- مسدود سازی TLS به صورت کلی
- مسدود سازی ارتباط به صورت random

در مورد مسدود سازی ها از طریق fingerprint و مسدود کردن های random، در بخش هایی مجزا توضیح داده خواهد شد.
در ادامه به دو مورد ساده تر می پردازیم.

### بررسی ساده ی SNI و TLS 

دسترسی به توییتر را با تنظیم دستی timeout به مدت 30 ثانیه مورد آزمایش قرار می دهیم:

<div dir="ltr">

`$ curl -v -m 30 --doh-url https://mozilla.cloudflare-dns.com/dns-query https://twitter.com/`
<br/>

![HTTPS curl twitter with doh and timeout30](/images/docs/measure-internet-censorship/Application/HTTPS-curl-twitter-with-doh-timeout30.png)
</div>


در اینجا می بینیم که بعد از Client hello و بعد از گذشت 30 ثانیه، با پیام timeout ارتباط بسته شده است. این به این دلیل است که در این اتفاق احتمالا به دلیل SNI است اما برای اطمینان باید بررسی بیشتر انجام دهیم.

برای اینکه ببینیم که آیا سیستم سانسور به نام توییتر در SNI حساس است یا مسئله ی دیگر، باید از IP سایتی که می دانیم فیلتر نیست استفاده کنیم و اما از آدرس توییتر در SNI استفاده کنیم. مانند دستور زیر که از IP ی سایت example.org استفاده کردیم:

<div dir="ltr">

`$ curl -v -m30 --resolve 'twitter.com:443:93.184.216.34' https://twitter.com/`
<br/>

![HTTPS curl twitter with resolve cache example.com ip timeout30](/images/docs/measure-internet-censorship/Application/HTTPS-curl-twitter-with-exampledotcom-ip-timeout30.png)
</div>

به دلیل اینکه پیامدی مشابه داشته، به این نتیجه می رسیم که این مسدود سازی از طریق شناسایی آدرس twitter.com در بخش SNI در Client hello صورت گرفته است.

اما سیستم سانسور جمهوری اسلامی گاهی رفتاری متفاوت نیز دارد. 

برای نشان دادن نمونه های بیشتر، این بار آدرس dns.google را امتحان می کنیم:
<div dir="ltr">

`$ curl -v -m30 https://dns.google/`
<br/>

![HTTPS curl dns.google](/images/docs/measure-internet-censorship/Application/HTTPS-curl-dnsgoogle.png)
</div>

همانند قبل ارتباط بعد از Client hello وارد سیاه چاله شده است. 

در ادامه تست مسدود بودن SNI را انجام می دهیم و از IP ی سایت example.org استفاده می کنیم:
<div dir="ltr">

`$ curl -v -m30 --resolve dns.google:443:93.184.216.34  https://dns.google/`
<br/>

![HTTPS curl dns.google with example.com ip](/images/docs/measure-internet-censorship/Application/HTTPS-curl-dnsgoogle-with-exampledotcom-ip.png)
</div>

در ایجا می بینیم که TLS handshake به صورت کامل انجام شده است. 
(خطا به خاطر همسان نبودن Certificate ارسالی از سرور و آدرس درخواستی است)

بنابراین مسدود سازی نمی تواند صرفا به خاطر آدرس dns.google باشد. 
سوال این است: 
آیا از هر ارتباط TLS به سمت آدرس 8.8.4.4 جلوگیری می شود؟ 

برای پاسخ به این سوال، این بار از آدرس IP ی dns.google استفاده می کنیم اما از آدرس دامنه ی example.com برای SNI استفاده می کنیم:
<div dir="ltr">

`$ curl -v -m30 --resolve example.com:443:8.8.4.4 https://example.com/`
<br/>

![HTTPS curl example.com with dns.google ip](/images/docs/measure-internet-censorship/Application/HTTPS-curl-exampledotcom-with-dnsgoogle-ip.png)
</div>

در اینجا می بینیم که سیستم سانسور جمهوری اسلامی فقط نسبت به هر ارتباط TLS به این IP حساس است و آدرس دامنه را ملاک قرار نمی دهد.

البته این رفتار در همه ی شبکه ها و یا نسبت به همه ی آدرس ها یکسان نیست. 
به عنوان مثال، تست های بالا را که در مخابرات (AS58224) انجام شده بود را در همراه اول (AS197207) تکرار می کنیم:
<div dir="ltr">

`$ curl -v -m30 --resolve dns.google:443:8.8.4.4  https://dns.google/`
<br/>

![HTTPS mci curl dns.google 8.8.4.4 ip](/images/docs/measure-internet-censorship/Application/HTTPS-mci-curl-dnsgoogle-8844-ip.jpg)

</div>

می بینیم که بعد از به پایان رسیدن TLS Handshake ارتباط وارد سیاه چاله شده است؛ زمانی که ارتباط HTTP  شروع می شود. 
این می تواند به دلیل تاخیر در واکنش سیستم سانسور باشد که اولین بار این رفتار [در چین مشاهده شد](https://github.com/net4people/bbs/issues/43#issuecomment-673490763).

البته در اینجا حساسیت فقط به آدرس `dns.google` نیست. 
به عنوان مثال در تست بعدی، همانند قبل پیش می رویم و از IP ی سایت `example.com` استفاده می کنیم.
همچنین به دلیل اینکه مسدود شدن ارتباط بعد از TLS Handshake اتفاق افتاده بود، در تست بعدی از `-k` نیز استفاده می کنیم تا تفاوت certificate با آدرس درخواستی در SNI، باعث پایان یافتن ارتباط قبل از شروع HTTP نشود.

<div dir="ltr">

`$ curl -v -m30 -k --resolve dns.google:443:93.184.216.34  https://dns.google/`
<br/>

![HTTPS mci curl dns.google with example.com ip part 1](/images/docs/measure-internet-censorship/Application/HTTPS-mci-curl-dnsgoogle-with-exampledotcom-ip-1.jpg)
<br/>

![HTTPS mci curl dns.google with example.com ip part 2](/images/docs/measure-internet-censorship/Application/HTTPS-mci-curl-dnsgoogle-with-exampledotcom-ip-2.jpg)
</div>

ارتباط مسدود نشده است.

و در تست بعدی، از IP ی `dns.google` استفاده می کنیم و SNI را برای `example.com` در نظر می گیریم:
<div dir="ltr">

`$ curl -v -m30 -k --resolve example.com:443:8.8.4.4 https://example.com/`
<br/>

![HTTPS mci curl example.com with dns.google ip](/images/docs/measure-internet-censorship/Application/HTTPS-mci-curl-exampledotcom-with-dnsgoogle-ip.jpg)
</div>

در نتیجه در این شبکه، بعضی از سرویس ها فقط در صورت ای مسدود خواهند شد که هم آدرس در SNI و هم آدرس IP با rule تنظیم شده در سیستم سانسور مطابقت داشته باشند.
در مورد این موضوع در بخش مسدود سازی random سرویس ها باز هم توضیح داده خواهد شد.

## تشخیص تحریم از فیلتر

برای تشخیص اینکه یک ارتباط فیلتر شده است و یا به دلیل تحریم امکان ارتباط با آن وجود ندارد، باید از ارتباط HTTPS استفاده شود.
به دلیل اینکه این امکان فراهم می شود تا این بررسی از دستکاری شدن محتوایش در مسیر ارتباط در امان بماند.

همچنین طی یک سال اخیر، در اکثر موارد تحریم بدون نمایش خطای HTTP، از ارتباط جلوگیری نشده است. 
یعنی اگر تحریم صورت گرفته باشد، ما معمولا می توانیم به صورت کامل TCP handshake و TLS handshake داریم.

به دلیل سادگی تشخیص تحریم و پیچیدگی فیلتر، بهتر است که از OONI همانطور که توضیح داده شد استفاده کنید. 
اما اگر به صورت دستی می خواهید این کار را انجام دهید، می توانید ابتدا یک بار آدرس را با استفاده از شبکه ی خود تست کنید و اگر خطای HTTP ی 403 و یا 404 دریافت کردید و اما وقتی از فیلترشکن استفاده می کنید خطایی مشابه دریافت نکردید، این نشان می دهد که ارتباط با ایران تحریم شده است.

البته گاهی ممکن است که یک آدرس، هم تحریم و هم فیلتر شود.
مانند Firebase :

<blockquote class="twitter-tweet" data-lang="en" data-dnt="true" data-theme="dark"><p lang="fa" dir="rtl">دامنه‌ی <a href="https://t.co/dJqPfPjT8M">https://t.co/dJqPfPjT8M</a> از بیخ و بن فیلتر شد؟! 😕<a href="https://twitter.com/azarijahromi?ref_src=twsrc%5Etfw">@azarijahromi</a></p>&mdash; Shecan | شکن (@shecan_ir) <a href="https://twitter.com/shecan_ir/status/1263424522784591872?ref_src=twsrc%5Etfw">May 21, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

