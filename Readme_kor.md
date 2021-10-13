#### Simple blind auction sample
- 경매 기간이 끝날 때까지 입찰가가 비공개로 유지되는 경매를 실행합니다
- 공개 원장에 전체 입찰가를 표시하는 대신 구매자는 입찰이 진행되는 동안 다른 입찰가의 해시만 볼 수 있습니다
- 입찰 기간이 끝난 후 참가자들은 경매에서 승리하기 위해 자신의 입찰가를 공개합니다
- 경매에 참여하는 조직은 공개된 입찰가가 공개 원장의 해시와 일치하는지 확인합니다
- 가장 높은 입찰가를 가진 쪽이 이깁니다.
- 경매를 종료하는 거래를 승인하기 전에 각 조직은 동료에 대한 암시적 개인 데이터 수집을 쿼리하여 조직 구성원이 아직 공개되지 않은 낙찰가가 있는지 확인합니다. 낙찰된 입찰이 발견되면 조직은 승인을 보류하고 경매가 종료되지 않도록 합니다.

#### 체인코드 배포
cd fabric-samples/test-network
./network.sh up createChannel -ca
or 
./network.sh up createChannel -ca -s couchdb

./network.sh deployCC -ccn auction -ccp ../auction/chaincode-go/ -ccl go -ccep "OR('Org1MSP.peer','Org2MSP.peer')"

#### 애플리케이션 종속성 설치
cd fabric-samples/auction/application-javascript
npm install

#### 애플리케이션 ID 등록 및 등록
- Org1 및 Org2의 인증 기관 관리자를 등록
node enrollAdmin.js org1
node enrollAdmin.js org2

- 경매를 생성할 판매자 ID를 등록하고 등록합니다. 판매자는 Org1에 속합니다.
node registerEnrollUser.js org1 seller

- Org1에서 2명의 입찰자를 등록하고 Org2에서 다른 2명의 입찰자를 등록합니다.
node registerEnrollUser.js org1 bidder1
node registerEnrollUser.js org1 bidder2
node registerEnrollUser.js org2 bidder3
node registerEnrollUser.js org2 bidder4

#### 경매 만들기
- Org1의 판매자는 빈티지 성냥갑 그림을 판매하기 위해 경매를 만들고 싶습니다.
node createAuction.js org1 seller PaintingAuction painting

#### 경매에 입찰가 및 입찰
- 입찰자 지갑을 사용하여 경매에 입찰가를 제출할 수 있습니다.

#### 조직1에서 입찰자1로 입찰
- Bidder1은 그림을 800달러에 구매하기 위한 입찰을 생성합니다.
- 입찰가는 Org1 암시적 데이터 수집에 저장됩니다.
node bid.js org1 bidder1 PaintingAuction 800

export BIDDER1_BID_ID=8ef83011a5fb791f75ed008337839426f6b87981519e5d58ef5ada39c3044edd

- 입찰을 경매에 제출할 수 있습니다.
- 비공개 입찰 목록에 입찰가 해시가 추가됩니다.
node submitBid.js org1 bidder1 PaintingAuction $BIDDER1_BID_ID

#### 조직1에서 입찰자2로 입찰
- Bidder2는 그림을 500달러에 구매하고 싶습니다.
node bid.js org1 bidder2 PaintingAuction 500

- 애플리케이션에서 반환된 입찰 ID를 저장
export BIDDER2_BID_ID=915a908c8f2c368f4a3aedd73176656af81ddfab000b11629503403f3d97b185

- 입찰자2의 입찰가를 경매에 제출:
node submitBid.js org1 bidder2 PaintingAuction $BIDDER2_BID_ID

#### 조직2에서 입찰자3로 입찰
- Bidder3는 그림에 대해 700달러를 입찰
node bid.js org2 bidder3 PaintingAuction 700

- 애플리케이션에서 반환된 입찰 ID를 저장
export BIDDER3_BID_ID=5e4e637c68833b178739575f6fe09820b019551a8cfbb43a4d172e0aa864dfad

