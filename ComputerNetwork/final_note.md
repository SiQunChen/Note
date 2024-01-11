# Chapter 4 Network Layer : Data Plane

## Two key network-layer functions
* forwarding : 把封包從input port 轉到 output port
* routing : 處理整個路徑的規劃
## Network-layer : data plane, control plane
* Data plane : 針對每個packet要怎麼做轉送
* Control plane : 從A到B要怎麼做路徑規劃
    * traditional routing algorithms:
        * 每個router根據目前網路狀況決定怎麼走 (**目前普遍使用**)
    * software-defined networking (SDN):
        * 有一個上帝(**Remote Controller**)決定怎麼走
## Network-layer : service model
* Reflections on best-effort service
    * 最簡單
    * 現在網路基本頻寬都很高，如果頻寬很好的話，架構簡單沒問題
    * application-layer提供服務(datacenters, CDN)
    * 有一些控制的功能
## Router architecture : 多個入口選擇多個出口
### Input:
* 特點:
    * 硬體
    * 馬上判斷
    * 查看destination決定出口
* Input port queuing
    * data速度 > switch速度 => packet來不及處理故存在buffer
    * Head-of-the-Line (HOL) blocking => 2個input要送到同個output，其中一個要先queuing，但會慢一個時間
        * 解決 : switching要夠快 
* forwarding:
    * 根據20位後決定destination
    * longest prefix matching(就是if...else...)
        * 一層一層比較，比前面就好，選擇符合比較長的bit
### Switching fabrics:
* 特點:
    * 軟體
    * 不需馬上判斷
    * 速度快(通常 > input rate)
* 方式:
    * CPU, memory : n個input, 2n倍速度 => CPU負擔大
    * bus開門 : n個input, n倍速度 => 要求依舊高
    * interconnection network(九宮格)開門 : 同時 => 一個input速度
### Output port:
* Buffer Management
    * Drop : if buffer full
        * tail drop : 先來先進，已經滿的話就不給進
        * priority : drop權重低的
    * marking : 對封包做標記，方便管理
