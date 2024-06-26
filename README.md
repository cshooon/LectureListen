# LectureListen
2024 인공지능 음성인식 부트캠프

## Goals
현재 음성인식기의 문제점 파악을 통한 개선점 도출
음성인식 성능을 향상(ERR, CER값의 감소)시켜 한국어 대학 강의에 대한 STT 성능 개선

## Base Model
기존 음성인식기를 살펴보겠습니다. 베이스라인 레시피는 aihub의 케이스푼스피치를 바탕으로, 음절을 기본 단위로 수정해서 사용하고, conformer을 이용하고 있었습니다. 
데이터는 AIHUB에서 제공하는 [한국어 대학 강의 데이터셋](https://aihub.or.kr/aihubdata/data/view.do?currMenu=&topMenu=&aihubDataSe=data&dataSetSn=71627)을 사용했습니다. 데이터 전처리 과정을 거쳐 최종적으로 150만 줄의 데이터를 학습에 사용하였으며, 한 번 학습하는 데에 약 이틀이 소요되었습니다. 학습은 [espnet toolkit](https://github.com/espnet/espnet)을 활용했습니다.


베이스라인 분석 이후 음성인식 성능을 높이기 위한 다양한 실험을 진행하였는데요. Stage1 부터 시작해 옵션을 바꾸어가며 실험했습니다. Stage 1에서는 데이터 준비에 초점을 맞췄습니다. 데이터셋 크기를 조정하고 이중 전사 표기를 선택했습니다. Stage 2에서는 속도 변화를 적용해 보았습니다. Stage 5에서는 모델의 핵심이 되는 토큰 목록을 생성, 변경해 보았습니다. 이후 단계에서는 언어 모델을 적용하고, ASR 모델을 학습시켰습니다. 마지막으로, Stage 12와 13에서는 실제 음성 데이터로 모델을 디코딩하고, 그 성능을 평가하는 스코어링 과정을 거쳤습니다.

![image](https://github.com/cshooon/LectureListen/assets/113033780/c298dcef-8f13-46ee-8e45-4f06806eda5a)

## Experiments
<img width="1469" alt="Screenshot 2024-04-21 at 5 41 51 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/41122735-c61b-4522-b030-e81310540c1c">

우선 stage1에서 데이터셋 크기를 조정하였습니다. 그 이유는 데이터셋을 한 번 학습시키는데 이틀 이상의 시간이 소요되어 제한된 시간 안에 다양한 실험을 할 수 없다고 판단했기 때문입니다. 
따라서 데이터셋 크기를 조정하여 많은 실험을 진행해보고, 결과가 우수했던 실험만 150만줄 데이터에 적용하기로 했습니다. Head 명령어를 사용해서 7만줄, 20만줄, 30만줄 학습을 진행했고, 성능과 학습시간의 trade-off를 판단했습니다. 

빨간 네모 칸 안의 err 값과 시간을 중심으로 봐주세요. 데이터셋의 크기가 작을수록 높은 err값을 가지고 있었습니다. 30만줄에서는 학습하는 데에 8시간 걸렸으나 20만줄에서는 6시간이 걸렸습니다. 이를 통해 20만줄 데이터셋이 시간대비 err이 낮다는 것을 알 수 있었습니다. 따라서 20만줄 데이터셋에서 실험을 진행하기로 결정했습니다. 

<img width="614" alt="image" src="https://github.com/cshooon/LectureListen/assets/113033780/e3de76dc-5c6b-4225-a741-53c10c743d65">

다음으로 stage1에서 이중전사 표기를 선택하였습니다. 한국어 대학 강의 데이터셋은 음성 파일마다 해당하는 텍스트가 외국어문자와 한글 두 가지 형태로 전사되어 있습니다. 이는 이중 전사 방식으로 오류값을 높입니다. 음성인식 성능을 높이기 위해 외국어 문자와 한글 중에 어떤 것을 선택해야 할지 실험을 진행했습니다. 

<img width="1470" alt="Screenshot 2024-04-21 at 5 43 25 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/d3584c27-2712-4e0d-a3ce-22e4178f0001">

첫째 줄의 20만줄이 전사처리를 하지 않은 데이터셋입니다. 외국어 문자를 선택했을 때는 err 값이 28.1이 나왔는 것에 비해, 한글을 선택했을 때 27.2가 나왔습니다. 이는 전사처리를 하지 않았을 때보다 0.5정도 향상된 값입니다. 

<img width="1475" alt="Screenshot 2024-04-21 at 5 45 22 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/c1da43d7-c322-46cc-90e1-82b9a15da6a2">

다음 stage5 토큰 리스트 변경을 살펴보겠습니다. 토크나이저는 텍스트를 작은 단위인 토큰으로 분리하는 도구입니다. 토큰 목록은 특정 언어나 데이터셋에 대한 사전을 형성하며 텍스트 데이터를 숫자 형태로 변환하는 데에 사용됩니다. 오른쪽 같은 찰 토크나이저의 토큰 목록에 포함된 토큰들은 모델이 학습 과정에서 인식하고 처리할 수 있는 모든 글자들을 나타냅니다.

<img width="383" alt="image" src="https://github.com/cshooon/LectureListen/assets/113033780/2913b39b-59fa-4300-80ec-9052a9e18ae5">
<img width="417" alt="image" src="https://github.com/cshooon/LectureListen/assets/113033780/df0eedc4-30d5-4af3-b746-8617caf80260">

따라서 토큰 타입을 변경하여 char 보다 더 많은 토큰을 가지고 있는 Hugging Face 중 코발트와 코찰일렉트라 tokens.txt 파일을 사용했습니다. 
위 표를 보면 두번째 줄 코발트베이스 토크나이저와 세번째즐 코찰일렉트라 토크나이저를 적용했을 때 높은 성능 향상을 보입니다. 이러한 결과의 원인으로, 추가된 토크나이저의 토큰 목록에는 존재하지만, 기존 토큰 목록에는 없는 문자들이 포함되었기 때문이라는 가설을 세웠습니다. 

하지만 기본 토큰 목록에는 없는 새로운 토크나이저의 토큰 목록을 출력해보니 사용하지 않는 희귀 글자만 존재했습니다. 이는 글자의 유무가 큰 영향을 미치지 않았다는 뜻입니다. 
성능이 좋아진 이유는 데이터셋 크기에 달려있었습니다. 20만줄 크기의 데이터로 실험할 때에는 드문 음절을 추론하고 인식하는 데 토크나이저가 큰 도움이 되었습니다. 이는 왼쪽 그래프보다 오른쪽 그래프에서 loss값이 훨씬 줄어드는 것을 보면 알 수 있죠. 

<img width="1398" alt="Screenshot 2024-04-21 at 5 46 30 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/8fb196b3-1aa4-4e5b-82e3-f3841ea6ebdf">

그러나 150만줄 크기의 큰 데이터셋에서는 별 다른 성능 개선이 이루어지지 않았습니다. 이미 다양한 음절들을 포함하고 있기 때문입니다. 왼쪽과 오른쪽 그래프의 차이가 뚜렷하지 않는 것으로 해당 결과를 유추할 수 있습니다. 따라서 저희는 이미 충분한 데이터를 가지고 있어서 토크나이저 목록 추가 생성이 음성인식 성능에 큰 도움이 되지 않는다고 결론을 내렸습니다. 이후, 토큰타입을 char로 다시 변경했습니다. 

<img width="1433" alt="Screenshot 2024-04-21 at 5 48 39 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/c75fd1c7-c395-4b2b-85b2-1aaa382ed2e6">

다음 스테이지 6~8 lm을 보겠습니다. 처음 엘엠을 적용했을 때는 CER이 증가하는 현상이 나타났습니다. 이런 결과가 나온 이유는, 지금까지 사용한 150만 데이터셋에서 20만을 랜덤추출하여 데이터의 분포가 균일하지 않았습니다. 특히 평가 데이터에 있는 공학용 데이터가 실험 데이터에 부족했기 떄문에 성능 저하처럼 보인 것으로 판단되었습니다. 이후 모든 실험에서 공학용 데이터셋 30만줄을 사용하여 실험을 진행했습니다. 

<img width="600" alt="Screenshot 2024-04-21 at 6 08 21 PM" src="https://github.com/Harmony2024/LectureListen/assets/113033780/2306a0b2-9368-4fbb-97c9-8c13c0f70e59">

공학용 데이터셋에서 실험한 결과 train loss 값은 낮고 valid loss 값은 높은 전형적인 오버피팅 현상이 나타났습니다. 이를 해결하기 위해 러닝레이트와 드롭아웃을 조절하였습니다. 150만에서 드롭아웃을 0.3으로 설정했을 때 피피엘이 가장 낮게 나와, 이 값을 사용하기로 했습니다. 

<img width="1460" alt="Screenshot 2024-04-21 at 6 06 12 PM" src="https://github.com/Harmony2024/LectureListen/assets/113033780/22aeb9fe-9a1f-4d43-8c2c-2417935235f1">

다음 stage10-11에서 librispeech, branchformer asr 모델을 학습시켰습니다. 실험할 모델을 정할 때에는 남은 시간이 많지 않아 짧은 시간 안에 성능 향상을 할 수 있는 모델들로 찾아 선정했습니다. 가장 첫줄이 베이스라인에서 사용하던 모델입니다. 실험에서 err은 브랜치포머가 가장 낮게 나왔고, s.err은 케이스푼스피치가 가장 낮게 나왔습니다. 둘 중 우열을 가리기 어려웠기 떄문에 epoch 10 이후 valid loss를 비교해서 최종선택하기로 했습니다.

<img width="1485" alt="Screenshot 2024-04-21 at 6 09 08 PM" src="https://github.com/Harmony2024/LectureListen/assets/113033780/34b5eb5c-9676-49d8-a10a-354bbdccd44b">

Stage 12, 13에서는 지금까지 나온 유의미한 실험 결과를 실제 모델에 적용시켜 디코딩 했습니다. 베이스라인의 err 값은 12.4였던 반면, 실험 결과는 9.3으로 총 25% 성능이 향상되었습니다. 하지만 그래프에서 알 수 있듯이 ctc 값이 1.0일 때 성능이 가장 좋았습니다. 이는 언어 모델을 적용했을 때 성능이 소폭 하락되었음을 의미합니다.

## Service

<img width="1451" alt="Screenshot 2024-04-21 at 5 58 06 PM" src="https://github.com/cshooon/LectureListen/assets/113033780/7990f399-c0d8-4e3b-90b4-843271b9ed33">

다음은 음성인식 서비스-음성 챗지피티에 대해 설명드리겠습니다. 음성인식을 통해 음성 파일을 텍스트로 바꾼 다음 해당 내용을 챗지피티에 질문하는 기술입니다. 지피티 api를 사용하여 음성인식으로 챗지피티를 사용할 수 있다는 게 특징입니다. 해당 기술을 안드로이드 등 모바일에 연결하면 모바일 기능을 이용해 음성 녹음하고, 이를 챗지피티에 연결해 보다 편리하게 음성인식 챗지피티를 이용할 수 있습니다. 또한 음성파일을 듣고 직접 입력해야 하는 번거로움을 줄일 수 있습니다. 학교에서 할당받은 서버에 서비스를 구현해보았는데요. 서비스 데모영상은 다음과 같습니다. 전체 영상은 data 폴더를 확인해주세요!

![ezgif-3-aa797120f4](https://github.com/cshooon/LectureListen/assets/113033780/88dc14a1-e47c-4591-bb5f-5cf2cc8bdef2)


## Conclusion

마지막으로 음성인식 성능향상 최종 결론을 말씀드리겠습니다. 최종 err은 기존 베이스라인보다 25퍼센트 개선되었습니다. 여기에는 이중 전사에서 한글을 선택하고, 토큰 목록을 변경하고, ctc ,lm 웨이트를 조정한 것이 반영되었습니다. 또한 토큰 수가 많은 토큰 목록을 사용하는 것은 데이터가 부족할 때 유용하며, 속도 증강은 많은 시간이 필요하다는 점을 실험을 통해 알아내었습니다. 
언어모델 성능이 하락된 것이 아쉽고, 왜 하락되었는지 탐구하고 싶었으나 시간 부족으로 알아내지 못했습니다. 또한 인공지능 학습이 제한된 시간 속에서 선택의 연속임을 알게되었습니다. 많은 실험을 하면 성능이 향상될 것은 분명하지만 한계가 존재해 어떤 학습을 할지 끝없이 고민하고 선택해야 함을 깨달았습니다.

## Members

### Role
| ![최승훈](https://github.com/cshooon.png) | ![강연주](https://github.com/sally7788.png)   | ![진민찬](https://github.com/JinMinChan.png) | ![박율](https://github.com/Park-Yul.png)  |
| :--------------------------------------------------------------: | :--------------------------------------------------------------: | :--------------------------------------------------------------: | :--------------------------------------------------------------: |
|            [최승훈](https://github.com/cshooon)             |            [강연주](https://github.com/sally7788)             |            [진민찬](https://github.com/JinMinChan)             |            [박율](https://github.com/Park-Yul)             |
|                   모델 학습, 음성인식 서비스                           |                            발표, 모델 학습                             |                            데이터 전처리, 음성인식 서비스                             |                            ppt 제작, 모델 학습                            |

### 소감

최승훈: 인공지능 학습을 colab이 아닌 서버에서 할 수 있다는 점이 좋았다. A6000 4대를 활용할 수 있었고 다양한 실험을 할 수 있었다. 처음에 batch_bin을 작게 실험해서 학습을 더 많이 돌리지 못해 아쉽다. 그렇지만 이중 전사, tokenizer 등 여러 실험을 해보았고 최종 학습에서 소폭 성능 향상을 보였다. 겨울 방학 기간에 뜻깊은 시간을 보낸 것 같고 함께해준 팀원에게 고마웠다는 말을 전하고 싶다.

강연주: 실제 서버에 연결하여 학습을 진행할 수 있어서 실무와 비슷한 경험을 쌓을 수 있었다. 실제 데이터를 활용해 학습 방향을 찾고 평가하는 것은 처음이어서 부족한 점이 많았지만, 학습 결과를 토대로 좋은 성능을 낼 수 있어서 만족감이 컸다. 이번 부트캠프에서 가장 중요한 것은 학습 결과를 판단하고 추후 학습 방향을 정하는 것이라고 생각된다. 따라서 결과를 분석하는 힘을 길러야겠다고 다짐했다. 팀원들과 함께 했어서 뜻깊은 결과를 낼 수 있었다. 

진민찬: 인공지능 학습이라는 큰 흐름과 이를 서비스화 하는 단계를 경험해 볼 수 있던 점이 가장 인상깊었다. 평소 인공지능에 관심이 많았지만 이론적인 내용만 알고 있어 크게 와닿지는 않았었다. 하지만 실제로 모델의 성능을 향항시키기 위해 데이터의 전처리는 어떻게 해야할지, 어떤 모델과 파라미터값을 정해야 할지, 또한 비록 성능은 향상되지만 시간이 너무 오래걸린다면 이를 최적화 할 수 있는 방안이나 시간대비 성능 향상이 적절한지 등을 고민해보며 성장할 수 있는 계기가 되었다.

박율: 인공지능 부트캠프를 통해 인공지능 모델을 최적화하는 과정을 경험해본 것이 가장 의미있었던 것 같다. 인공지능에 대해 아무것도 알고있지 못했지만 팀원들과 함께 연구를 진행하면서 인공지능에 대한 전체적인 틀을 알아가고, 알게 된 내용을 기반으로 리눅스 환경에서 데이터셋 전처리와 인공지능 레시피 모델링, 학습 진행과 결과 분석 등 인공지능을 활용하는 일련의 과정을 경험해볼 수 있었다. 비록 팀에 큰 도움은 되지 못했지만 이번 부트캠프를 기점으로, 관심있었던 인공지능 반도체 관련 분야의 연구에 한 발자국 다가갈 수 있게 될 것 같다.