- 입찰자3의 입찰가를 경매에 추가:
node submitBid.js org2 bidder3 PaintingAuction $BIDDER3_BID_ID

#### 조직2에서 입찰자4로 입찰
- Org2의 입찰자4가 그림을 900달러에 구매하려고 합니다.
node bid.js org2 bidder4 PaintingAuction 900

- 애플리케이션에서 반환된 입찰 ID를 저장
export BIDDER4_BID_ID=49466271ae879bd009e75a60730a12bfa986e75f263202ab81ccd3deec544a35

- 입찰자 4의 입찰가를 경매에 추가:
node submitBid.js org2 bidder4 PaintingAuction $BIDDER4_BID_ID


#### 경매 종료
- 네 명의 입찰자가 모두 경매에 참여했으므로 판매자는 경매를 종료하고 구매자가 입찰가를 공개할 수 있도록 하려고 합니다
node closeAuction.js org1 seller PaintingAuction

#### 입찰가 공개
- 입찰가를 공개하기 위한 거래는 다음 네 가지 검사를 통과해야 합니다.
Step1) 경매가 마감되었습니다.
Step2) 입찰을 생성한 ID가 거래를 제출했습니다.
Step3) 공개된 입찰의 해시는 채널 원장의 입찰 해시와 일치합니다. 이는 입찰가가 비공개 데이터 수집에 저장된 입찰가와 동일함을 확인합니다.
Step4) 공개된 입찰가의 해시는 경매에 제출된 해시와 일치합니다. 이는 경매가 종료된 후 입찰가가 변경되지 않았음을 확인합니다.

- Org1의 Bidder1의 입찰가를 공개 (800)
node revealBid.js org1 bidder1 PaintingAuction $BIDDER1_BID_ID

- Org2의 Bidder3의 입찰가를 공개 (700)
node revealBid.js org2 bidder3 PaintingAuction $BIDDER3_BID_ID

## 경매가 지금 종료되면 Bidder1이 이깁니다. 판매자 ID를 사용하여 경매를 종료하고 어떻게 되는지 살펴보겠습니다.
node endAuction.js org1 seller PaintingAuction

- 경매를 종료하는 대신 거래로 인해 보증 정책이 실패합니다. 경매의 종료는 Org2의 승인을 받아야 합니다
- 트랜잭션을 승인하기 전에 Org2 피어는 아직 공개되지 않은 낙찰된 입찰에 대해 개인 데이터 컬렉션을 쿼리합니다.
- Bidder4가 낙찰 가격보다 높은 입찰가를 생성했기 때문에 Org2 피어는 경매를 종료할 거래를 승인하는 것을 거부합니다.
--> Submit the transaction to end the auction
2020-11-06T13:16:11.591Z - warn: [TransactionEventHandler]: strategyFail: commit failure for transaction "99feade5b7ec223839200867b57d18971c3e9f923efc95aaeec720727f927366": TransactionError: Commit of transaction 99feade5b7ec223839200867b57d18971c3e9f923efc95aaeec720727f927366 failed on peer peer0.org1.example.com:7051 with status ENDORSEMENT_POLICY_FAILURE
******** FAILED to submit bid: TransactionError: Commit of transaction 99feade5b7ec223839200867b57d18971c3e9f923efc95aaeec720727f927366 failed on peer peer0.org1.example.com:7051 with status ENDORSEMENT_POLICY_FAILURE


- Org2의 Bidder4의 입찰가를 공개
- 경매를 끝내기 전에 bidder4의 입찰가를 공개해야 합니다.
node revealBid.js org2 bidder4 PaintingAuction $BIDDER4_BID_ID

#### 경매 종료
- 낙찰된 입찰가가 공개되었으므로 경매를 종료할 수 있습니다.
node endAuction org1 seller PaintingAuction

#### Clean System
rm -rf wallet
cd ../../test-network/
./network.sh down











