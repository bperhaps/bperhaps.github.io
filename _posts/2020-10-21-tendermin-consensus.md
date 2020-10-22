### Tendermint

#### Entities
---
  - Nodes  
    - 서로 연결되어 p2p 네트워크를 이룸.  
    - *gossip* 을 통해 새로운 정보를 주고 받는다.
    - 각 노드는 정렬된 트랜잭션들의 블록체인의 형태로 보관.
    - User들이 서명된 트랜잭션을 특정 노드에게 제출할 때 노드는 새로운 트랜잭션의 중계자 역활을 한다.
    -
  - Users
    - 시스템의 Account를 가지고 있다.
    - User의 Account는 publc key 또는 address로 식별된다.
    - 각각의 Account는 transaction을 통해 변화시킬 수 있는  coin을 소유하고 있다.

#### Block Structure
---
검증된 트랜잭션은 블록 단위로 grouping 된다.

[](/assets/fig1-block-structure.jpg)

- Validation Hash : 해당 블록에 포함된 '검증자들의 서명'으로 이루어진 merkle tree root.
- Transaction Hash : 해당 블록에 포함된 transaction으로 이루언진 merkle tree root.
- StateHash : 블록의 트랜잭션들을 적용한뒤 persistent account state(블록체인 외부의)의 merkel hash root와 같다. (무슨말이지?.)
- Blockhash : header, validation, transactoins hash들 을 해쉬한 값. 이를 통해 block hash 자체가 블록과 account state의 구성요소들의 머클 루트 해쉬가 된다. 따라 구성 요소들을 머클 해쉬를 통한 검증이 가능해짐.

블록의 모든 트랜잭션이 유효하고 충분한 validatino을 포함한 서명이 있을때 블록이 유효하다고 말한다.

#### Validator
---
- *bond transaction* 을 통해  코인이 보증금(bond deposit)으로 잡혀있는 계정을 가진 User를 의미.
- 다음 블록에 합의하기 위해 암호학적 서명, 즉 표(votes )를 브로드캐스트 하여 합의 프로토콜에 잠여한다.
- Validator의 보증금의 양에 따라 그 양만큼의 *voting power* 를 가졌다고 한다.
- *unbonding transaction* 을 통해 보즘금을 돌려받을 수 있다. 트랜잭션을 개시하고 나면 *unbonding period* 라는 미리 정해진 기간동안 잠기고, 그 이후에 Validator는 해당 코인을 자유롭게 사용할 수 있다.

- 총 투표 파워(Total voting power)의 최소 2/3를 가진 validator들의 집합을 3/2 majority of validators (검증자의 2/3 다수) 라고 부른다. 마찬가지고 총 투표권력의 최소 1/3을 가진 점증자 집단을 검증자의 2/3 라고 부른다.

- 블록은 Validators 의 2/3 majority가 해당 블록을 위해 서명했을 떄 commit 된다.
- 만일 같은 높이의 서로 다른 두 블록이 2/3 majority of validators의 서명을 받으면 fork가 일어난다.
- 위와 같은 경우는 일반적인 계산으로 봤을 때 최소 1/3 majority of validators의 validators가 서명을 중복해서 하였을 때 발생.
- 서명이 중복 생성 되었을 때, 중복된 서명을 전송 받은 누구든 *evidence transaction* 을 생성할 수 있음.
- *evidence transaction* 은 블록에 commit 되며 잘못된 행동을 한 validator의 보증금을 파괴시킨다. 이를 통해 validator의 잘못된 행위를 방지할 수 있다.

### 합의

