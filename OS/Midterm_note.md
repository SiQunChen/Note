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
### 各方法資訊整理
![image](https://hackmd.io/_uploads/SkkaNtp10.png)
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
* CPU utilization : 讓 CPU 越忙越好
* Throughput : 單位時間內完成任務的數量越多越好
* Turnaround time : 執行時間越短越好
* Waiting time : 等待時間越短越好
* Response time : 回應時間越短越好
## Scheduling Algorithms
* FCFS (First Come First Served)
    * Convoy effect : 先執行時間較久的 process 會導致 waiting time 較多
    > 範例 1
    > ![image](https://hackmd.io/_uploads/HJG8Bvr1A.png)
    > 範例 2
    > ![image](https://hackmd.io/_uploads/BJ-6rvSyR.png)
    > 範例 2 比範例 1 更好，因為順序的原因導致 waiting time 較少
* SJF (Shortest-Job-First)
    * 較短的 process 優先
    > 範例 1
    > ![image](https://hackmd.io/_uploads/SyffKsB1C.png)
    > 範例 2 (preemptive)  
    > 如果有新的 process 進來會重新進行評估，讓最短的插隊  
    > waiting time : 每個 process 用**全部跑完的時間 - 抵達的時間 - 執行時間**
    > 舉例 process 1 : 從 10 開始全部跑完 - 最一開始執行到 1 被中斷
    > ![image](https://hackmd.io/_uploads/HyM7qjS10.png)
    * 缺點 : 我們無法知道下個 process 的時間多少 => 只能用預測 (exponential averaging)
        * 從上一個預測的值或上一個實際的值做預測，α = 1/2 為兩者以一樣的權重預測(0~1)
        * ![image](https://hackmd.io/_uploads/SJ283sSJA.png)
        > 範例  
        > ![image](https://hackmd.io/_uploads/SyvbpjrJC.png)
* RR (Round Robin)
    * 公平分配每個 process 的時間，時間到不管如何就切換。
    * 一個時間單位叫做 time quantum
    * 主要在減少 response time 的時間
    * Performance
        * q large => FIFO
        * q small => 太頻繁切換導致 overhead 太大
        * q 最好大過於 80% 的 process  
        * ![image](https://hackmd.io/_uploads/rJfCq2YJA.png)
    > 範例  
    > Quantum = 4  
    > ![image](https://hackmd.io/_uploads/ryubvnKyA.png)
* Priority Scheduling
    * 每個 process 賦予一個 priority
    * Preemptive
    * Nonpreemptive
    * Problem : Starvation (priority 小到一直有權限比他大的插隊)
        * Solution : Aging (動態提升 priority => 等得越久，priority 越高)
    > 範例 1  
    > ![image](https://hackmd.io/_uploads/S1r903t1C.png)
    > 範例 2  
    > priority 值相同的情況使用 Round-Robin
    > ![image](https://hackmd.io/_uploads/BkCTAnY10.png)
* Multilevel Queue (Priority Scheduling 延伸)
    ![image](https://hackmd.io/_uploads/SJGrepYyR.png)  
    * Multilevel Feedback Queue
        * 動態調整 priority
        > 範例
        > ![image](https://hackmd.io/_uploads/rJilZTY10.png)
## Thread Scheduling  
* process-contention scope (PCS) : 同一個 process 內的 thread 競爭
* system-contention scope (SCS) : 不同 process 的 thread 競爭
## Multi-Processor Scheduling
> 一個 CPU 有 8 個核心，每個核心又有兩個 hardware threads  
> ![image](https://hackmd.io/_uploads/H1GDNpFk0.png)  
> 第一層為 hardware threads 要如何分配執行 software threads  
> 第二層為 processing core 要如何分配執行 hardware threads  
> ![image](https://hackmd.io/_uploads/SyDDBaYkA.png)  
* Symmetric multiprocessing (SMP)
    * common ready queue
        * ![image](https://hackmd.io/_uploads/Sy2Y76Fk0.png)
    * per-core queues
        * ![image](https://hackmd.io/_uploads/HyaqQ6FkC.png)
* memory stall
    * ![image](https://hackmd.io/_uploads/Bk5XVTY1A.png)
### Load Balancing : 如何讓每個 CPU 都很忙
* Push migration : 很忙的 core 丟任務給空閒的 core
* Pull migration : 空閒的 core 去很忙的 core 抓任務執行
### Processor Affinity
舉例 : core1 執行到一半的任務丟給 core2 執行，但是 core1 已經處理完前面的 data 存放在 cache 了，這樣反而會讓 core2 重頭執行，如果全部 data 丟過去會消耗更多資源
* Soft affinity : 盡量讓同個任務在同個 core 執行，但是不保證，load 時間太久還是會移走
* Hard affinity : 強制讓同個任務在同個 core 執行，不會被移走
### NUMA and CPU Scheduling
用到其他核心的 memory
![image](https://hackmd.io/_uploads/BJEvO6ty0.png)
## Algorithm Evaluation
* Deterministic modeling
    * 假設固定的 workload 用不同的演算法測試計算 waiting time, response time...
    * 優點 : 簡單
    * 缺點 : 因為指針對預先固定的 Workload，所以有可能在其他的 workload 上不適用
* Queueing Models
    * 不固定 workload，用機率方式描述 workload
    * 優點 : 較有系統化的測試，預測各種 workload 進行評估
    * 缺點 : 實際上的案例跟機率計算的案例不一定符合
* Simulations
    * 蒐集各種真實數據，模擬實際情況
    * 優點 : 相較於前兩種方法較準確
    * 缺點 : 麻煩，準確度也僅限蒐集到的資料
* Implementation
    * 不模擬，實際操作
    * 優點 : 最為準確，能夠正確評估各種 process 的效率
    * 缺點 : 成本高、風險高(如果程式寫錯，會導致整台電腦 error)

# Chapter 6 : Synchronization Tools
## Background
* 資料不一致 : 如果一個 process update data, 另一個 process read data
## The Critical-Section Problem
* 設計一個 protocol 當一個 process 在使用 data 時，其他的 process 要先發出 entry section 訪問存取權，當 process 使用完 data 時要發出 exit section 通知其他 process
* 為了實作 protocol 需要設計一些要求
    * Mutual Exclusion : 當一個 process 在使用 data 時，其他的 process 不能訪問 data
    * Progress : 沒有 process 使用 data 時，要在有限的時間內決定下一個 process
    * Bounded Waiting : 等待中的 process 有上限，不能等太久
### Interrupt-based Solution
* 當 Entry section : 把 interrupts 關閉，別人不能發出 interrupts
* 當 Exit section : 把 interrupts 打開，別人能夠發出 interrupts
* 可能遇到的問題
    * 成功 Entry section 的 process 執行太久，其他 process 不能用
    * 等待的 process 一直被插隊，導致一直不能使用
    * 如果有兩個 CPU 的話，會有更多的問題
### Software Solution 1
* 從 code 上做檢查
* 兩個假設簡化問題
    * 假設只有兩個 process
    * 假設執行過程不會被 interrupt
* ![image](https://hackmd.io/_uploads/rJsFFMFeA.png)
* 問題
    * 可能不符合 Progress ，假設 j 速度很快，i 就一直輪不到他執行
## Peterson's Solution
* 兩個假設簡化問題
    * 假設只有兩個 process
    * 假設執行過程不會被 interrupt
* 多了 flag 表示 process 是否已經準備好 entry section
* ![image](https://hackmd.io/_uploads/HkM-TBFeA.png)
* 在現代架構中可能無法 work
    * processors or compilers 可能會因為效率問題 reorder
    * ![image](https://hackmd.io/_uploads/HJ6QJLYlC.png)
### Memory Barrier (硬體 support)
* data 改變時會立即告訴 CPU
* 確保 load and store 能夠完全做完，即使 reorder 也無仿
* ![image](https://hackmd.io/_uploads/r1erZ8FlR.png)
## Mutex Locks (spinlock)
* 兩個模式 : acquire, release
* 兩個模式必須 atomic (不會被 interrupt)
* acquire : 進去前取得許可
* release : 出來前釋放資源
* 問題
    * busy waiting : 要不斷在 while loop 不斷 check
    * 如果在雙核心以上，一個 core 在 busy waiting ，其他的 core 還可以動
    * 但在單核心如果沒有適當機制切換的話就會卡住
## ✭ Semaphores
* 兩個模式 : wait, signal
* 兩個模式必須 atomic (不會被 interrupt)
* wait : check S 的值是否大於 0，如果小於 0 就 wait，會把 S--
* signal : 把 S++
* Binary semaphore : 值在0, 1之間切換，如同 Mutex Locks
* Counting semaphore : 可以計算
### Semaphores Implementation with no Busy waiting
* use waiting queue
* block : if a process waiting, let it go the block
* wakeup : if a process signal, from block choose one wakeup
* ![image](https://hackmd.io/_uploads/SkK_ewKeC.png)
### Problems with Semaphores
* 先 signal 後 wait
* 只有 wait 沒有 signal
## Monitors
* 類似於 class 的方式，能夠更輕鬆實作
* 自動幫 process 增加保護機制(signal, wait...)
# Chapter 7 : Synchronization Examples
## Bounded-Buffer Problem
* 共用的有限 buffer 問題
* mutex : 控制同一時間只有一個 process 能夠 access，初始值為 1
* full : 目前有多少個空間被占用，每次 +1，初始值為 0
* empty : 有 n 個空間，每次 -1，初始值為 n
## Readers-Writers Problem
* 資料不同步問題
* rw_mutex : 控制讀或寫，初始值為 1
* mutex : 控制 buffer 存取，初始值為 1
* read_count : 用整數紀錄目前 read 的 process 數量，初始值為 0
### Readers-Writers Problem Variations
* Problem
    * First reader-writer
        * 如果太多 read，導致沒有辦法 write
* Solution
    * Second reader-writer
        * 如果 write 進 wait，後面 read 要等 write 結束
        * 但可能會變成太多 write 導致不能 read
## Dining-Philosophers Problem
* 假想問題
    * 5 個哲學家、 5 個碗、 5 支筷子，他們必須要 share 筷子
    * ![image](https://hackmd.io/_uploads/BJH6xnAgC.png)
* Solution
    * 如果某個哲學家要用餐的話，要先問過左右邊的人
        * 假如左上跟下面一組，右上跟左下一組，他們輪流吃飯，會導致右下角一直吃不到
        * 改成如果要用餐時，左右邊有人吃飯就設為 hungry 並等待。
        * 吃完的人確認左右邊是否有人 hungry，有的話 signal 他
        * 還是有可能會有人沒吃到的問題，並沒有完全解決
## Kernel Synchronization - Windows
* spinlocks
* dispatcher objects (類似 mutex, semaphore)
    * Events (類似 condition variable)
    * signaled-state
    * non-signaled-state
## Linux Synchronization
* Atomic integers
* Mutex locks
* Spinlocks, Semaphores
* Reader-writer versions of both
## Alternative Approaches
* Transactional Memory
    * 硬體 support
* OpenMP
    * 能夠告訴編譯器需要做一些特殊處理
* Functional Programming Languages
    * 必須透過 function 才能執行
    * 假設一個變數，一定需要 function 才能夠變更他的 value

# Chapter 8 : Deadlocks
* Deadlock with Semaphores
    * ![image](https://hackmd.io/_uploads/SyL8taAeC.png)
* Deadlock Characterization
    * Mutual exclusion : 同一個時間只能有一個 process 使用
    * Hold and wait : 一個 process hold 一個資源，並且再 wait 其他 process
    * No preemption : 除非主動釋放，否則其他 process 不能動用資源
    * Circular wait : 多個 process 都在 hold and wait
## Resource-Allocation Graph
* V(vertices) : 
    * P : 多個 process
    * R : 多個 resource
* E(edges) : 
    * request edge : Pi -> Rj
    * assignment edge : Rj -> Pi3
* Example 1 : 
    * ![image](https://hackmd.io/_uploads/HkLJ2aReA.png)
    * if T3 -> R2 will appear deadlock
    * cuz R2 only 2 resource, already allocate to T1 and T2. And T1 wait T2, T2 wait T3.
    * so T3 has to wait T1 or T2 release resource
    * ![image](https://hackmd.io/_uploads/HywEhTAxC.png)
* Example 2 : 
    * Graph with a Cycle But no Deadlock
    * cuz T1 only wait T2 release resource, don't wait T3. And T3 both
    * ![image](https://hackmd.io/_uploads/r1nuTT0xR.png)
* Basic Facts
    * if graph no cycles => no deadlock
    * if graph have a cycle
        * if only one instance => deadlock
        * if several instances => **possibility** of deadlock
## Methods for Handling Deadlocks
* 如何確保系統不會 deadlock ?
    Deadlock prevention : 盡可能讓上面的四個形成 deadlock 必要條件不成立
    Deadlock avoidance : 監控 process 運作時所需要的 resource 並適時調整
* 等 deadlock 再解決
* 不管 deadlock => 目前大部分作業系統採用這個，遇到就自行重新開機
### Deadlock Prevention
* Mutual exclusion (較難解決)
    * 如果請求全部都是 read ，因為不會更動到 data，所以可以讓 process 同時取用
* Hold and wait (較難解決)
    * 在程式最一開始就配置好所有 resource
        * 可能在過程中還是要 share data，無法在一開始就全部分配好
* No preemption (較難解決)
    * 如果無法立即拿到資源的話，就釋放手中的資源
        * 資源使用效率低
* Circular wait
    * 照順序拿資源
### Deadlock Avoidance
* 需要事先就宣告系統執行時所需要的資源
* 動態監控系統中的資源，Ex : 還剩多少、已使用多少...
* 確保系統不會進入不安全狀態
* 如果系統目前在不安全狀態，則讓目前發送要求的 process 先 wait
* 其他 process 如果能讓系統在安全狀態就先執行
#### What is Safe State?
* 如果現在系統剩下的資源能夠讓接下來所有的 process 都執行完
#### How to inspect Safe State?
* Single instance
    * Use a resource-allocation graph
        * Claim edge (通常用虛線表示) : 當 process 需要資源時，會需要先發送宣告，確保不會不安全，之後才會正式發送 request。
        > Example1
        > T1, T2 需要 R2 正在發送宣告，T2 已發出 request 給 R1，R1 再給 T1 資源
        > ![image](https://hackmd.io/_uploads/S19Q6mdWR.png)
        > Example2
        > T2 發送 request 後，R2 給 T2 資源
        > 但如果 T1 發送 request 給 R2 的話就會進入不安全狀態
        > 所以 T1 發送宣告的時候就會被擋下來了
        > ![image](https://hackmd.io/_uploads/ByBja7dWC.png)

* Multiple instances
    * Use the Banker's Algorithm
        * Available : 表示系統目前各種資源有多少是可用的
        * Max : 每個 process 最多會用到多少的資源
        * Allocation : 目前配置多少的資源給每個 process 了
        * Need : 目前的 process 還需要多少資源
    * Safety Algorithm
        ![image](https://hackmd.io/_uploads/HJp1eNO-C.png)
    * Resource-Request Algorithm
        * 判斷是否可以配置資源給 process
        * 假設如果真的配置給 process 的話會不會不安全
        * 如果不會的話再真的配置給他
        * ![image](https://hackmd.io/_uploads/HyqMKZtWA.png)
    * Example
        * Banker's Algorithm
            > 執行順序如果是 P1, P3, P4, P2 ， 則會保持 Safe  
            > P0 的話會不安全
            > ![image](https://hackmd.io/_uploads/Sy6pKWY-0.png)
            > (Need 可以從 Max - Allocation 獲得)  
            > ![image](https://hackmd.io/_uploads/S1Q49WtWR.png)
        * Resource-Request Algorithm
            > ![image](https://hackmd.io/_uploads/BJv3aZY-A.png)
### Deadlock Detection
* Single instance
    * Wait-for Graph
        > Resource-Allocation Graph, Wait-for Graph 比較圖
        > ![image](https://hackmd.io/_uploads/Bkgq7yfFb0.png)
    * 演算法檢查是否 cycle，最基本要 O(n^2) 的時間
* Multiple instances
    > ![image](https://hackmd.io/_uploads/BJowgztbR.png)
    > ![image](https://hackmd.io/_uploads/r12dgzKZ0.png)
    * Example
        > ![image](https://hackmd.io/_uploads/SJ3vWGtbR.png)
        > ![image](https://hackmd.io/_uploads/HkmKWzKZ0.png)
* Usage
    * 偵測頻率等等參數需要是系統情況而定
* 發生 Deadlock 後該如何解開?
    * 停止全部 process 執行並重新啟動
    * 將發生 deadlock 的 cycle 一個一個終止
        * 先終止哪個 process ?
            * Priority
            * 執行過久的
            * 用太多資源的
            * 還需要很多資源的
            * 哪幾個 process 最快可以解開 deadlock
    * Resource Preemption
        * 允許優先權較高的 process 先使用 => Selecting a victim
        * 把原本暫停的 process 回復 => Rollback
        * 如果 process 一直被暫停 => Starvation
# Chapter 9 : Main Memory
* Logical address : process 看到的 memory 位置 => 相對位置
* Physical address : Memory 中實際的位置
* Ex : 
    * process 上面寫 memory 從 0 開始，但實際上的 Memory 不是從 0 開始，而是那個 process 的 0 開始
* Memory-Management Unit (MMU)
    * 將 Logical address 轉換成 Physical address 的東西
## Contiguous Memory Allocation
* 給每一個 process 一個範圍，讓他只能使用那個範圍
* 位置計算就直接相加就好
* 問題 : 
    * 限制了 process 使用記憶體的空間
    * 如果不用這麼大的空間會導致浪費
* 解決 : 
    * Variable Partition
        * 要多少空間就給多少空間
    * 延伸問題
        > 記憶體可能會變得零碎，導致不連續 => Hole
        > ![image](https://hackmd.io/_uploads/Hyhr14t-A.png)
        * Fragmentation
            * External Fragmentation : 所有空間加起來很大，但是不連續
            * Internal Fragmentation : 配置很多空間，但實際上用的不多
    * 解決
        * Dynamic Storage-Allocation Problem
            * First-fit
                * 選一個 Hole
            * Best-fit
                * 比 process 大的所有 Hole 選最小的
            * Worst-fit
                * 選最大的 hole，這樣切完的 hole 還是會很大
        * compaction
            * 移動記憶體位置，使其合併
            * 但如果執行到一半移動位置，可能會造成 error
            * 而且會花很多時間
## Discontiguous Memory Allocation
### Segmentation
* 將 process 分開放
* Data 放一塊，stack 放一塊，code 放一塊3
* 還是會有 External Fragmentation 的問題
### Paging
* 將 memory 切成好幾塊
    * physical memory 叫做 frames
    * logical memory 叫做 pages
* 假設 pages 是連續的，但在 physical memory 可以隨便放不用連續，只需要紀錄好 frames, pages 的對應關係
* page table : 存放 frames, pages 的對應關係
* 還是有 Internal Fragmentation 的問題
## Swapping