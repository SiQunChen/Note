# Chapter 1 : Introduction
* Interrupt : 可以讓 CPU 跟 I/O 同時做事情
* Dual-mode : OS 區隔權限成兩種模式(User mode, kernel mode)
    * CPU 裡有些指令集需要權限才能執行
![image](https://hackmd.io/_uploads/Skkevbz6p.png)
* Timer : 避免程式當機、無窮迴圈資源被限制住
* 如何同時執行多種作業系統?
    * Emulation : 進行不同作業系統之間的轉換
    * Virtualization : 透過虛擬機
## 作業系統中的CPU資源分配
### 1. 多重程式(Multiprogramming)
* 概念:
    * 當某程式暫時不需使用CPU的時候，監控程式會啟動另外的正在等待CPU資源的程式。目的是讓CPU資源能充分被使用。
* 優點:
    * 看似原始但在當時確實大幅提高了CPU使用率。
* 缺點:
    * 程式之間的排程策略太過粗糙，因為CPU是前一程式不使用時才能輪到其他需要CPU資源的程式使用CPU，也就是程式沒有優先序高低的差別；這會讓一些急需使用CPU完成任務的程式(例如使用者互動的任務)，可能需要等待很長的時間才分配到CPU資源。這會造成你在你的laptop按了一下滑鼠、或者在鍵盤上敲了幾個鍵，結果過了10分鐘系統才有反應。
### 2. 分時系統(Time-Sharing System)
* 概念:
    * 經過稍微改進後，程式執行模式變成一種合作模式: 每個程式執行一段時間後，會主動讓出CPU給其他程式。
* 優點:
    * 在一小段時間內讓每個程式都有機會被執行到，對一些互動式任務來說很重要，可以解決前述問題，按下鍵盤滑鼠時，程式需要處理的任務可能不多，但需要系統盡快處理任務，讓使用者能立即看到結果。
* 缺點:
    * 如果遇到程式在進行很耗時的計算，一直霸佔著CPU資源，那作業系統也沒辦法，其他程式都只能等著。整個系統會看起來像當機一樣卡住。例如程式進入while(1)無窮迴圈，會讓整個系統都卡住。
### 3. 多工(Multi-tasking)系統
因為前兩種排程方式的缺點，因而演進成我們現在很熟悉的多工(Multi-tasking)模式。
* 概念: 使用一種叫先佔式(Preemptive)的CPU分配方式
    * 作業系統接管了所有硬體資源，且本身是執行在受硬體保護的級別。
    * 所有程式都以行程(Process)的方式執行在比作業系統權限更低的級別，每個行程都有自己獨立的位址空間、有單獨的記憶體，使行程間的位址空間互相隔離。
    * CPU由作業系統統一分配:
        * 每個行程根據行程優先序都有機會得到CPU資源，但如果執行時間超過一定時間，作業系統會暫停該行程，將CPU資源釋出給其他等待執行的行程。
        * 作業系統可強制剝奪CPU資源並分配給它認為目前最需要的行程。
        * 如果作業系統分配給每個行程的時間都很短，即CPU在多個行程間快速切換，看起來會像是很多行程都在同時執行。
    * 目前幾乎所有現代OS都是採用這種方式，比如UNIX、Linux、Windows NT以後的版本、Mac OS X以後的版本。
## Management
### Process Management
* Program 是靜態的， process 是動態的
* 管理 CPU, memory, I/O 等
* Single-threaded process
    * program counter : 紀錄下個指令的位置
* Multi-threaded process
    * program counter : 指向某個 thread
### Memory Management
* 有效配置有限的記憶體空間
### File-System Management
* 有效地儲存檔案資料
* 管理檔案權限
### Mass-Storage Management
* 有效配置有限的硬體空間
* Disk scheduling : 探討 HDD 如何進行有效的讀寫頭移動
### Caching
* CPU 的 caching 是把從 memory 讀到 register 的資料暫存
* 硬碟的 caching 是把經常使用的資料暫存在 memory 的 caching
### I/O Subsystem
* buffering
* caching
* spooling
# Chapter 2 : Operating-system Services
* 使用者角度
    * User interface : CLI, GUI, touch-screen, Batch
    * Program execution
    * I/O operations
    * File-system manipulation
    * Communications
    * Error detection
* 系統角度
    * Resource allocation
    * Logging
    * Protection and security
## System Calls
* 寫高階程式語言並透過 API 呼叫
    * Win32 API for Windows
    * POSIX API for UNIX, Linux, and Mac OS X...
![image](https://hackmd.io/_uploads/rJ2s8QfTT.png)
### Types of System Calls
* Process control
* File management
* Device management
* Information maintenance
* Communications
* Protection
## Operating System Structure
* Simple structure(最簡單的) - MS-DOS
* More complex(還是很簡單、沒有層次) - UNIX
    * Linux 屬於這種進階，因為 kernel 有模組化較方便理解，但沒有層次
    * kernel 功能全部混在一起
    * ![image](https://hackmd.io/_uploads/S1fJO93p6.png)
* Layered(有層次) - an abstraction
    * 一層一層包住，裡面最底層，外面最上層
    * 架構較嚴謹
    * 每一層只能使用上一層提供的功能，第二層使用第一層硬體提供的基本功能，第三層使用第二層的功能，以此類推
    * ![image](https://hackmd.io/_uploads/SJSqIcnpa.png)
* Microkernel(精簡化) - Mach
    * 只保留最重要的功能在 kernel 
    * 其餘功能移到 user mode
    * 優點 : 
        * 容易擴充，不用寫 kernel mode 的 code，只需要寫 user mode 的 code
        * 較可靠、安全
    * 缺點 : 
        * user, kernel 需一直切換，導致效率降低
    * ![image](https://hackmd.io/_uploads/ByZEZ3n6p.png)
* Hybrid Systems
    * 將上面幾種方式混合，某部分用 micro，某部分用 layered 等等
# Chapter 3 : Processes => 執行中的程式
## Process Concept
* Process in Memory
    * stack : 先進後出 => function 概念
    * heap : 先進先出 => 動態配置
    * data : 分配固定的位置給變數等等
    * text : 放 code
    * ![image](https://hackmd.io/_uploads/HkpjPhh6a.png)
* Process State
    * New : 程式被 load 到 memory，剛開始執行的狀態
    * Ready : 會有好幾個程式等待被執行
    * Running : 執行程式
    * Waiting : 遇到事件時的等待，假設有很多程式執行到一半都需要用 I/O，就需要一一等待
    * Terminated : 結束
    * ![image](https://hackmd.io/_uploads/B1NeYhn66.png)
* Process Control Block (PCB)
    * Linux 系統叫 task_struct
    * 紀錄執行中的各種狀態
    * ![image](https://hackmd.io/_uploads/Sk30gp3Tp.png)
## Process Scheduling
* Process Scheduling
    * 目的 : 使 CPU 效率最大化
    * 決定接下來執行哪個程式
    * scheduling queues
        * queue 裡面存放還沒被執行的程式
        * Ready queue
        * Wait queue
    * ![image](https://hackmd.io/_uploads/B1O0HT2aT.png)
* CPU Switch From Process to Process
    * 儲存原本程式執行到一半的進度
    * 從另一個程式一半的進度繼續執行
    * ![image](https://hackmd.io/_uploads/rJpDLa36a.png)
## Operations on Processes
* Process creation
    * 程式執行時會有 Parent 跟 children 形成 tree
    * 每個動作都會有一個 pid (process identifier)
    * parent 跟 children 可以共享資源
    * 可以針對執行中的程式做調整，parent, children 同時做或只能一次做一個等等
    * ![image](https://hackmd.io/_uploads/Hyq596hTT.png)
* Process termination
    * 正常結束時，OS 收到 exit() 後就會釋放記憶體空間等等
    * 執行錯誤、佔用太多資源等等發生，OS 就會收到 abort() 結束
    * 如果 parent 結束，children 仍還在執行
        * cascading termination : 強制等待 children 結束，parent 才能結束
        * 如果 parent 沒有寫 wait，OS 會將 children 認為是 **zombie**
        * 如果 parent 結束了，children 還沒結束，OS 會將 children 認為是 **orphan**
## Interprocess Communication
* 有很多的 process 需要執行，所以彼此需要溝通
* Producer-Consumer Problem
    * unbounded-buffer
        * 沒有限制的 buffer，Producer 可以一直丟東西進去
    * bounded-buffer
        * 有限制的 buffer，如果 buffer 滿了，Producer 就有 wait
* 分為兩種方法
    * ![image](https://hackmd.io/_uploads/r1XYUinC6.png)
### IPC in Shared-Memory Systems
* 一個共享記憶體給他們讀寫
* 可能會有同步問題 => Producer 正在寫，Consumer 正要讀
* ![image](https://hackmd.io/_uploads/H1qOis3RT.png)
### IPC in Message-Passing Systems
* 透過 send, receive 溝通，類似於網路通訊
* 要考慮的點很多，分很多種
    * Direct or indirect
        * Direct : A 的訊息直接傳到 B 手上
        * indirect : A 把訊息放在郵箱裡，B 再去郵箱把訊息拿出來。
    * Synchronization
        * Blocking : 
            * send : A 送完訊息需確認 B 收到
            * receive : B 一直守在信箱前面等
        * Non-blocking : 
            * send : A 送完訊息直接走
            * receive : B 確認過沒信就走
    * Buffering
        * buffer 要多大
## Communication in Client-Server Systems
* Sockets
* Remote Procedure Calls(RPC)



# Chapter 4 : Threads & Concurrency
## Multicore Programming
![image](https://hackmd.io/_uploads/H1tgXOTA6.png)
### Types of parallelism
![image](https://hackmd.io/_uploads/BJpj7OT06.png)
* Data parallelism : 將一個 data 切割成多個 data 執行，每個 core 執行同樣的任務
* Task parallelism : 將一個 data 切割成多個 task 執行，每個 core 執行不同的任務
### Amdahl's Law : 評估多核的效率
> S 表示必須依序完成的部分(不能同時進行)，N 表示有幾個 cores  
> 若 N 為無窮大，則能夠快 1/S 倍  
> S 為決定效率最重要的部分  
> ![image](https://hackmd.io/_uploads/B1xc4ua0a.png)
:::info
* Q : 假設一個應用程式有 75% parallel and 25% serial，並且有 2 cores
* A : 1/(0.25 + (0.75 / 2)) = 1.6 倍，即 2 cores 比 1 core 快 1.6 倍
:::
## Multithreading Models
* Many-to-One
    * 較早期的作法，因為只有單核，所以 user threads 必須輪流進行
    * ![image](https://hackmd.io/_uploads/SJWCo_60p.png)
* One-to-One
    * 能夠平行處理，但缺點是效率有所限制 => user threads 不能超過核心數
    * ![image](https://hackmd.io/_uploads/BJdznupCa.png)
* Many-to-Many
    * 較複雜，需要進行處理
    * ![image](https://hackmd.io/_uploads/HJkt2_p06.png)
* Two-level Model 
    * 較常見，融合 One-to-One, Many-to-Many
    * ![image](https://hackmd.io/_uploads/B1heTdpAp.png)
## Thread Libraries
提供一個 API 控制哪些程式的 Function 要分配給哪些 threads 執行
* Pthreads : POSIX API，主要用在 UNIX 系統
* Windows 也有提供相關 API，只是沒有固定名稱
* Java Threads 
## Operating System Examples
* Windwos Threads
    * kernel space
        * ETHREAD : 指向 KTHREAD
        * KTHREAD : 指向 TEB
    * user space
        * TEB
* Linux Threads  
    * Linux 叫 tasks 而不是叫 threads
# Chapter 5 : CPU Scheduling
CPU 空閒下來時，選一個 process 執行
## Scheduling Criteria
## Scheduling Algorithms
## Thread Scheduling
## Multi-Processor Scheduling
## Operating Systems Examples
## Algorithm Evaluation