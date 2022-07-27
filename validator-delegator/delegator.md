# 委托人

成為 BitTorrent-chain 的委托人沒有先決條件。您所要做的就是擁有一個TRON帳戶。

委托人無需托管完整節點即可參與驗證。他們可以將BTT代幣投票給驗證人，並獲得部分獎勵作為交換。因為他們與驗證人共享獎勵，所以委托人也分擔了風險。委托人在系統中起著至關重要的作用，因為他們可以根據自己的意願選擇驗證人。

## 投票相關合約接口說明
### 為驗證人投票
* 合約方法：ValidatorShare:buyVoucher(uint256, uint256)
* 參數：
    * _amount：投票數量
    * _minSharesToMint：可接受的最少份額幣數量，委托人為驗證人投票的BTT會轉化為份額幣，以表示用戶為驗證人所投票數占總票數的份額。委托人可通過驗證人的ValidatorShare合約的balanceOf方法來查詢他所擁有的份額幣數量。
* 說明：
    1. 在調用buyVoucher方法之前，需要先調用[`BTT`](https://tronscan.org/#/contract/TAFjULxiVgT4qWk6UZwjqwZXTSaGaqnVp4/code)的approve方法授權大於投票數量的BTT給[`StakeManagerProxy`](https://tronscan.org/#/contract/TEpjT8xbAe3FPCPFziqFfEjLVXaw9NbGXj/code)合約。
    2. 每一個驗證人都有對應的ValidatorShare合約， 可以訪問StakeManagerProxy的validators[validatorId].contractAddress來獲取某一個驗證人對應的ValidatorShare合約地址
    3. 此方法也可為驗證人追加投票


### 領取獎勵
* 合約方法：ValidatorShare:withdrawRewards()
* 參數：無
* 說明
    1. 委托人調用驗證人的ValidatorShare合約的withdrawRewards方法來為提取獎勵，執行成功後獎勵立刻到達委托人賬戶。

### 取消投票
* 合約方法：ValidatorShare:sellVoucher_new:(uint256, uint256)
* 參數：
    * uint256 claimAmount：取消投票的BTT數量；委托人為驗證人投票的總BTT數量，可通過ValidatorShare:getTotalStake方法獲取。
    * uint256 maximumSharesToBurn：可接受燃燒的最大份額幣數量；委托人所擁有的份額幣數量可通過ValidatorShare:balanceOf方法獲取。
* 說明
    1. 取消投票可以分多次進行，但是每次之間至少間隔1個檢查點。
    2. 取消投票後，質押金BTT需要經過80個檢查點的鎖定期，才可提取。


### 提取投票所質押的BTT
* 合約方法：ValidatorShare:unstakeClaimTokens_new(uint256) 
* 參數
    * uint256 unbondNonce：`取消投票`操作對應的nonce，即提取第nonce次`取消投票`的BTT。委托人總的取消投票次數可以通過ValidatorShare:unbondNonces方法查詢。
* 說明
    1. 該方法需要在取消投票後，經過80個檢查點的鎖定期後，才可調用。


### 獎勵覆投
獎勵覆投是將已獲得但未提取的BTT投票獎勵再次投票給驗證人，以獲取更多的投票獎勵。
* 合約方法：ValidatorShare:reStake()
* 參數：無


### 轉移投票
轉移投票是轉移一部分票數給另一個驗證人。
* 合約方法：StakeManagerProxy:migrateDelegation(uint256, uint256, uint256)
* 參數
    * uint256 fromValidatorId：源validator id
    * uint256 toValidatorId：目標validator id
    * uint256 amount：轉移的BTT數量
* 說明
    1. 只能轉移給validatorID 大於7的驗證人
    2. 轉移投票會自動觸發領取獎勵操作