* scheduling discipline : 選擇要從buffer裡送什麼資料出去
    * FCFS(FIFO)
    * priority
    * round robin : 分類照順序走
    * weighted fair queueing : 
        結合priority、round robin，每個class都保證有一個最小頻寬 => 一個cycle每個queue拿到的比重![image](https://hackmd.io/_uploads/rJ8Dhv24T.png)

* buffer : 因為一次只能output一個data，所以多的要存在buffer
    * How much buffering?
        * RTT * C(link capacity)
        * with N flows : (RTT * C)/sqrt(N)
    * but too much buffering can increace delays
## IP Datagram format
### 重點 : 
* IP 內容原則上最多 1500 bytes
* 前兩列 : 標頭長度、封包長度
    * 標頭 : 20 bytes of TCP or 8 bytes of UDP + 20 bytes of IP = 40 bytes or 28 bytes
* 第三列 : 
    * TTL : packet 最多能經過幾個router，每經過一個 router 就會 -1 ，變 0 後在下個 router 就會被丟掉 => 防止無窮迴圈
    * upper layer : 查看是什麼類型的封包
    * checksum : 查看傳輸有沒有錯誤(經過router時會改變，因為 TTL -1)
* 第四、五列 : client, server 的 IP
* 後兩列 : 不重要
![image](https://hackmd.io/_uploads/HkOJSder6.png)
## IP addressing
* IPv4 : 32 bits
* CIDR : 用於IP地址分配的機制 => a.b.c.d/x
* How does host get IP address?
    * 手動設定
    * DHCP : 自動找沒人使用的 IP 來用
        * 有時限 (lifetime)
        * can renew ( 有可能會跟原本的一樣 )
        * overview : discover -> offer -> request -> ack
        ![image](https://hackmd.io/_uploads/H1vicKmrp.png)
        
## Subnets
* 同一個 subnets 的 host 不需要經過 router 協助，就可以把 data 送到同個 subnets 的其他 host
* example : **140.124.181.139** => **140.124.181** 是他的subnet， **139** 是他的host
* example : **140.124.181.0/24** => **/24** 代表 subnet mask，意思是前面 24 個 bits 代表 subnet
* 下圖有 6 個 subnets => router 與 router 之間也有一個 subnet
![image](https://hackmd.io/_uploads/ByJVxtmBa.png)

## NAT
* 一個私人網路裡所有的電腦經過路由器轉換後共用同一個IP (**but diffent port**)
* Q : 從路由器回傳資料要怎麼知道是哪台電腦? A : 利用 port 判斷
* 一個 port 經過路由器後 port 也會跟著改變，意思是私人網路跟外網表示同一台電腦的 port, IP 都是不一樣的
* advantage
    * 所有設備只需要從 ISP 取得 IP
    * 可以在不更改外網的情況下更改內網的位址
    * 可以在不更改內網的情況下更改外網的位址
    * 內網的設備不能直接尋址，所以較為安全
* 違反網路層的設計 => port number 不應該由網路層處理
* NAT traversal => 當 server 在 NAT 後所需要使用到的技術，將 server address 對外公布

## IPv6
### motivation
* 解決 IPv4 不夠的問題
* 能夠更有效率的傳輸
    * 統一標頭為 40 bytes
    * 增加了 flow 的概念
> ![image](https://hackmd.io/_uploads/SkxVwqDH6.png)
> 沒有 checksum => 光纖出錯率低
> 不做切割 => 切封包要更多時間
> 沒有 options 欄位，有要做其他功能就丟給上層做
### Transition from IPv4 to IPv6
* tunneling : IPv6 packet 包在 IPv4 packet 裡面
    * 對 IPv6 而言 : IPv4 扮演著 link layer
    * 對 IPv4 而言 : IPv6 扮演著資料內容
> ![image](https://hackmd.io/_uploads/BJ7hnxtSa.png)

## Generalized forwarding => 透過 SDN (軟體定義網路)
### match plus action
* destination-based forwarding : forward based on dest
* generalized forwarding : 
    * match : many header fields can determine action
    * actions : many action possible : drop / copy / modify / log packet
    * priority
    * counters

### OpenFlow abstraction
* Router
* Switch
* Firewall
* NAT

# Chapter 5 Network Layer : Control Plane
## routing protocols
![image](https://hackmd.io/_uploads/rkSlRCfUp.png)
* notation
    * C(x,y) => x 到 y 的距離
    * D(v) => 到目前 v 點的最短路徑
    * p(v) => 到前一個點的最短路徑
    * N' => 紀錄經過了哪些點

* link state
    * 如果用負載當作 cost ，可能會出現震盪
    * Dijkstra's link-state routing algorithm
        ![image](https://hackmd.io/_uploads/rJIyL1QIp.png)
* distance vector
    * 只適用於某些情況
    * 如果有 link cost changes ，則會陷入死循環
    * Bellman-Ford(BF)
        ![image](https://hackmd.io/_uploads/HkoM4eQIT.png)
## routing classification : 
### intra-AS routing : 在同一個 domain 進行傳送
* OSPF
    * open : publicly available
    * link-state : 檔案架在 IP 上，不需要透過 TCP/UDP 。因為 TCP/UDP 是 end-to-end 用的(client, server)，OSPF 是用在 router-to-router。
    * 可以分層架構
### inter-AS : 要傳送到其他的 domain

## Some methods
* BGP : 
    * 利用 policy 做 routing ，而不是最便宜的路徑。
    * eBGP : 外部
    * iBGP : 內部
* Hot potato routing (燙手山芋) :
    * 優先考慮同個 AS domain 的 weights

## SDN : 
### Data-plane switches : 
* 照表抄課
* 透過 southbound API (Openflow)與 controller 做溝通
* Data 轉送
### SDN controller (network OS) : 紀錄網路狀態
* 透過 northbound API 接受上層的指令控制下層的行為
* 分散式架構
* 如果上層表達沒有這麼明確，會透過 intent 進行運算執行下層的功能
### network-control apps : 
* 透過 northbound API 與下層做溝通
* 主要有 routing, access control, load balance

## Internet Control Message Protocol (ICMP) : router 與 router 之間傳遞訊息的 protocol
* UDP
* error reporting
* echo request/reply (used by ping) => 可以 ping 某個 router 是否還活著
* data 直接放在 IP 層上面
* Traceroute : 可以知道傳輸過程經過了哪些 router

## network managerment configuration
* 網管系統裡有管理的人、被管理的人、protocol ，他們互相之間要交換一些 data
### Approaches
* CLI (Command Line Interface)
* SNMP
    1. request, response
    2. trap message : device 自動送 message
* NETCONF/YANG : 可以一次對許多 device 做設定

# Chapter 6 The Link Layer and LANs
## itroduction : 2個相鄰的 devices 如何運作
* 方式
    * wired
    * wireless
    * LANs
* 單位 : frame
* sevices : 
    * 加標頭、加尾巴(CRC) => 18 bytes
    * channel access => 避免碰撞
    * MAC addresses
    * error control
    * half-duplex and full-duplex (半雙工、全雙工)
* Q:在哪裡運作?
    * A:網路卡(有些會跟主機板結合)
## Error detection : 
* 在 header 加 EDC 傳送，接受端檢查是否有問題
### Parity checking : 
* single bit parity : 
    * 給定位數的二進位數中1的個數是奇數還是偶數的二進位數
    * 缺點 : 只能知道錯誤但不知道哪裡錯
* two-dimensional bit parity : 
    * 二維陣列的 data ，可以知道哪個 bit 發生錯誤並加以修正
    * 缺點 : 他要加 9 個 detect bits
### CRC : 
* 原理 : 
    * 因為 => 被除數 / 除數 = 商 + 餘數
    * 所以 => (被除數 - 餘數) / 除數 => 可以被整除
* 方法 : 
    1. 被除數 D / 除數 G = 商 + 餘數 R
    2. 實際上傳出去的值為 D+R
    3. 接收端再用 (D+R) / G
    4. 整除的話代表沒有 error，則把 R 拿掉就為實際上的 Data
* example : 
    1. 因為 G 為 4 位數，所以在 D 後面補上 3 個 0(G - 1) => 右圖直式除法
    2. 接著用 D / G = (商不重要) + 011
    3. 把 D 後面 3 個 0 換成 011，所以傳出去的 data 是 101110**011**
    4. 接收端用 101110011 / 1001 ，如果整除代表沒有 error，於是把 011拿掉就為原本的data
    > ![image](https://hackmd.io/_uploads/SyGZXKrPT.png)
* 特色 : 
    * 非常方便
    * 可以偵測到小於 r+1 bits 的 burst errors(error 都聚集在一起)
        * 假設 r+1 bits = 4，則只要連續 errors 小於 4 個都能夠偵測到
## multiple access protocols (MAC protocols): 
* 只存在於區域網路 => 很多人要利用這個網路上網
* 方法
    * point-to-point
        * 封閉性 : 別人不能使用
    * broadcast
        * 無線網路
        * 開放性 : 大家共享
        * 必須有協調機制，才不會使數據撞在一起
* 分類
    * channel partitioning
        * 分成固定份量，假設四個人要用，就分成四份，如果有人不用就浪費掉。
        * 份量的依據可以是 time slots, frequency, code 等等
    * random access
        * 沒有固定份量，誰想用就用
        * 但有可能會有碰撞，發生碰撞再想辦法處理
        * example : 
            * ALOHA, slotted ALOHA
            * CSMA, CSMA/CD, CSMA/CA
    * taking turns
        * 綜合以上兩種方法的特色
        * 輪流使用
### Channel partitioning MAC protocols: 
* TDMA
    * 輪流訪問
    * 固定長度
    * 不用就浪費掉
* FDMA
    * 分為頻段
    * 固定頻段
    * 不用就浪費掉
### Random access protocols
* Slotted ALOHA : 
    * 理想 : 
        * 所有 frames 尺寸相同
        * 時間分成相等大小的時隙
        * 節點僅開始傳輸時隙開始
        * 節點同步
        * 如果2個以上的節點在時隙中傳輸，所有節點都會偵測到衝突
    * 實作 : 
        * 沒有衝突 : 節點在下個時隙傳送新幀
        * 發生衝突 : 節點在每個後續時隙以機率 p (隨機)重傳幀，直到成功。
    * 優點 : 
        * 單一節點可以在通道全速率下連續傳輸
        * 去中心化 : 只有節點中的槽需要同步
        * 簡單
    * 缺點 : 
        * 碰撞、浪費時隙
        * 空閒槽
        * 節點能夠在傳輸資料時偵測到衝突
        * 時鐘同步
    * 效率 : 
        * 假設 : N個節點有許多 frame 要傳送，每個節點在時隙中傳送機率為 p
        * 給定節點在時隙中成功的機率 : p(1-p)N-1
        * 任意節點成功的機率 : Np(1-p)N-1
        * 最大效率 : 求 Np(1-p)N-1 最大化的 P*
        * 對於許多節點，當 N 趨近無窮大時，取 Np*(1-p*)N-1 的極限，得最大效率 = 1/e = 0.37
        * 最多 : 37% 的時間用於有用傳輸的頻道
* Pure ALOHA : 
    * unslotted Aloha：更簡單，無需同步.
        * 當幀首次到達時：立即發送
    * 不同步時碰撞機率會增加 : 
        * 在 t0 發送的訊框與在 [t(0)-1,t(0)+1] 發送的其他訊框發生衝突
    * pure Aloha efficiency: 18%
* CSMA
    * simple CSMA : listen before transmit
        * 如果偵測到通道空閒 ： 傳送整個幀
        * 若偵測到頻道繁忙 ： 延遲傳輸
        * 不要打擾別人
    * CSMA/CD : CSMA with collision detection
        * 短時間內偵測到的碰撞
        * 衝突傳輸中止，減少頻道浪費
        * 碰撞偵測在有線中容易，在無線中困難
        * 有禮貌的健談者
    * collisions : 整個資料包傳輸時間被浪費
        * 傳播延遲意味著兩個節點可能聽不到彼此剛開始的傳輸
        * 距離和傳播延遲在決定碰撞機率中發揮作用
        * CSMA/CD 減少了衝突中浪費的時間 => 偵測到衝突時傳輸中止
    * 乙太網路CSMA/CD演算法
        1. NIC接收來自網路層的資料報，建立幀
        2. 如果網路卡偵測到通道 : 
            * 如果空閒：開始幀傳輸。
            * 如果忙：等到通道空閒，然後傳輸
        3. 如果 NIC 傳輸整個幀且沒有衝突，則 NIC 已完成該幀
        4. 如果NIC在發送時偵測到另一個傳輸：中止，發送堵塞訊號
        5. 中止後，NIC 進入二進位（指數）退避：
            * 在第 m 次衝突後，NIC 從 {0,1,2, …, 2m-1} 中隨機選取 K。 NIC等待K·512位元時間，返回步驟2
            * 更多衝突：較長的退避間隔
    * CSMA/CD 效率 : 
        * t(prop) = LAN 中 2 個節點之間的最大傳播延遲
        * t(trans) = 傳輸最大尺寸幀的時間
        * 效率達到 1 : 
            * 當 t(prop) 變為 0 時
            * 當 t(trans) 趨於無窮大時
            * 比 ALOHA 更好的效能 ： 而且簡單、便宜、分散
### Taking turns
* polling
    * 原理 : 
        * 一個主席輪流給麥克風，如果接到麥克風沒有話要講就還給主席，如果要講話限時 N 分鐘。
    * 缺點 : 
        * 給麥克風的時間會造成頻寬的浪費
        * 主席壞掉，整個系統會壞掉
* token passing : 
    * 原理 : 
        * 輪流傳麥克風，不用還給主席，直接把麥克風給其他同學
### Cable access network : 結合 FDM, TDM, random access
* downstream(下行) : 電視 => FDM
* upstream(上行) : 上網
    * 一段用 TDM => 實際傳送 data
    * 一段用 random access => request
## LANs
### addresses
* IP address : 
    * 32-bit
    * 十進制
    * 猶如住址(可改變)
* MAC address : 
    * 48-bit
    * 十六進制
    * 由 IEEE 提供
    * 猶如身分證號碼(不可改變)
* ARP : 
    * 原理 : 
        * 在區網內傳送訊息要判斷 MAC 而不是 IP
    * 問題 : 
        * 在區網內當 A 知道 B 的 IP，但不知道 B 的 MAC。
    * 解決 : 
        * 透過 ARP 得知 B 的 MAC (廣播封包)
    * 補充 : 
        * A 會把 B 的 MAC 存起來，但有時效，時效過了就會再問。
* 如果要傳送 data 到其他子網路
    1. A 先傳送到中間 router
        * MAC src : A
        * MAC dest : router
        * IP src : A
        * IP dest : B
    2. router 再傳送到 B
        * MAC src : router
        * MAC dest : B
        * IP src : A
        * IP dest : B
    * 補充 : 
        1. A 透過 DHCP 知道 router 的 IP
        2. 再透過 ARP 知道 router 的 MAC
### Ethernet
* 技術
    1. bus -> 90s
    2. switched -> now
        * switch 不會有碰撞，但依舊使用CSMA/CD，故傳送前依舊會確認是否能夠傳送
        * 假設 A, B 都要送到 C ，switch 有 buffer 能夠控制
        * 不是 broadcast
        * self-learning : 會將建立過的連線存進 switch table，但有 TTL ，所以會不斷學習
    3. connectionless : 不用建立連線，直接傳送資料
    4. unreliable : receiver 不會傳送 ACK or NAK
    5. 有許多不一樣的傳輸介質，傳輸速率也有很多種
* frame 架構
    * 18 bytes (6 dest address, 6 source address, 2 type, 4 CRC)
    * preamble(8 bytes) : 告訴 reciver 開始跟結束
* switch vs router
    * both are store-and-forward : 
        * router : 網路層 => 根據 IP 做 routing
        * switch : 連結層 => 做 data 轉送，使其不會車禍
    * both have forwarding tables : 
        * router : 看 IP
        * switch : 看 MAC
### VLANs
* 目的 : 
    * 分割區域 => 資安考量
* port-based VLAN : 
    > 透過一個 switch 跟一個 router，能夠分為兩個網段
    > ![image](https://hackmd.io/_uploads/HJW_0Bdup.png)
    > 也能與其他 switch 融合(透過 VLAN ID)
    > ![image](https://hackmd.io/_uploads/SkLJk8d_6.png)
### Datacenter networks
* network elements : 
    * Server racks(機櫃)
    * TOR switch : 每個機櫃一個，為那個機櫃進行資料的轉送
    * Tier-2 switches : 第二層的 switch，連接所有 TOR switch
    * Tier-1 switches : 第三層的 switch，連接第二層的 switch，會錯綜複雜的連接，以防一個壞掉就整個癱瘓
    * Border routers : 對外的 router
    * load balancer : 進來的 data 會經過這台電腦進行運算，決定他的路徑
    * ![image](https://hackmd.io/_uploads/rJfDWLdda.png)
### a day in the life of a web request
1. 電腦搜尋 www.google.com
2. DHCP request 獲取 IP
3. request 經過 UDP -> IP -> 最後到 Ethernet 發送 broadcast
4. DHCP server 送 ACK 回去告訴他 IP、DNS 的 name, address、router IP
5. ARP 確認 router 的 MAC
6. 發送 DNS 出去要到 google 的 IP
7. 建立 TCP 連線
8. 收到 ACK 後把 HTTP request 送出去
9. google 把網頁訊息送回去
# Chapter 7 Wireless and Mobile Networks
* wireless : 無線(不一定能移動)
* mobility : 移動(不一定是無線)
## wireless network : 
* wireless hosts : laptop, smartphone, IoT
* base station : 基地台上去大部分是有線，跟使用者連接是無線
* wireless link : 因為是 broadcast，所以會有碰撞要解決
* structure
    * infrastructure mode(基礎建設) : 
        * host 之間傳輸要透過基地台
        * handoff : 一台設備從 A 基地台到 B 基地台的過程
    * ad hoc mode : 
        * 不用透過基地台傳輸，設備之間能直接傳輸
### Wireless link characteristics : 
* decreased signal strength : 會隨著距離衰減
* interference from other sources : 很容易受到干擾
* multipath propagation : 延遲不一樣，可能造成訊號波形受到影響
* SNR 與 BER 的平衡
    * SNR : 訊號雜訊比
    * BER : 錯誤機率
    * given physical layer : 加強電力 -> 增加 SNR -> 減少 BER，但耗電快
    * given SNR : 增加 SNR -> 減少 BER
* Hidden terminal problem
    > A 看不到 C，C 看不到 A，當他們都要傳送到 B 時會發生碰撞
    > ![image](https://hackmd.io/_uploads/Sy887uOua.png)
* Signal attenuation
    > 因為訊號會衰弱，導致 A 不知道 C 有再傳，C 也不知道 A 有再傳
    > ![image](https://hackmd.io/_uploads/ByFg4_uup.png)
### CDMA
* 原理 : 
    * 每個人用不同的編碼
* 演算法 : 
    * encoding : 
        * original data * chipping sequence(變換很快的訊號) => 做內積
    * decoding : 
        * encoded data * chipping sequence(變換很快的訊號) => 做內積
    ![image](https://hackmd.io/_uploads/SkQZ_udO6.png)
## 802.11
* Channels
    * 分為不同頻率的頻道
        * AP Admin 會選擇頻率
        * 可能會跟鄰近的AP頻道相同導致互相干擾
    * 選擇一個 AP 上網
* scanning
    * passive scanning
        1. AP 傳送 frames 證明它存在
        2. 電腦發送請求上網
        3. AP 發送回應給電腦
        ![image](https://hackmd.io/_uploads/HkDDc__up.png)
    * active scanning
        1. 電腦主動說想要上網
        2. 附近的 AP 回應誰可以上網
        3. 電腦發送請求上網
        4. AP 發送回應給電腦
        ![image](https://hackmd.io/_uploads/SkmC9dudT.png)
* multiple access
    * 因為訊號衰弱、遮擋等等情況，導致 CSMA 的偵測會不準，故有了 CSMA/CA
    * CSMA/CA
        * sender
            1. 如果 channel 空閒的話傳送訊號，並不會持續監聽碰撞，因為也聽不到(訊號衰弱)=>沒有 CD
            2. 如果 channel 繁忙的話等待一段**空閒時間**(頻道中途繁忙的時間不算入)，如果沒有收到 ACK ，則等待一段時間後再次發送。
        * receiver
            * 收到的話發送 ACK
        * Avoiding collisions (optional)
            * 預約制
            * 傳送一個很小的封包(RTS)給 AP 預約
            * AP broadcast CTS
* frame : addressing
    * address1 : dest address
    * address2 : src address
    * address3 : router address
    * address4 : ad hoc address
* advanced capabilities
    * Rate adaptation
        * 原本是綠色線，但 client 跟 AP 距離變長導致 SNR 從 30 變 20，也會導致 BER 增加
        * 所以當 SNR 變 20 時，會改變傳輸頻寬變成紅色那條，進而維持 BER 是低的
        * 以此類推，可能會變藍色的線
        ![image](https://hackmd.io/_uploads/B182zYO_T.png)
    * power management
        * 節能 : 沒有傳 data 會進行休眠，要用到再起床
    


