# 驗證人

驗證人(Validator)是網絡中的參與者，他將代幣鎖定在網絡中並運行驗證人節點以幫助運行網絡。驗證人有以下職責：

* 質押網絡令牌並運行驗證人節點以作為驗證器加入網絡
* 通過驗證區塊鏈上的狀態轉換獲得質押獎勵
* 因停機等活動而受到處罰

區塊鏈驗證人是負責驗證區塊鏈內交易的人，對於 BitTorrent-Chain，任何參與者都可以通過運行全節點獲得獎勵和交易費用，從而有資格成為BitTorrent-Chain的驗證人。BitTorrent-Chain中的驗證人是通過定期發生的鏈上拍賣過程選擇的，這些選定的驗證人將作為區塊生產者和驗證者參與。

Validator需要有兩個地址：

1. Owner 地址：驗證人可以通過該地址處理`與管理相關`的功能，比如取消抵押、獲取獎勵、設置委托參數。
2. Signer 地址：驗證人從這個地址簽署檢查點並運行節點。

## 架構

BitTorrent-Chain 網絡分為3層

### Root Contracts

TRON及其他區塊鏈網絡上的Root合約，支持用戶通過存取款的方式將代幣映射到 BitTorrent-Chain，及支持質押等功能。

### Validator

驗證BitTorrent-Chain區塊，定期發送Checkpoint至支持的TRON及其他區塊鏈網絡。

**Bridge**：負責監聽各鏈路事件，發送事件消息等。

**Core**：共識模塊，包括Checkpoint(BitTorrent-Chain鏈的狀態快照)的驗證，Statesync事件\&Staking事件的共識。

**REST-Server**：提供相關API服務。

### BitTorrent-Chain

BitTorrent-Chain層的區塊生產者是驗證者的一個子集，由驗證人定期改組。

