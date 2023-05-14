## Learning Affinity From Attention End-to-End Weakly-Supervised Semantic Segmentation With Transformers (cvpr 2022)
- - - 
> **논문 제목만 읽고 유추한 내용**..<br>
Transformer를 이용하여 end-to-end waekly supervised semantic segmentation을 수행함.<br>
이때, 입력 받은 패치 간의 관계성을 나타내는 self-attention과 패치 간의 유사도를 나타내는 semantic affinity 간에 inherent consistency가 존재한다는 가정 하에 self-attention을 활용하여 affinity를 학습하는 방법론을 제안하겠다<br> 
(In segmentation, affinity: 주어진 커널 내 픽셀 위치 간 semantic 유사도를 나타내는 행렬)

### Overview
- Background
  - Multi-stage Methods
    - 대개 multi-stage framework로 구성됨(1. train classification model 2. generate CAM 3. leverage pseudo labels to train semantic segmentation)
    - training streamline이 복잡하며 efficiency가 좋지 못함
  - End-to-End Methods
    - convolution neura network 기반이어서 global features를 잘 포착 못함
  - Transformer in Vision
    - global feature relations을 잘 모델링함
    - MHSA는 semantic-level affinity를 잘 포착하여 coarse pseudo label의 quality가 증가하나 여전히 inaccurate한 문제점 존재
  <br><br>
- 이에, 본 논문은 **WSSS를 수행하는 Transformer 기반 end-to-end framework**을 제안함 
  - Global information을 고려하며 End-to-end WSSS를 더 잘 수행할 수 있도록 **Transformer**를 사용한 방법을 제안
    - **Affinity From Attention(AFA) module**<br>: MHSA에서 학습한 affinity label을 supervise하기 위해 신뢰할 수 있는 pseudo label affinity를 도출. 이에 random walk propagation을 수행해 initial pseudo label을 refine(diffuse object regions & dampen the falsely activated regions)
    - **Pixel-Adaptive Refinement module**<br>: local pixels의 RGB와 position information를 내포하는 Pixel-adaptive convolution으로 pseudo label의 local consistency를 보장


### Method
1. **Transformer Backbone**<br>: Mix Transformer proposed in Segformer / segmentation decoder: MLP decoder head <br> (Backbone parmaeters are initizlied with ImageNet-1k pretrained weights, while other parameters are randomly initialized.) 
2. **CAM Generation**<br>: initial pseudo labels로 사용 (global max pooling으로 뽑는 것이 가장 좋았음)
3. **Affinity from Attention**<br> 
   - semantic affinity는 multi-head attention을 linear combining하여 생성 
   - 그러나 self-attnetion mechanism은 directed graphical model임을 고려해, affinity는 symmetric 구조를 갖게 함 (since nodes sharing the same semantics)
   - **Pseudo Affinity Label Generation**
     - Affinity를 supervision하려면 reliable한 pseudo affinity label이 필요<br> → refined psuedo labels(Y_p)로부터 Y_aff를 생성
     1. CAM(M)을 특정 threshold beta_l, beta_h에 따라 reliable foreground, background, uncertain regions로 구분하여 Y_p를 생성
     2. Y_p 내 픽셀 (i, j)와 (k, l)이 동일한 semantic을 가지면 affinity를 positive로, 그렇지 않으면 negative로 지정. 둘 중 하나가 uncertain region일 경우 ignore함<br> 이때,한 쌍의 픽셀이 동일한 local window에 있는 상황만을 고려(멀리 있는 픽셀 쌍은 고려 안함)
4. **Pixel-Adaptive Refinement**<br>
   - local consistency를 위해 기존에는 dense CRF를 사용하였으나, training efficiency 측면에서 적합하지 않음.
   - local RGB와 spatial infomration을 통합해 low-level pairwise affinity를 생성하고 CAM에 mlutiple iterions 적용하여 pseudo label을 refine<br>(RGB information을 통해 색상 정보를 추출하고, spatial information을 통해 물체의 경계나 구조 정보를 추출한 다음, integrate하면 local semantic 정보를 더 고려할 수 있어서?)

### Experiment
- Dataset: PASCAL VOC 2012, MS COCO 2014 
- image-level과 saliency map을 이용하거나 image-level만 이용한 Multi-stage weakly-supervised models보다는 성능이 좋지 않으나 End-to-End weakly-supervised model 중에서는 가장 우수한 성능을 보임
- end-to-end는 single stage이어서 복잡한 training streamline으로 구성된 multi-stage보다 정확도가 낮겠지만 efficiency 측면에서는 더 우수함 