- 가정
  - Partially Synchronous.
  - 메세지 전송에 time bound가 있음.
  - 모든 정상 노드들은 다음 블록에 대한 합의가 이루어 질 때 까지 짧은 시간 동안 충분히 정확성을 유지할 수 있는 내부 시계에 접근이 가능하다.
  - 최대 1/3의 비잔틴 노드에 대한 투표를 허용한다.

  ### 알고리즘
  ---
  validators는 블록을 서명함으로 써 합의 프로세스에 참여한다.  
  3개의 투표 과정이 있음.
  - *prevote*
  - *precommit*
  - *commit*

  블록은 2/3 majorty of validators가 블록에 대한 커밋을 서명하고 브로드 캐스트 했을 때 네트워크에 커밋(*committed by the network*)됐다고 한다.

  - 블록체인의 각 높이에서, tendermint의 합의는 round-based 프로토콜을 통해 다음 블록을 결정한다.
  - 각 Round는 3개의 단계로 나뉘어짐
    - *Propose*
    - *prevote*
    - *Precommit*
  - 2개의 특별한 단계또한 존재함
    - *Commit*
    - *NewHeight*

    prepose, prevote, precommit 과정은 Round에 할당된 총 시간의 1/3을 차지한다. 각 round는 이전 라운드보다 약간의 고정된 시간만큼 길어진다. 이것은 부분 동기 네트워크(partially synchronous network)가 결국에는 합의에 도달할 수 있도록 함.

    [fig3. overview of state machine]()

    각 라운드는 validators 중에서 voting power가 가장 높은 validator를 *proposer* 로 정함.  
    결정론적 round-robin 알고리즘과 주어진 높이 *H* 의 블록체인은 모든 노드가 모든 라운드의 제안들에 대한 시퀀스에 동의하는것을 명확하게 한다.

    #### *Proposer*
    ----
    - 합의의 제일 첫 단계.
    - *gossip* 을 통해 피어들에게 proposal을 전송함.
    - proposer가 몇개의 이전 라운드의 블록에 잠긴 경우, *proof-of-lock* 을 proposal안에 추가하여 잠긴 블록을 제안함 (자세한 내용 후술)
    - Propose 단계동안 모든 노드는 proposal을 자신의 이웃 피어에게 전송한다.

    #### *Prevote*
    ---
    - *Prevote* 단계의 시작에서 각 validator는 결정을 한다. *-> 무슨 결정?*
    - validator가 어떤 이전 라운드의 제안된 블록에 잠겨있을 경우,  lock된 블록을 위한 prevote를 서명하고 브로드 캐스트 한다.
    - validator가 현제 라운드에 대해 정상적인 proposal을 받았다면, 제안된 블록에 대한 prevote를 서명하고 브로드캐스트 한다.
    - validator가 proposal을 받지 못했거나 잘못된 것을 받았다면, *nil* prevote를 서명한다.
    - Prevote 단계에서는 locking이 일어나지 않는다.
    - 모든 노드는 라운드에 대한 모든 *prevote* 들을 이웃 피어에게 전송함.

    ### *Precommit*
    ---
    - validator가 특정한 받아 들여질 수 있는 블록에 대해 2/3 이상의 *prevote* 를 받으면 validator는 블록에 대한 *precommit* 를 서명하고 브로드 캐스트 한다. 또한 블록을 lock하고 이전 lock을 풀어준다.
    - 노드는 한 시기에 최대 1개의 블록에 lock을 걸 수 있다.
    - 만일 노드가 2/3이상의 *nil* prevote를 받으면 단순히 unlock 한다.
    - locking or unlocking 할 때 노드는 잠금된 블록을 위한 prevote(또는 *nil* prevote )들을 모은다. 그런 뒤 나중에 propose할 차례가 됐을 때 모은 prevote들을 *proof-of-lock* 으로 패키지 한다.
    - 만약 노드가 블록(또는 *nil*)에 대해 2/3이상의 prevote를 받지 못했다면, 아무것도 서명이나 lock을 하지 않는다.
    - *Precommit* 단계동안 모든 노드는 precommit들을 모든 이웃 피어에게 전송한다.

    ### *Commit*
    - 특별한 단계, 2개의 분리된 상태가 존제하며 라운드롤 종료하기 전에 두 상태가 모두 만족되어야 한다.
      - 첫째 : 노드는 반드시 커밋된 블록을 수신받아야 한다.  
      검증자가 한번 블록을 수신하면 해당 블록에 대한 커밋을 서명하고 브로드 캐스트한다.
      - 둘째 :노드는 반드시 네트워크에 의해 precommitted된 블록에 대한 commmit을 최소 2/3개를 수신할때까지 기다려야 한다.

    두 상태가 만족되면 노드는 CommitTime을 현재 시간으로 설정하고 NewHeight 단계로 전환한다.

    CommitTime의 비동기 및 로컬 특성은 주어진 Height의 합의과정 동안 시계가 충분히 정확하다면 네트워크가 drifting clock임에도 불구하고 네트워크가 컨센서스를 유지할 수 있게 한다.

    ## *NewHeight*
    - 높이 *H* 에서 첫 라운드(round 0) 이전 단계이다.
    - 높이 *H-1* 에서 이전에 커밋된 블록에 대한 commit을 추가적으로 모으는 단계.
    - 노드는 *CommitTime* 이후에 일정 시간동안 이 단계를 유지함.
    - 최소 이 단계를 통해 최소 2/3개의 commit이 블록 proposal에 추가됨. 따라서 느린 validator의 commit이 블록체인에 포함될 수 있음.

    합의 과정을 진행하는 동안 노드가 특정한 블록에 대한 2/3개 이상의 commit을 받는다면, 노드는 즉시 Commit 단계로 진입한다. 그렇기 때문에 Commit 단계로 진입할 수 있는 방법은 두가지가 존재한다.

    라운드 *R* 에서의 블록을 위한 commit-vote는 *R<R'* 인 모든 라운드 *R'* 에 대한 prevote, precommit들로 계산된다. Commit-votes는 현재 라운드 또는 단계에 관계 없이 백그라운드에서 이웃 피어에게 전송된다.

    합의과정을 진행하는 어느때든, 만약 노드가 라운드 *R* 에서 lock 되어있지만 *R'* (*R < R'*) 의 *proof-of-lock* 을 전송받으면 노드는 unlock 된다.