![](https://i.imgur.com/Ohh3DKD.png)

## 功能

區塊鏈驗證人是負責驗證區塊鏈內交易的人。對於BitTorrent-Chain來說，任何參與者都有資格成為BitTorrent-Chain的驗證人，通過運行一個完整的節點來獲得獎勵和交易費用。為了確保驗證人的良好參與，他們鎖定了他們的一些BTT代幣作為生態系統的股份。

BitTorrent-Chain的驗證人是通過鏈上的質押來選擇的，這個過程會定期進行。這些被選中的驗證人，作為區塊生產者和驗證者參與其中。一旦一個檢查點（一組區塊）被參與者驗證，那麽就會在TRON&以太坊\&BSC上進行更新，根據驗證人在網絡中的股份，為其發放獎勵。

### 驗證人的職責

* 通過在TRON上的質押合約中鎖定BTT代幣來加入網絡。
* 驗證人可以隨時退出系統，可以通過unstake 在合約上執行交易來完成，
* 驗證人可以隨時增加質押BTT代幣數量，以增加質押能力。
* 設置驗證人節點後，驗證人將執行以下操作： 1.區塊生產者選擇 2.在BitTorrent-Chain 上驗證塊 3.檢查點提交 4.在以太坊上同步對BitTorrent-Chain 質押合約的更改 5.從TRON&以太坊\&BSC 到BitTorrent-Chain層的狀態同步
* 驗證人需要保持最低數量的代幣來支付相關鏈上的交易費用。

### Validator層

Validator層將BitTorrent-Chain產生的區塊聚合成默克爾樹，並定期將默克爾根發布到根鏈。這種定期發布被稱為“檢查點”。對於 BitTorrent-Chain 上的每幾個區塊，一個驗證人（Validator）：

1. 驗證自上次檢查點以來的所有塊
2. 創建塊哈希的默克爾樹
3. 將merkle root發布到主鏈

檢查點很重要，原因有兩個：

1. 在根鏈上提供終結性
2. 在提取資產時提供銷毀證明

### BitTorrent-Chain層

BitTorrent-chain層中的區塊生產者，BitTorrent-chain層中的VM與EVM兼容，是一個基本的Geth實現，並對共識算法進行了自定義修改。

### 檢查點機制(Checkpoint)

在Validator中通過Tendermint的加權輪回算法來選擇一個提議者，在Tendermint上成功提交一個檢查點有2個階段的提交過程，一個是通過上述Tendermint算法選擇的提議者發送一個檢查點，在提議者字段中包含他的地址，所有其他提議者在將其添加到他們的狀態中之前將對此進行驗證。

然後下一個提議者發送一個確認交易，以證明之前的檢查點交易在以太坊主網中已經成功了。每一個驗證者集的變化將由Validator上的驗證人節點轉發，該節點被嵌入到驗證人節點上。這使得驗證人在任何時候都能與TRON\&Ethereum等鏈上的BitTorrent-chain合約狀態保持同步。

部署在TRON\&Ethereum等鏈上的BitTorrent-chain合約被認為是最終的真相來源，因此所有的驗證都是通過查詢TRON\&Ethereum等鏈上的BitTorrent-chain合約完成的。

### 交易費用

BitTorrent-chain層的每個區塊生產者都將獲得每個區塊收取的一定比例的交易費用。

### 狀態同步機制

Validator層上的驗證人接收StateSynced事件並將其傳遞給BitTorrent-chain層。

接收者合約繼承了IStateReceiver,相關自定義邏輯位於onStateReceive函數內。

Dapp/用戶 需要做的事情是與state-sync 一起工作。

1. 調用StateSender合約的 `syncState()`函數。
2. 上述函數將觸發`StateSynced(uint256 indexed id, address indexed contractAddress, bytes data);`事件
3. Validator層上的所有驗證人都會收到這個事件。
4. 一旦Validator層上的狀態同步交易被包含在一個區塊中，它就會被添加到待定狀態同步列表中。
5. BitTorrent-chain層節點通過API調用從Validator上獲取待定的狀態同步事件。
6. 接收者合同繼承了IStateReceiver接口，解碼數據字節和執行任何行動的自定義邏輯位於onStateReceive函數中。

## 質押相關合約接口說明

### 質押

* 合約方法：StakeManagerProxy:stakeFor(address, uint256, uint256, bool, bytes memory)
* 參數
  * address user：質押賬號地址，即驗證人的owner地址
  * uint256 amount：質押的BTT數量
  * uint256 deliveryFee：中間層手續費
  * bool acceptDelegation：驗證人是否接受委托人的投票；建議將其設置為true，即接收委托人的投票，之後可以通過ValidatorShare:updateDelegation方法關閉或開啟；但是如果調用 stakeFor 時 acceptDelegation 是 false，之後就無法再更改，也就無法再接受投票。
  * bytes memory signerPubkey：簽名賬戶公鑰；即驗證人的signer地址的公鑰，需要把前導“04”去掉
* 說明
  1. 參數`amount`允許的最小值可通過StakeManagerProxy:minDeposit方法查詢（目前為10^30， 也就是10^12個BTT）。
  2. 參數`deliveryFee`允許的最小值可通過StakeManagerProxy:minHeimdallFee方法查詢（目前為10^23，也就是100000個BTT）。
  3. 在調用stakeFor方法之前，需要先調用[`BTT`](https://tronscan.org/#/contract/TAFjULxiVgT4qWk6UZwjqwZXTSaGaqnVp4/code)的approve方法授權大於質押數量的BTT給[`StakeManagerProxy`](https://tronscan.org/#/contract/TEpjT8xbAe3FPCPFziqFfEjLVXaw9NbGXj/code)合約。
  4. 用戶質押成功後，可通過stakeManagerProxy:getValidatorId方法獲取到驗證人的validatorID，然後通過stakeManagerProxy:validators方法獲取到validator的詳細信息。

### 追加質押

* 合約方法：StakeManagerProxy:restake(uint256，uint256，bool)
* 參數
  * uint256 validatorId：質押的validator id
  * uint256 amount：質押數量
  * bool stakeRewards：獎勵是否加入質押
* 說明
  1. 調用此方法的前提條件是成功調用了StakeManagerProxy:stakeFor方法並成為驗證人。

### 領取獎勵

* 合約方法：StakeManagerProxy:withdrawRewards(uint256)
* 參數
  * uint256 validatorId：領取獎勵的validator id
* 說明
  1. 驗證人可通過withdrawRewards方法來領取獎勵，執行成功後獎勵立刻到達驗證人賬戶。

### 取消質押

當驗證人想退出系統，停止驗證區塊和提交檢查點時，驗證人可以取消質押。為了保證良好的參與度，取消質押的驗證人的質押部分代幣將被鎖定withdrawalDelay個周期。

* 合約方法：StakeManagerProxy:unstake(uint256)
* 參數
  * uint256 validatorId：解除質押的validator id
* 說明
  1. 驗證人可通過unstake方法來取消質押，取消質押後立刻返還獎勵的代幣到驗證人賬戶，但質押的代幣需要通過unstakeClaim函數來申領。
  2. unstakeClaim方法必須等待withdrawalDelay（目前為80）個檢查點後才可以被調用。

### 領取質押的BTT

* 合約方法：StakeManagerProxy:unstakeClaim(uint256)
* 參數
  * uint256 validatorId：領取質押金的validator id
* 說明
  1. 在取消質押後，需要等待withdrawalDelay（目前為80）個檢查點後，才可以調用此方法來領取之前質押的BTT。

### 更新validator簽名公鑰

* 合約方法：StakeManagerProxy:updateSigner(uint256，bytes memory)
* 參數
  * uint256 validatorId：validator id
  * bytes memory signerPubkey：新簽名公鑰
* 說明
  1. 驗證者可以更新簽名賬戶，但是兩次更新操作的時間間隔需要滿足大於signerUpdateLimit（目前為100）個檢查點。

### 更新傭金比例

* 合約方法：StakeManagerProxy:updateCommissionRate(uint256，uint256)
* 參數
  * uint256 validatorId：validator id
  * uint256 newCommissionRate：新傭金比例
* 說明
  1. 驗證者可以更新傭金比例，但是兩次更新操作的時間間隔需要滿足大於WITHDRAWAL\_DELAY（目前為80）個檢查點。
  2. 傭金比例需要小於等於100
