# Chapter 9 : Main Memory
## Paging
* 將 memory 切成好幾塊
    * physical memory 叫做 frames
    * logical memory 叫做 pages
* 假設 pages 是連續的，但在 physical memory 可以隨便放不用連續，只需要紀錄好 frames, pages 的對應關係
* page table : 存放 frames, pages 的對應關係
* free frames list : 紀錄 free frames (還沒有放入空間的 physical memory )
* page table 在 main memory 的表示
    * page-table base register (PTBR) : 紀錄 page-table base
    * page-table length register (PTLR) : 紀錄 page-table size
* Address Translation Scheme
    * page number : 可以透過 page number 查到對應的 frame number
    * page offset : 固定的大小，因為 physical memory 和 logical memory 是一樣大的
* ![image](https://hackmd.io/_uploads/BJvS0keM0.png)
* 問題
    * 還是有 Internal Fragmentation 的問題
        * 將 physical memory 切的更細，但 page table 所佔的空間也會變大
    * 要花兩倍讀寫記憶體的時間 : page table + physical memory
        * 將 page table 放在 cache (TLB)
            > TLB : 平行處理  
            > 因為 TLB 很小，如果 TLB miss 就還是要用 page table  
            > Hit ratio : 80% 的 page number 都查得到  
            > 翻譯後置器（Translation Lookaside Buffer）的縮寫，是一種用於提高虛擬記憶體管理效率的硬體快取。TLB 主要用於處理虛擬地址到物理地址的轉換，以提高存取記憶體的速度。  
            > 通常與虛擬記憶體系統結合使用。當 CPU 存取虛擬記憶體時，它發出一個虛擬地址。這個虛擬地址必須經過轉換才能找到對應的物理地址，而這個轉換過程需要額外的時間。為了加快這個轉換過程，TLB 將最近的一些虛擬地址和對應的物理地址的映射關係保存在快取中。  
            > ![image](https://hackmd.io/_uploads/BJq9fggGC.png)
    * page table 有可能會超出 physical memory 範圍
        > Valid-invalid bit 紀錄 page 空間是否合法
        > ![image](https://hackmd.io/_uploads/BJcx4eeG0.png)
    * 有些 code 不需要每個都執行
        > Shared Pages : page number 對應到同樣的 frame number
        > ![image](https://hackmd.io/_uploads/r1T6NegMR.png)
    * 實際情況中，page table 非常占空間
        * Hierarchical Paging
            > 多層的 page table
            > ![image](https://hackmd.io/_uploads/ByTlPgeGR.png)
        * Hashed Page Tables
            > 主要耗費時間在 chain 的步驟
            > chain 可能會花很久
            > ![image](https://hackmd.io/_uploads/ByGIHrTM0.png)
        * Inverted Page Tables
            > 因為 frame 的數量是固定的
            > 透過查詢 frame 的方式反查 pid 在哪個 frame
            > ![image](https://hackmd.io/_uploads/HysevHpzA.png)
## Swapping
如果 physical memory 不夠的情況
* Backing store
    * 硬碟裡面有一塊特別用於 swapping 的 space
    * 如果 physical memory 不夠的話會暫時存放在硬碟裡，等到空間夠了再移回去
    * 問題 : 
        * 如果移到硬碟中並且沒有經過處理，這樣無法存取 process
            * 通常會考慮什麼樣的 process 可以進來，什麼樣的要留在 physical memory 
        * 從 memory 移到硬碟或硬碟移到 memory 的 swap time 可能過多
        * 硬碟移回 memory 的位置要放在哪裡
    * ![image](https://hackmd.io/_uploads/SJFKYSpfA.png)
* Mobile System
    * 通常不使用 swapping 機制
        * 本身 memory 就有限
        * 並且使用 flash memory ，如果一直 swapping 會壞掉
        * flash memory 跟 CPU 之間的傳輸太久
    * 如果 memory 不足
        * 強制某些 process 釋放 memory
# Chapter 10 : Virtual Memory
* 把不常用到、沒那麼急的 process 放到硬碟
* 同一時間有更多的 process share
* 節省大量 Main Memory
* 執行較快，只需要 load 一小部分就可以開始執行了
## Demand Paging
![image](https://hackmd.io/_uploads/SJB2bIpMC.png)
* Page is needed => reference to it
    * invalid reference => abort
    * not-in-memory => bring to memory
* Lazy swapper (Pure demand paging)
    * 用到的前一刻才 load，否則全部都放在硬碟
* 要怎麼知道要預先 load 哪些
    * page table 可以記錄哪個 page 在 memory，哪個不在 memory
* Valid-Invalid Bit
    * 原本
        * 區別 page 是不是允許 access 的範圍
    * 現在
        * 區別是否 load 進 memory
        * 但是要先確定這些 page 都是可以 access 的
* Steps in Handling Page Fault
> 1. 當取用 reference 的時候，page table 上面對應到的關係是 invalid，觸發 Page fault
> 2. OS kernel 會去判斷 reference 是不屬於存取範圍，還是只是還沒 load 到 memory
> 3. 如果只是還沒 load ，就找一個空的 frame
> 4. 從硬碟 load 進 memory
> 5. 填入 page table 並且將 bit 設為 valid
> 6. 重新執行觸發 Page fault 的指令
> ![image](https://hackmd.io/_uploads/HJHFL86fC.png)
* 問題
    * 硬體是否支援
    * Page Fault 太高會導致速度變慢
    * Instruction Restart
        * 有些指令可能執行到一半遇到 Page Fault
        * 但如果重新執行的話會造成一些 error
    * free-frame 怎麼找
        * free-frame list 紀錄
        * zero-fill-on-demand : 在配置之前先清空 frame
* Performance of Demand Paging
    * Three major activities
        * interrupt
        * Disk load to Memory
        * Restart process
    * Effective Access Time(EAT)
        > Page Fault Rate : 0~1
        > ![image](https://hackmd.io/_uploads/rkmdq8pMR.png)
## Copy-on-Write (COW)
* 在 share pages 的過程中，如果有一個 process 被修改了
    * 把 process copy 出來進行修改
## Page Replacement
* 如果沒有可用的 free frame 情況
    * 把一個不急著用、沒有在用的位置替換掉
### 要怎麼知道不急著用、沒有在用的位置是哪個?
* modify (dirty) bit : 判斷是否有修改過
    * 可以直接丟掉
    * 減少寫入 disk 的時間
* page-replacement algorithm
    reference string : 判斷接下來會存取的 page number 順序
    * FIFO
        > 把最先進來的踢掉
        > ![image](https://hackmd.io/_uploads/S1-Z-vaGC.png)
        * Belady's Anomaly
            > 增加 page frames 但 page faults 增加了
            > ![image](https://hackmd.io/_uploads/HkXBWw6zC.png)
    * Optimal
        * 理想情況，實際無法達成
        * 因為實際情況不知道下個準備要執行哪個 frame
        > 把最久用不到的踢掉
        > ![image](https://hackmd.io/_uploads/SJW6bwaMA.png)
    * LRU
        * implementation
            * Counter : 紀錄 page 被 accessed 的時間
            * Stack : 被 accessed 的 page number 移到最上面
        > 看過去使用的狀況，挑最久沒被用到的
        > ![image](https://hackmd.io/_uploads/HkAU0k8QR.png)
    * LRU Approximation
        * 改善 LRU 變成實際可以使用的演算法
            * 因為 LRU 效率太差，會 accessed 很多次 memory
            * 起始 bits 全部都設為 0
            * 當 access 到的 page 設為 1，否則為 0，過一段時間會重新檢查(設為 0)
            * 問題
                * 可能會很多 0，無法知道先後順序
                * 太久檢查一遍會太多 1，太快檢查一遍會太多 0
        * Second-chance algorithm
            * 加入順序概念，並且檢查完最後一個，就直接從頭重新檢查，反覆循環
            * 如果要替換的時候遇到 1，代表他再檢查後有被使用到
            * 那就把他設為 0，但不替換他，接著繼續找 bit 為 0 的 page 替換
        * Enhanced Second-Chance
            * 多一個 modify bit 確認要被替換的 bit 有沒有被 modify 過
            * 有的話就 swap out 到硬碟上
            * 沒有的話直接丟掉
            * ![image](https://hackmd.io/_uploads/BJBN4xIX0.png)
    * Counting Algorithms
        * Least Frequently Used (LFU) Algorithm
            * 替換掉過去被用到的次數比較少的 (假設未來也比較不會用到的)
        * Most Frequently Used (MfU) Algorithm
            * 替換掉過去被用到的次數比較多的 (假設可能跳出迴圈或 function 等等，已經用不到了)
## Allocation of Frames
### Global vs. Local Allocation
* Global replacement : 從全部的 page 中選擇做替換
    * Reclaiming Pages : 當記憶體空間小於一個門檻值就會觸發 replacement
* Local replacement : 只從自己用到的 page 
    * 不會影響到其他 process ，效率較穩定
    * 但因為只考慮部分的 page ，所以可能記憶體使用率較低
## Thrashing
* 如果在替換掉 page 後，馬上又要用到被替換的 page
* 導致 page fault 不斷上升
* 又導致 cpu 使用率下降，所以電腦為了讓 cpu 忙一點，會塞入更多的 process 給 cpu
* 所以在 cpu 更忙的情況下， page fault 又上升造成死循環
* ![image](https://hackmd.io/_uploads/r1GsGWLQ0.png)
### Demand Paging and Thrashing
* Locality model : 執行 damand page 時是區域、依序執行的，意旨執行完前一個 page 才會再執行下一個 page
    * 當 locality 的總大小 > 記憶體全部的大小 => 造成 thrashing
* 解決辦法
    * Working-Set Model : 
        * 計算每個 process 需要用到多少 page，觀察是否快造成 thrashing
    * Page-Fault Frequency : 
        * 觀察 page fault 頻率
        * 如果很低，代表應該還行，不會造成 thrashing
        * 如果高到一個程度，代表快 thrashing ，就要進行處理
# Chapter 11 : Mass-Storage Systems
## HDD Scheduling
* Performance
    * 大部分的時間都在旋轉跟移動讀寫頭
    * ![image](https://hackmd.io/_uploads/rynaZU5NR.png)
### Disk Scheduling
* FCFS
* SCAN (elevator)
    * 不要一直來回反覆移動，盡量只往一個方向移動
    * 問題 : 如果 queue 平均分布，另一半的 request 會等很久
    * ![image](https://hackmd.io/_uploads/ryRpvIcNA.png)
* C-SCAN
    * 只往一個方向處理
    * 一個方向到底後退到 0 從頭開始
    * ![image](https://hackmd.io/_uploads/BJBGj85VC.png)
## NVM Scheduling
* 優 : 不易損毀、速度快
* 缺 : 壽命短、不能 rewrite
* Controller Algorithms
    * garbage collection : 當某個程式占用的一部分主記憶體空間不再被這個程式訪問時，這個程式會藉助垃圾回收演算法向作業系統歸還這部分主記憶體空間。
    * wear leveling : 不要重複在同一個地方寫入太多次，要平均分散在不同的地方，不然那個地方容易壞掉。
* Volatile Memory
    * 將 ram 的一部分當作 driver 使用
    * 與 cache, buffer 的差異
        * cache, buffer 由 OS 管理，使用者不能控制
        * Volatile Memory 可由使用者自行管理
## Storage Device Management
* Partition
    * 切割不同用途
* Logical formatting
    * 檔案系統的方式儲存
    * 電腦中的格式化是指這個
## Swap-Space Management
* 不是用檔案系統儲存，而是以 page 形式儲存
## Storage Attachment
### NAS
* 透過 NFS, CIFS 協定實作 
### Cloud
* 透過 API request
## RAID Structure
* Disk striping : 將好幾顆硬碟當作一顆硬碟使用
* Mirroring : 複製到幾顆硬碟上
* RAID 0 : non-redundant striping
    * 將好幾顆硬碟當作一顆硬碟使用
    * 不能有任何一顆壞掉，否則資料會遺失
* RAID 1 : mirrored disks
    * 原本用 4 顆硬碟
    * 現在用 8 顆硬碟，多的 4 顆跟前 4 顆資料一模一樣
* RAID 4 : block-interleaved parity
    * 多一個硬碟紀錄 parity (check sum)
    * parity 能夠知道資料是否錯誤
    * 如果 parity disk 壞掉就沒辦法驗算資料
* RAID 5 : block-interleaved distributed parity
    * 每一個 disk 都有 parity
* RAID 6 : P + Q redundancy
    * 多一個 parity 驗算並且存在每一個 disk
* RAID 10 (RAID 1+0) : Striped mirrors
    * 先複製再串起來
* RAID (0+1) : mirrored stripes
    * 先串起來再複製

# Chapter 13 : File-System Interface
## Access Methods
* Sequential Access : 指資料的讀取或寫入必須按照資料在儲存介質上的物理順序進行。要訪問某一特定資料單元，必須先訪問該單元之前的所有資料。
    * 訪問速度較慢，因為需要依次讀取或寫入資料。在讀取大量連續資料時效能較好，但在訪問隨機資料時效能較差。
    * 優點：適合大數據量的連續讀取和寫入，結構簡單，成本低。
    * 缺點：隨機存取效能差，訪問特定資料需遍歷前面的資料。
* Direct Access : 指可以隨機地訪問儲存介質上的任意資料單元，而不必按照順序進行。可以直接跳轉到所需的資料單元，進行讀取或寫入操作。
    * 訪問速度較快，可以隨機訪問任意資料單元，適合頻繁隨機讀取或寫入的應用場景。
    * 優點：隨機訪問效能高，適合頻繁且隨機的讀寫操作。
    * 缺點：結構相對複雜，成本較高。
## Disk and Directory Structure
### Single-Level Directory
* Naming problem : 名字容易重複
* Grouping problem : 不同功能的檔案混在一起
### Two-Level Directory
* Path name : 解決名字問題
* Efficient searching
* 無法解決 Grouping problem
### Tree-Structured Directories
* 不同目錄底下放不同用途的檔案
* Efficient searching
* 解決 Grouping problem
### Acyclic-Graph Directories
* 不同目錄底下可以共用同個檔案
    > 如下圖 count 
    > ![image](https://hackmd.io/_uploads/HylW_co4R.png)
* 問題
    * 同個檔案有多個名稱，刪除、編輯等等會有問題
        * Link 同步
### General Graph Directory
* 會有 cycle
    > 如下圖
    > ![image](https://hackmd.io/_uploads/r1VyK5iE0.png)
* 解決
    * link 指向檔案，不能指向sub directory
## File-System Mounting
* 指將一個文件系統（例如一個磁碟分區、USB 驅動器或網絡共享）連接到操作系統的目錄樹中的一個目錄，使其成為該目錄的子樹，從而使該文件系統的內容可以通過該目錄進行訪問。
## File Sharing
* 同一個檔案可以讓不同使用者使用，只是有不同的權限
* 透過 User IDs, Group IDs 判斷權限
* 問題
    * 權限
    * 遠端
    * 同步
## Protection
### Access Lists and Groups
* Mode of access : read, write, execute
* Three classes of users on Unix/Linux
    * owner, group, public
* example
    * chmod 761 game
    * mean
        * owner => 111 => can r,w,x
        * group => 110 => can r,w
        * public => 001 => can x

# Chapter 14 : File System Implementation
## File-System Structure
* File control block(FCB) : 紀錄檔案資訊
### File System Layers
* application programs -> logical file system -> file-organization module -> basic file system -> I/O control -> devices
    * Logical file system
        * 把檔名轉換成檔案編號
    * file-organization module
        * 轉換 logical block 到 physical block
        * 管理 free space, disk allocation
    * basic file system
        * 接收指令並告訴 device driver
        * 將常用檔案暫存在 caches
        * 將檔案放在 buffer
    * Device drivers
        * 實作指定(read, write)
## File-System Operations
### On-disk Structures
* Boot control block
    * 開機所需要的檔案都放在裡面
    * 通常是第一個 block
* Volume control block
    * 紀錄有多少 block, 有多少可用 block, 每個 block 多大等等
* Directory structure
    * 紀錄整個檔案的分布情況
* Per-file File Control Block(FCB)
    * 紀錄每個檔案個別的資訊
### In-Memory File System Structures
* Mount table
    * 紀錄開機時所需要的檔案資訊
* system-wide open-file table
* per-process open-file table
## Directory Implementation
* Linear list
    * 將所有檔案資訊都記錄起來
    * 簡單、花時間
* Hash Table
    * 速度快，但可能會有碰撞
## Allocation Methods
### Contiguous
* 將 block 依序擺放
* 簡單 : 紀錄開始位置跟長度就可以了
* 問題 : external fragmentation
* 解決 : 
    *  Extent-Based Systems
        * 將多個小空間串在一起變成大空間
### Linked (很像 link list)
* 從第一個 block 往後串，紀錄下一個 block 的 pointer
* 優點 : 不會有 external fragmentation
* 缺點 : 
    * 不可靠，如果中間有一個壞掉，會導致後面的 block 都找不到
    * 速度慢
### Indexed
* 將所有可用位置的 pointer 集中起來放在 index table(index block)
* 只需要拿到 index table 就可以拿到所有 pointer
* 優點 : 速度快
* 缺點 : index table 不能不見、壞掉
## Free-Space Management
* Bit vector 紀錄目前可用的欄位，但需要額外空間儲存
* Link list 
    * 優點 : 不會浪費空間
    * 缺點 : 速度慢
* Grouping
    * 多個 block 為一組，一次記錄他們的情況
### TRIMing
* 針對 SSD ，因為 NVM 無法做覆寫
* 所以當有 free 的位置出來後，只需要紀錄位置，不需要真的清除該位置的內容
## Efficiency and Performance
* 盡量讓 data 跟 metadata 放一起
    * 讀到檔名後，可以更快速的讀取接下來的內容
* Buffer cache 
    * 讓經常使用到的資輛放在記憶體
* Synchronous
    * update 的時候要一直寫回去
    * 可設為 asynchronous ，變成全部 update 結束後再一起寫回去
* read-ahead 
    * 如果之後可能要用到的話，先預先讀近來
* Unified Buffer Cache
    * 將重複的 cache 整合成一個 cache
## Recovery