# Pre-Q-Learning
data/factory_order_test.csv : test 데이터  
data/factory_order_train.csv : train 데이터  
data/obstacles.csv : 장애물 위치 좌표  
data/Q_finder.csv : 수렴시킬 테이블 아이템 목록 (agent 이름)  
data/box_PQ.csv : 목적지 좌표  
result_gif : PQ 결과물 디렉토리  
Parallel_Q_Learning.ipynb : PQ 알고리즘의 실행 노트북 파일

# Introduction
![image](https://user-images.githubusercontent.com/96896665/172670859-5486a631-8f08-4a8f-a550-16869bb6adf7.png)

아버지를 아버지라 부르지 못하고... Pre-Q러닝이 되어버린 슬픈 아이입니다.  
원래의 이름은 Parallel Q러닝  
~~흔적이 여기저기 남아있어요~~

저어는 컴퓨터 공학에 대해 1도 모르는 상태로 AI부터 배운사람이라  
쓰레딩 이런 개념으로 병렬적이란 단어를 사용한 것이 아닙니다.

단지 Q러닝의 off-policy한 특성 : 과거의 경험을 현재 최적 정책을 찾는데 사용 가능이라는 것을 생각해

![image](https://user-images.githubusercontent.com/96896665/172681111-ff50e7ae-37f1-4e98-a40f-ca874e86dc3a.png)

나루토 환영분신술 마냥
여러 위치에서 각자 다른 임무 'A Q-table 찾아와' 'B Q-table 찾아와' 해서 
Q_tables로 수렴하는 상황을 병렬적으로 수행한다는 의미로 붙인 것입니다.

저는 A-T에 이르는 과정을 한번에 코드에 넣고 돌리긴 했지만
사실 A수렴끝나고 B다시 시작하는거나 이론상 같은거니까.
여러모로 오해의 소지도 있고, 
귀신같이 아이디어를 내주신 휘정님 덕에
바꾼 Pre라는 이름이 맘에 안드는 것도 아니지만...

뭔가 명명권은 연구자의 몇 안되는 자존심? 
그런 느낌이라 아쉬움이 있긴합니다ㅋㅋㅋ

Q러닝으로 첫 도전을 해본건  
아무래도 배운지 3주밖에 안된 왕초보라  
제일 쉬운거부터 해보자는 생각이었던거 같아요.

아무튼 제 첫 딸내미? 같은 존재입니다. 많관부많관부😇

## 알고리즘 플로우차트
![image](https://user-images.githubusercontent.com/96896665/172262570-0bf6f52b-c485-42e0-82e6-af2c1ddfd438.png)


### Exploration Loop
offpolicy인 만큼 Exploration과 Exploitation을 완전히 분리했습니다.

여러 에이전트(18개의 타겟)가 하나의 타겟을 목표로 탐색을 시작합니다.  
처음에는 ε-greedy를 통한 탐색을 해도 epoch를 늘리면 언젠가는 수렴할꺼라 예상했으나.  
'시작위치로 돌아와야한다'는 점에서, 고민이 많았습니다.  
시작위치에서 가장 멀리있는 점까지 ε-greedy을 사용하면  
대충 4^12정도 해당하는 확률로 구석자리에 도착하니  
직관적으로 이건 불가능하다는 생각이 들었고

짱구를 굴리다보니, 시작위치를 완전 랜덤하게 하면 되지 않을까라는 결론에 도달!  
(cheating? 아닌가하는 생각도 들긴했습니다.)

그럼 장애물에서 시작하면 어떻게하나?는 생각도 들긴했는데  
장애물 내부의 Q-Table은 어차피 사용되지 않으므로,  
마스킹 등의 방식을 생각안한것은 아니지만 심플하게 구현했습니다.

Q_tables를 저장할때 {A:A_Q_table} 이런식으로 저장해보고 싶었는데...
코드에 대한 이해도가 부족해서 아쉽지만 그냥 (18, 9, 10, 4) ndarray로 사용했습니다.

이후에 흥선님이 시작위치를 '아예 모든 위치에서 시작하도록 고정하면 되잖아'  
라는 말씀도 인상깊었습니다.

뭐 그래도 랜덤이 느낌 있잖아요?😎  
통계... 느낌 아니까

+ 초기모델과 달리 Action Masking과 Move Block(아이템 정면에서만 접근 가능하도록 함)  
이 구현되어 있습니다.  
가급적 env에 손 안대고 싶었지만, 다른 분들 모델과 비교 위해 맞춰서 수정했습니다.

### Q테이블 수렴 확인
![image](https://user-images.githubusercontent.com/96896665/172262697-583f56b3-35c2-493a-8ab6-a74662adc76a.png)

1500 에포크면 100%성공률을 가지는 것을 확인했습니다.  
물론 A~Q + T의 18개의 에이전트가 동시에 돌아다니므로  
실질 루프는 12000번이라고 볼 수 있습니다.

파이토치에 대한 이해도가 부족해 GPU측정은 끝까지 못해서 아쉬웠지만,  
CPU만 사용해도 걸리는 시간은 대충 7분내외.

DQN에 비하면 압도적인 학습속도를 자랑한다고 볼 수 있습니다.  
물론, 적은 epoch면 최적 경로를 잘 찾지는 못하므로  
10000번 정도는 돌려줬습니다.

일단.. 비교대상이 성공률이 아니라 속도인 것만으로도  
나름대로 경쟁력이 있다는 생각이 있습니다.


### 정책테이블 수렴 시각화
![image](https://user-images.githubusercontent.com/96896665/172262650-a63a91c2-2ba0-401d-9ee7-3ead178cbe07.png)

흥선님이 짜주신 코드를 기반으로  
정책테이블을 시각화 해보았습니다.  
20000번 학습에도 몇몇 군데 최적정책이 아닌 것을  
확인 할 수 있습니다.

장애물 주변에 죽는 것 때문에 더 많은 학습이 필요했던 걸까요?

### Exploitation Loop
탐색이 끝나면, 이제 만들어진 Q-Tables을 지도처럼 펼친 후   
목표에 따라 바꿔 들면서 argmax로 이동하면 됩니다.  
ε도 0으로 맞추고, 업데이트 코드도 주석처리해버렸습니다.

아마 우리집 강아지도 할 수 있을꺼 같아요(거짓말)


### 테스트 데이터
![image](https://user-images.githubusercontent.com/96896665/172262733-bf32148a-1233-4014-b3ae-f95fb02c12a4.png)

모든 테스트 데이터를 성공적으로 수행했습니다 🎉  
특이점은 거의 즉시시전이었습니다. CPU로 30초 내로 끝남.

나중에야 깨닫게 된건데, DQN도 학습시간이 오래걸리는 거지
어차피 θ가 수렴해있다면 테스트는 똑같이 빠를 것으로 예상할 수 있더라구요.

## 성공 Gif
![result627_성공](https://user-images.githubusercontent.com/96896665/172262471-c938bac5-c7d4-41a6-ac45-b82c7346cba7.gif)

성공 전체 파일읏 results 폴더에서 확인하실 수 있습니다.

## PQ러닝의 한계

### 1) 항상 Random한 위치에서 탐색을 시작할 수 없다.
말도 많고... 탈도 많았던 내용...  
너무나 당연하게도 시작할 수 없을꺼라고 생각했슴다.  
내심 이렇게 좋은 방법이 있는데 안쓰는 것은 뭔가 이유가 있을꺼라  
막연하게도 생각했었는지도 몰라요.

팀원들과의 많은 토론(?) 끝에 삭제했고...  
다들 할 수 있다는 말에  
제대로 된 반박도 못하고  
MDP가 deterministic이니, 장애물이 많은 특수한 상황이니 반례를 들어보려다  
포기해버렸지만

다시 생각을 정리해보니 이 말은 틀리지 않았던 것 같습니다.

강화학습 Acolyte로서  
우리는 너무나 당연한 것을 등한시 했던 것이었습니다.  
**'env는 우리 마음대로 바꿀 수 없다'**

현실을 simulate하는 강화학습 상황에서  
물리적으로 불가능한 상황은 극복할 수 없을 때가 대부분이고,

단지 우리는 '가상환경의 코드'라 주어진 상황,   
즉, env를 맘대로 수정할 수 있는 엄청난 특권을 마음대로 휘둘러 왔기 때문에  
당연히 random한 위치에서 시작할 수 있다고 착각해버린 것이 아닐까 합니다.

심지어 같은 가상환경인 게임상황 같은 경우에도, 코드를 수정할 '권한'은 주어지지 않을때도 있으니..

![image](https://user-images.githubusercontent.com/96896665/172691190-dd884424-4107-4e08-ae31-1d072b47163c.png)  
~~필리시 3돌이면 랜덤위치에서 시작할 수 있대요 글쎄~~

흔히말하는 in vitro환경에서 세팅을 잘 한다면 가능할지 모르지만  
무조건 적용할 수 있다는건 당연히 아닌게 맞는 것 같아요.

다만 이 가능성을 찾았다는 건 의의가 있는 것 같으니  
아직 잘 모르니까 할 수 있었던 틀깨기라고 긍정적으로 생각해 봅시다.

!근데 현실을 모방에서 환경을 재현하는 상황을 자유롭게 구사할 수 있다면?

### 2) table기반 vs 네트워크 기반
ε-greedy만 쓸 수 있다면, 단순 탐색도 엄청나게 어려웠단 사실을 기억한다면,  
이 경우 조금이라도 예측 능력이 있는 네트워크가 유리할 것이고

random이라는 꼼수를 쓴다고 쳐도, 타겟의 수 * 그리드의 크기 * 액션의 수 * 다양한 환경변수와 같은 이유로  
테이블이 계속 늘어날테니 노트에 적은대로,  
네트워크 연산량이 더 적어지는 시점은 분명 오겠죠?

Q러닝부터 시작해서 액터크리틱까지 하기에는 시간이 부족했나봅니다.
이건 나중에 꼭 스터디로 도전 해보는 것으로 하겠습니다!

### 3) 가치기반의 강화학습의 한계
정책이 가변적이면, 아마도 여기서는 없었던 MDP가 확률적인 상황이나 각종 외부 변수에도  
유연하게 대응할 수 있을 것 같다는 생각이 듭니다.
정책이 고정되는건 심플해서 구현하기 좋지만, 정책기반 모델도 꼭 해보고 싶네요

그래서 off-policy 설명 좀 잘 해주실분?

### 4) 환경변화에 민감
많은 강화학습 모델들이 그렇겠지만 특히 PQ는 Q테이블을 미리 저장해놓는 방식이라  
탐색 과정이 완전 분리되어있기 때문에
단순한 변화에도 처음부터 새롭게 테이블을 만들어야 하는 단점이 있다.

주기적이거나 예측할 수 있는 환경변화라면  
그에 따른 Q테이블 또한 새롭게 만들 순 있겠지만...  
메모리 vs 병렬 처리 승자는 누구?

## 회고
고민도 많이 했고, 이리저리 휘둘리기도 많이 했지만  
역시 처음으로 만들어서 성공해본 모델이라 정이 많이 간다.

물론 좋은 팀원들이 있어 가능했겠지!

몇 주동안 푹빠져서 강화학습과 동고동락해보니,  
나랑 정말 잘 맞는 것 같다.  

계속 공부해서 다둥이 아빠가 되어야겠다
