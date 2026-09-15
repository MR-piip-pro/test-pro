مرحبا صابقاكنا نتحدث عن هادة وتوقفنا "أكيد. على Windows 10/11 افتح PowerShell كمسؤول (Run as Administrator) ونفّذ هذه الأوامر بالترتيب: powershell Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
powershell Start-Service sshd
powershell Set-Service -Name sshd -StartupType Automatic
ثم للتأكد من أن الخدمة تعمل: powershell Get-Service sshd
إذا ظهر Running، فتم الأمر بنجاح. ومن Termux على الهاتف، وأنت على نفس Wi-Fi، ستتصل هكذا: bash ssh اسم_المستخدم@IP_الحاسوب
ولمعرفة IP الحاسوب في PowerShell: powershell ipconfig
وابحث عن IPv4 Address، مثل 192.168.1.25. إذا أردت، أستطيع بعد ذلك إعطاؤك طريقة تجعل في Termux أوامر مثل start-stream وstop-stream لتشغيل وإيقاف البث من الهاتف بأمر واحد.
تقريبًا أي أمر تستطيع تشغيله من PowerShell أو CMD على Windows يمكن إرساله عبر SSH، لأن SSH يعطيك جلسة أوامر على الحاسوب. مثلاً من Termux: bash ssh username@192.168.1.25
وبعد الدخول يمكنك تنفيذ: powershell ipconfig
لعرض معلومات الشبكة. powershell tasklist
لعرض البرامج والعمليات الحالية. powershell Get-Process
لعرض العمليات في PowerShell. powershell Get-Service
لعرض خدمات Windows. powershell shutdown /s /t 0
لإيقاف تشغيل الحاسوب. powershell shutdown /r /t 0
لإعادة تشغيله. ويمكنك أيضًا تشغيل البرامج، مثلًا: powershell Start-Process "C:\Program Files\Google\Chrome\Application\chrome.exe"
والأجمل أنك لا تحتاج حتى للدخول إلى جلسة SSH؛ تستطيع تنفيذ الأمر مباشرة من Termux: bash ssh username@192.168.1.25 "ipconfig"
أو: bash ssh username@192.168.1.25 "shutdown /r /t 0"
وبالتالي يمكننا لاحقًا عمل أوامر مختصرة في Termux مثل: bash start-stream stop-stream restart-pc
بحيث تكتب أمرًا واحدًا من الهاتف وينفذ الحاسوب المطلوب.
أكيد. سنجعلها من الصفر إلى أول اتصال ناجح بين Windows وTermux، وبطريقة تجعل SSH يعمل تلقائيًا مع كل تشغيل لـWindows. سأفترض أن: • الحاسوب Windows 10 أو 11. • الهاتف Android وعليه Termux. • الهاتف والحاسوب متصلان بنفس شبكة Wi-Fi. • أنت تريد التحكم بحاسوبك الشخصي. 1. إعداد Windows على الحاسوب، افتح PowerShell كمسؤول: اضغط زر Start واكتب: PowerShell ثم اضغط بزر الفأرة الأيمن عليه واختر Run as administrator. نفّذ الأمر الأول: powershell Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
انتظر حتى ينتهي التثبيت. ثم شغّل SSH: powershell Start-Service sshd
ثم اجعله يبدأ تلقائيًا مع كل تشغيل Windows: powershell Set-Service -Name sshd -StartupType Automatic
وأخيرًا تأكد من حالته: powershell Get-Service sshd
يجب أن ترى شيئًا مثل: text Status   Name   DisplayName ------   ----   ----------- Running  sshd   OpenSSH SSH Server
إذا كانت Running فأنت تمام. ✅
───
2. السماح بالاتصال من الشبكة عادةً يقوم OpenSSH بإنشاء قاعدة Firewall تلقائيًا، لكن لنتأكد. في PowerShell كمسؤول نفّذ: powershell Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP"
إذا ظهرت قاعدة باسم OpenSSH-Server-In-TCP فهي موجودة. إذا لم تظهر، أنشئها: powershell New-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -DisplayName "OpenSSH Server (sshd)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
المنفذ الافتراضي لـSSH هو 22.
───
3. معرفة اسم مستخدم Windows في PowerShell اكتب: powershell $env:USERNAME
مثلاً إذا ظهر: text Ahmed
فاسم المستخدم هو: Ahmed احتفظ به لأننا سنحتاجه في Termux.
───
4. معرفة IP الحاسوب في PowerShell: powershell ipconfig
ابحث عن اتصال Wi-Fi الخاص بك. ستجد شيئًا مثل: text IPv4 Address . . . . . . : 192.168.1.25
إذن عنوان الحاسوب في الشبكة هو: 192.168.1.25 لا تستخدم هذا الرقم كمثال إذا كان عندك رقم مختلف.
───
5. إعداد Termux افتح Termux على الهاتف. أولًا حدّث الحزم: bash pkg update
ثم: bash pkg upgrade
إذا سألك عن التأكيد، اكتب: text y
ثم ثبّت عميل SSH: bash pkg install openssh
للتأكد من أنه تم تثبيته: bash ssh -V
ستظهر لك معلومات عن إصدار OpenSSH.
───
6. أول اتصال الآن لديك: اسم Windows: مثلاً Ahmed IP الحاسوب: مثلاً 192.168.1.25 في Termux اكتب: bash ssh Ahmed@192.168.1.25
استبدل Ahmed باسم مستخدم Windows عندك، واستبدل 192.168.1.25 بعنوان IP الحقيقي للحاسوب. في أول اتصال قد يظهر لك شيء مشابه: text The authenticity of host '192.168.1.25' can't be established. Are you sure you want to continue connecting (yes/no/[fingerprint])?
اكتب: text yes
ثم سيطلب كلمة مرور حساب Windows. اكتب كلمة مرور حساب Windows. أثناء كتابة كلمة المرور لن تظهر الأحرف أو النجوم في Termux. هذا طبيعي؛ اكتبها واضغط Enter. إذا نجح الاتصال، ستظهر لك جلسة أوامر Windows. 🎉 أنت الآن تتحكم في الحاسوب من Termux.
───
7. اختبار الاتصال بعد الدخول إلى SSH جرّب: powershell hostname
سيظهر اسم الحاسوب. ثم: powershell whoami
ثم: powershell ipconfig
إذا ظهرت معلومات Windows، فالاتصال يعمل بشكل صحيح.
───
8. تنفيذ أمر بدون الدخول إلى جلسة SSH وهذه النقطة مهمة جدًا لما تريد فعله لاحقًا. بدل أن تكتب: bash ssh Ahmed@192.168.1.25
ثم تدخل إلى Windows، يمكنك من Termux تنفيذ أمر مباشرة: bash ssh Ahmed@192.168.1.25 "hostname"
أو: bash ssh Ahmed@192.168.1.25 "whoami"
مثلاً إعادة تشغيل الكمبيوتر: bash ssh Ahmed@192.168.1.25 "shutdown /r /t 0"
وإيقاف تشغيله: bash ssh Ahmed@192.168.1.25 "shutdown /s /t 0"
───
9. ماذا يحدث بعد إعادة تشغيل Windows؟ هنا النقطة التي سألت عنها سابقًا. بما أننا نفذنا: powershell Set-Service -Name sshd -StartupType Automatic
فإن Windows سيشغل خدمة SSH تلقائيًا عند الإقلاع. يعني: تشغيل الحاسوب → Windows يبدأ → SSH يبدأ تلقائيًا → الهاتف يستطيع الاتصال. لا تحتاج إلى تشغيل sshd يدويًا كل مرة.
───
10. مشكلة مهمة: IP قد يتغير قد يكون حاسوبك اليوم: text 192.168.1.25
وبعد فترة يصبح: text 192.168.1.31
وهذا يحدث لأن الراوتر قد يعطي الجهاز عنوانًا مختلفًا. لذلك بعد أن نتأكد أن كل شيء يعمل، يمكننا جعل للحاسوب IP محلي ثابت أو حجز IP له من الراوتر. وهذا سيجعل الأمر دائمًا مثلاً: bash ssh Ahmed@192.168.1.25
بدون الحاجة لمعرفة الـIP كل مرة.
───
11. اختبار كامل أنصحك به الآن نفّذ بالترتيب: على Windows — PowerShell كمسؤول: powershell Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
ثم: powershell Start-Service sshd
ثم: powershell Set-Service -Name sshd -StartupType Automatic
ثم: powershell Get-Service sshd
ثم: powershell $env:USERNAME
ثم: powershell ipconfig
احتفظ باسم المستخدم وIPv4. على Termux: bash pkg update
bash pkg install openssh
ثم: bash ssh اسم_المستخدم@IP_الحاسوب
مثال فقط: bash ssh Ahmed@192.168.1.25
إذا دخلت إلى Windows بنجاح، لا تنتقل للخطوة التالية قبل أن تخبرني بما ظهر لك. بعدها نستطيع إعداد Termux بحيث يكون عندك أوامر قصيرة مثل: text start stop restart
لتشغيل وإيقاف الأشياء التي تريدها على الحاسوب من الهاتف.
نعم. وبما أن حساب Windows عندك بدون كلمة مرور، فـ SSH Keys هو الحل الأنسب. سنجعل الهاتف يثبت هويته للحاسوب باستخدام مفتاح، بحيث لا تضطر لكتابة كلمة مرور Windows عند كل اتصال. مهم: لا ترسل أبدًا الملف id_ed25519 أو محتواه لأي شخص. هذا هو المفتاح السري للهاتف. 1. أولًا: تأكد أن SSH يعمل في Windows افتح PowerShell كمسؤول على Windows ونفّذ: powershell Get-Service sshd
إذا ظهر: text Running
فأنت جاهز. وإذا كان Stopped، نفّذ: powershell Start-Service sshd Set-Service -Name sshd -StartupType Automatic
───
2. إنشاء المفتاح في Termux افتح Termux واكتب: bash pkg update
ثم تأكد من وجود SSH: bash pkg install openssh
الآن أنشئ مفتاحًا: bash ssh-keygen -t ed25519
سيظهر شيء مثل: text Enter file in which to save the key (/data/data/com.termux/files/home/.ssh/id_ed25519):
اضغط Enter فقط. بعدها سيطلب: text Enter passphrase (empty for no passphrase):
إذا كنت تريد الاتصال بدون كتابة أي شيء إضافي كل مرة، اضغط Enter واتركها فارغة. ثم سيطلب التأكيد: text Enter same passphrase again:
اضغط Enter مرة أخرى. سيتم إنشاء ملفين: text ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub
الأول 🔴 سري جدًا. الثاني 🟢 يمكن وضعه على الحاسوب.
───
3. نسخ المفتاح العام إلى Windows من Termux نفّذ: bash cat ~/.ssh/id_ed25519.pub
ستظهر لك سطر طويل يبدأ غالبًا بـ: text ssh-ed25519 AAAA...
انسخ السطر كاملًا من ssh-ed25519 حتى نهايته.
───
4. إنشاء ملف المفاتيح في Windows على Windows افتح PowerShell بحساب Windows العادي الذي تريد الدخول إليه عبر SSH. نفّذ: powershell New-Item -ItemType Directory -Force "$env:USERPROFILE.ssh"
ثم: powershell notepad "$env:USERPROFILE.ssh\authorized_keys"
إذا سألك Notepad عن إنشاء الملف، اختر Yes. ألصق داخله السطر الذي نسخته من Termux، ثم احفظ الملف وأغلق Notepad. يجب أن يكون الملف: text C:\Users\اسمك.ssh\authorized_keys
───
5. تجربة الاتصال ارجع إلى Termux. استخدم: bash ssh اسم_المستخدم@IP_الحاسوب
مثلاً: bash ssh Ahmed@192.168.1.25
إذا كان كل شيء مضبوطًا، سيدخل SSH بدون طلب كلمة مرور Windows. 🎉
───
6. اختبار أن المفتاح هو الذي يُستخدم من Termux يمكنك استخدام: bash ssh -v اسم_المستخدم@IP_الحاسوب
سيعرض تفاصيل الاتصال. ابحث عن شيء يشير إلى: text Offering public key
ثم: text Server accepts key
إذا رأيت ذلك، فالمفتاح يعمل.
───
7. جعل الأمر أسهل بدل أن تكتب كل مرة: bash ssh Ahmed@192.168.1.25
يمكننا إنشاء اختصار. في Termux: bash nano ~/.ssh/config
ضع بداخله: text Host mypc     HostName 192.168.1.25     User Ahmed     IdentityFile ~/.ssh/id_ed25519
احفظ الملف. ثم: bash chmod 600 ~/.ssh/config
الآن الاتصال يصبح ببساطة: bash ssh mypc
وهكذا من الحمام أو المطبخ 😄 تستطيع فتح Termux وكتابة: bash ssh mypc
والدخول إلى حاسوبك.
───
⚠️ إذا لم يعمل بسبب حساب Windows بدون كلمة مرور Windows قد يرفض بعض طرق تسجيل الدخول للحسابات المحلية التي لا تملك كلمة مرور، ولذلك إذا حصلت على خطأ مثل: text Permission denied
لا تغيّر إعدادات الأمان عشوائيًا. أرسل لي نص الخطأ كما ظهر في Termux، وسأحدد لك بالضبط ما يحتاج إلى تعديل. وبعد نجاح SSH Keys، يمكننا الانتقال للجزء الذي تريده فعلًا: .
نعم، وهذه نقطة مهمة جدًا. عند تنفيذ: bash ssh-keygen -t ed25519
سيُنشئ ملفين داخل مجلد .ssh:
الملف نوعه ماذا تفعل به؟
id_ed25519 🔴 المفتاح الخاص (Private Key) لا تشاركه أبدًا
id_ed25519.pub 🟢 المفتاح العام (Public Key) هذا الذي تنقله إلى Windows
كيف تعرف الفرق بسهولة؟ الجزء .pub في نهاية الاسم يعني Public = عام. إذن: text id_ed25519
🔴 سري — يبقى في Termux. بينما: text id_ed25519.pub
🟢 عام — هذا الذي ننسخه إلى authorized_keys في Windows. في حالتك بعد إنشاء المفتاح، لا تحتاج إلى اختيار واحد منهما يدويًا. عندما ننفذ: bash cat ~/.ssh/id_ed25519.pub
نحن نطلب من Termux عرض المفتاح العام فقط. سترى شيئًا شبيهًا بـ: text ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI............... هاتفك
هذا السطر كاملًا هو الذي تنسخه إلى Windows. أما هذا: text ~/.ssh/id_ed25519
🔴 لا تفتحه ولا تنسخه ولا ترسله لي أو لأي شخص. إذا أردت، أستطيع أن ، وأقول لك بالضبط ماذا تكتب في كل سؤال يظهر أمامك." لاكن انا اواجه مشكل في الترتيب وتأكد مادة افعل ولقد واجهة مشكل في صلاحيات المستخدم ايضا لهادة اريد مساعدة في الترتيب انا اريد فقط ماهو ضروري