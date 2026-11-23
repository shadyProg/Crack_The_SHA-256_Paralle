# Crack_The_SHA-256_Paralle
[] add jdk 27 link:     
https://download.oracle.com/java/27/latest/jdk-27_windows-x64_bin.exe 

[]that how install java on vscode https://youtu.be/S2Kud-BI9_o?si=EapU3cglzh72NWQE

[] test with run hello world java 


# A. Coordinator (Master Node)
ده الجهاز المسؤول عن إدارة عملية البحث كلها.

تخيل إن عندنا 4 أجهزة، جهاز منهم هيكون الـ Coordinator.

مسؤولياته:
- يستقبل الـ Target Hash.
- يستقبل Character Set.
- يستقبل Password Length.
- يحسب عدد الاحتمالات.
- يقسم الاحتمالات إلى Chunks.
- يوزع الـ Chunks على الأجهزة.
- يتابع الأجهزة ويتأكد إنها شغالة.
- لو جهاز فصل، يعيد توزيع شغله.
- لو جهاز لقى كلمة المرور، يبلغ كل الأجهزة توقف.



# B. Worker Nodes
دي الأجهزة اللي هتنفذ عملية البحث الفعلية.

كل Worker:
1. يتصل بالـ Coordinator.
2. يستقبل Chunk من الـ Keyspace.
3. يقسم الـ Chunk على الـ CPU Cores الموجودة عنده.
4. يبدأ يجرب Passwords.
5. يبلغ الـ Coordinator بحالة التنفيذ.
6. لو لقى Password، يبلغ الـ Coordinator.
7. لو وصله أمر إيقاف، يوقف التنفيذ.


# notes 
    - that character set [abcdefghijklmnopqrstuvwxyz0123456789] N = 36
    - Number of possibilities = N^L-1  L is length of password  and -1 because we start from zero . 
    - طريقة البحث كاتلي نولد حروف او ارقام نعملها هاش نقارن الهاش بالتارجت
    - هل احنا هنعمل ال list البنجبث عنمها ولا هندور ازاي وفين

سؤال مهم جدًا، وده بالظبط النقطة اللي بتفرّق بين حل صح وحل هيكره في المنتصف.

الإجابة المختصرة: لأ، مفيش "List" بنعملها أو بندور عليها

ليه؟ لو حسبنا حجم المساحة الكاملة حتى لأبسط test case:

Character set = 36 حرف (a-z + 0-9)
Length = 5
المساحة = 36^5 = 60,466,176 احتمال (60 مليون تقريبًا)

ولو وصلنا لـ L=6 (زي الـ "cluster stress" test في الورقة):

36^6 = 2,176,782,336 احتمال (2 مليار تقريبًا!)

لو حاولنا نعمل List فعلي فيها كل الاحتمالات دي (array أو ArrayList في الذاكرة)، هيحصل Out of Memory فورًا — ده مستحيل تخزينه فعليًا.

الحل الصح: Generate on-the-fly (ما يتولدش إلا وقت الحاجة)

بدل ما نخزن كل الاحتمالات، إحنا بنتعامل مع المساحة كلها كأنها أرقام فقط من 0 لحد N^L - 1 (زي ما الورقة قالت بالظبط: "treats the search space of size N^L as the integer interval [0, N^L − 1]").

يعني:

مفيش تخزين لأي سترينج قبل الوقت.
كل thread بياخد range أرقام (مثلاً من 1000 لحد 2000).
بيلف على كل رقم واحد واحد، وكل رقم بيتحول لسترينج لحظة الاستخدام بس (via CandidateGenerator)، يتعمله hash، يتقارن، وبعدين يترمي من الذاكرة ويروح للرقم اللي بعده.
مثال بسيط يوضح الفكرة (base conversion)

فكّر في الموضوع زي تحويل رقم من نظام عددي عادي (base 10) لنظام "base 36" (لأن عندنا 36 حرف في الـ character set):

index = 0   →  "aaaaa"
index = 1   →  "aaaab"
index = 2   →  "aaaac"
...
index = 36  →  "aaaba"
...
index = N^L - 1  →  "99999"  (آخر احتمال)

كل رقم بيتحول لسترينج بنفس الطريقة اللي بنحول بيها رقم عادي لنظام هيكساديسيمال مثلاً — بس هنا الـ "digits" مش 0-9 وA-F، لكن الـ character set بتاعنا (abcdefghijklmnopqrstuvwxyz0123456789).

يعني عمليًا إيه اللي هيحصل في الكود؟

الـ CandidateGenerator مش هيكون عنده أي List ولا Array مخزن. هيكون بس function واحدة:

java
public static String indexToString(long index, String charset, int length)

الـ function دي بتاخد رقم وترجع السترينج المقابل له على طول، من غير ما تخزن أي حاجة قبلها. وبعدين الـ loop بتاع كل thread بيكون شكله تقريبًا كده (توضيحي بس، مش الكود النهائي):

java
for (long i = start; i < end; i++) {
    String candidate = CandidateGenerator.indexToString(i, charset, length);
    if (HashUtils.sha256(candidate).equals(targetHash)) {
        // لقيناه!
    }
}

كده الذاكرة المستخدمة ثابتة طول الوقت (سترينج واحد بس في كل لحظة لكل thread)، مهما كانت المساحة الكلية كبيرة قد إيه.




com.alexu.hashcracker/
├── core/              ← المنطق المشترك (Hashing + Candidate generation)
│   ├── HashUtils.java
│   └── CandidateGenerator.java
├── coordinator/        ← كل حاجة خاصة بالـ Coordinator (الأسبوع 2)
│   └── CoordinatorMain.java
└── worker/              ← كل حاجة خاصة بالـ Worker (الأسبوع 2)
    └── WorkerMain.java
