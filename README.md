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
    - that character set [abcdefghijklmnopqrstuvwxyz0123456789]

هل احنا هنعمل ال list البنجبث عنمها ولا هندور ازاي وفين 

com.alexu.hashcracker/
├── core/              ← المنطق المشترك (Hashing + Candidate generation)
│   ├── HashUtils.java
│   └── CandidateGenerator.java
├── coordinator/        ← كل حاجة خاصة بالـ Coordinator (الأسبوع 2)
│   └── CoordinatorMain.java
└── worker/              ← كل حاجة خاصة بالـ Worker (الأسبوع 2)
    └── WorkerMain.java


distributed-hash-cracker/
│
├── pom.xml
│
├── src/main/java/
│   │
│   └── com/team/hashcracker/
│       │
│       ├── common/
│       │   ├── model/
│       │   │   ├── Chunk.java
│       │   │   ├── WorkerInfo.java
│       │   │   ├── SearchConfig.java
│       │   │   └── SearchResult.java
│       │   │
│       │   └── protocol/
│       │       ├── Message.java
│       │       └── MessageType.java
│       │
│       ├── coordinator/
│       │   ├── CoordinatorServer.java
│       │   ├── WorkerRegistry.java
│       │   ├── ChunkScheduler.java
│       │   ├── LeaseManager.java
│       │   └── ResultHandler.java
│       │
│       ├── worker/
│       │   ├── WorkerClient.java
│       │   ├── WorkerEngine.java
│       │   ├── LocalTaskManager.java
│       │   └── HashService.java
│       │
│       ├── networking/
│       │   ├── TcpServer.java
│       │   ├── TcpClient.java
│       │   ├── MessageSerializer.java
│       │   └── ConnectionHandler.java
│       │
│       └── monitoring/
│           ├── PerformanceMetrics.java
│           └── BenchmarkRunner.java
│
└── src/test/java/
    └── com/team/hashcracker/