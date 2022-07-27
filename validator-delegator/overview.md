# overview

### 概述

BitTorrent-Chain 是一個區塊鏈應用平台,如果您希望通過為BitTorrent-Chain設置節點來成validator,或者希望成為委托人以將代幣委托給validator並獲得獎勵，可以通過該文檔進行快速了解相關內容。

### PoS、質押和投票

#### 股權證明(PoS)

權益證明 (PoS) 是公共區塊鏈的一類共識算法，取決於驗證人在網絡中的經濟權益。在基於工作量證明 (PoW) 的公共區塊鏈（例如比特幣和以太坊的當前實現）中，該算法獎勵解決密碼難題的參與者，以驗證交易並創建新塊（即挖礦）。在基於 PoS 的公共區塊鏈中，一組驗證人輪流對下一個區塊進行提議和投票，每個驗證人投票的權重取決於其存款（即權益）的大小。PoS 的顯著優勢包括 安全性、降低中心化風險和能源效率。

有關更多詳細信息，請參閱 [https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ](https://github.com/ethereum/wiki/wiki/Proof-of-Stake-FAQ)。

#### 質押

Staking 是指將代幣鎖定到存款中以獲得在區塊鏈上驗證和生產區塊的權利的過程。通常，質押是在網絡的本機令牌中完成的。

#### 投票

投票是代幣持有者將其股份委托給驗證人的過程。它允許不具備運行節點的技能或願望的代幣持有者參與網絡，並根據投票的股份數量按比例獲得獎勵。

### 架構

BitTorrent-Chain 是一個區塊鏈應用平台，整體結構分為三層：

* Root Contracts層：TRON及其他區塊鏈網絡上的Root合約，支持用戶通過存取款的方式將代幣映射到 BitTorrent-Chain，及支持質押等功能。
* Validator層: 驗證BitTorrent-Chain區塊，定期發送Checkpoint至支持的TRON及其他區塊鏈網絡。 **Bridge**：負責監聽各鏈路事件，發送事件消息等。 **Core**：共識模塊，包括Checkpoint(BitTorrent-Chain鏈的狀態快照)的驗證，Statesync事件\&Staking事件的共識。\
  **REST-Server**：提供相關API服務。
* BitTorrent-Chain層。
