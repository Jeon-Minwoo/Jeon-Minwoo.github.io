---
layout: post
title: Attention & Transformer
date: 2022-07-28 19:20:23 +0900
category: Transformer
---
# Attention & Transformer

### Contents

1. NLP에서 attention 등장 배경
2. NLP(seq2seq)에서 Attention
3. Transformer (Attention is all you need)

## NLP에서 attention 등장 배경

**seq2seq(Sequence to Sequence) structure 문제점**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled.png)

- Encoder에서 각 단어의 embedding과 RNN의 hidden state를 거쳐서 정보가 압축이 되고, Encoder의 마지막 부분의 출력이 context vector가 된다.
- Decoder에서는 context vector와 문장의 처음을 표시하는 SOS(Start of Sequence) 를 입력으로 받는다.
- 각 단어의 의미를 하나의 벡터인 Context Vector안에 함축시키는 데 있어 정보 손실이 발생한다.
→ Gradient Vanishing 발생
→ Input Data가 Sequential하게 들어간다.

**Improved seq2seq**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%201.png)

- 여러 개의 단어 embedding을 context vector에 함축할 필요가 없다.
- 이전에 입력된 데이터에 대해서 출력의 영향이 낮아지는 문제가 완화된다.
- gradient vanishing이 완화된다.
- Encoder의 hidden state를 모두 넘겨주면 입력의 차원이 커지는 문제가 발생한다.

## NLP(seq2seq)에서 Attention

**Attention Mechanism**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%202.png)

Attention의 3 요소는 다음과 같습니다.

- Query: 질의, 찾고자 하는 대상.
- Key: 키, 저장된 데이터를 찾고자 할 때 참조하는 값
- Value: 값, 저장되는 데이터
- Attention에서는 Query에 대해서 어떤 Key와 유사한지 비교를 하고, 유사도를 반영하여 Key에 대응하는 Value를 합성(Aggregation)한 것이 Attention Value가 된다.

**Applying attention to seq2seq**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%203.png)

- Query는 Decoder의 hidden state가 된다.
- Key, Value는 Encoder의 hidden state가 된다.
- 먼저 Decoder의 hidden state는 RNN(LSTM)에서 받아서 연산하여 $s_i$ → $s_{i+1}$로 만듭니다.
- 그 후 Attenen value인 $a_i$와 $s_{i+1}$을 concat ($v_i=[s_i;a_{i-1}]$) 하여 을 만듭니다. 이 값을 FC layer와 Softmax를 거쳐서 최종 출력인 $y_i$를 출력합니다.
- 위의 구조는 아래와 같이도 표현할 수 있다.
    
    ![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%204.png)
    

## Transformer (Attention is all you need)

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%205.png)

**Positional Encoding**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%206.png)

- RNN에서는 모델 자체적으로 input의 순서를 고려한다.
- Recurrent connection이 제거된 Self-attention에서는 모델 자체가 위치 정보를 이용할 수 없다.
- 입력의 순서를 바꿔도 attention 결과가 바뀌지 않는다. (위의 그림 참고)
- Transformer에서는 위치정보를 제공할 수 있는 positional encoding 이용

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%207.png)

- Transformer는 RNN과 다르게 병렬적으로 계산할 수 있기 때문에 현재 계산하고 있는 단어가 어느 위치에 있는 단어인지 표현해야 한다. → Positional Encoding

**Dot-Product of Positional Encoding**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%208.png)

- Positional Encoding 값을 서로 Dot-Product 연산을 거치면 w와 t로만 이루어진 간단한 식을 도출할 수 있다.

**Self-Attention**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%209.png)

- 일반적인 Attention based model에서의 비교대상이 target sequence인데 반해 self-attention에서는 input간의 관계를 비교한다.

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2010.png)

- RNN의 한계점을 제거하기 위해 recurrent connection을 제거한다.
- 입력 Sequence의 vector간의 유사도를 계산한다.

**Scaled Dot-product Attention**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2011.png)

- Layer normalization을 수행하고 후 vector는 𝑁(0,1)의 분포
- Dot product의 값은 input의 값에 따라 매우 커지거나 작아질 수 있다.
- 이를 방지하기위해 softmax를 취하기전 일정한 scalar값으로 attention score를 나눈다.
    
    (논문에서는 queries와 keys의 dimension 𝑑𝑘의 제곱근을 이 값으로 사용)
    

**Masked Attention**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2012.png)

[Problem of Self-Attention in Decoder]

- Language modeling에서 Inference하는 과정의 미래의 출력은 알 수 없다.
- 일반적인 self-attention 계산방식으로는 self-attention score를 계산 하는 과정에서 미래의 출력들까지 고려하여 attention score를 계산한다.
- n번째 입력에 대한 출력은 n-1까지의 입력만을 고려해야한다.

→ decoder에서 self attention은 각각의 출력단어가 다른 모든 단어를 전부 참고하지 않고, 앞쪽에 등장한 단어만 참고하도록 Masked attention을 취한다.

**Multi-Head Attention**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2013.png)

- Scaled Dot-Product Attention을 h개 모아서 Attention Layer를 병렬적으로 사용한다.

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2014.png)

- 차원을 축소해서 보되 병렬적으로 전체를 검토한다.
- 병렬 연산으로 연산 속도를 증가시키면서 다방면으로 모델 학습 가능하다.

**Feed Forward Network**

![Untitled](/public/img/attention_transformer/Transformer%20d14d3d6dad1b49e5a17ca695ee00c09b/Untitled%2015.png)

- Self-Attention Layer의 출력에는 non-linearity가 적용되지 않았으므로 FFN 추가한다.
- 입력에 대해서 같은 weight를 갖는 FFN을 position별로 적용한다.
- 논문에서는 𝑑𝑚𝑜𝑑𝑒𝑙 = 512, 𝑑𝑓𝑓 = 2048
