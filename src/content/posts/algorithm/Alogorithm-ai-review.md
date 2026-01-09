---
title: AI에게 알고리즘 풀이를 리뷰받아보자!
published: 2026-01-09
tags:
  - Algorithm
  - AI
toc: false
lang: ko
abbrlink : algorithm-ai-review
---

최근 GeekNews에서 보게된 [알고리즘 풀이를 리뷰해주는 AI Github Action](https://news.hada.io/topic?id=25550)을 사용해봤습니다.  
마침 딱 취준하면서 1일1알고리즘을 실천중이던 저에게 적절한 적용이었습니다!   
  
> 사실 사용방법은 [AI Algorithm Methor 레포지토리](https://github.com/choam2426/AI-Algorithm-Mentor)에도 잘 나와 있습니다👍
  
##### 0️⃣ 준비물

1. 알고리즘 풀이 코드를 올릴 레포지토리
2. API Key : Gemini, GPT, ClaudeCode
3. 올릴 풀이 코드

##### 1️⃣ repository secret 추가

`Settings > Secrets> Repository secret` 에 API Key를 추가해줍니다.  
  
API Key 이름은 아래와 같습니다.  

- Gemini : `GEMINI_API_KEY`
- GPT : `OPENAI_API_KEY`
- ClaudeCode : `ANTHROPIC_API_KEY`

##### 2️⃣ github action workflow 생성

`.github/workflows` 디렉토리에 workflow를 생성합니다.  
  
저는 Anthopic api key를 사용했고,
아래 코드에서 `LLM_Provider, MODEL_NAME, *_API_KEY`만 바꿔서 작성하면 되겠죠?
  
```yaml
on:
  push:
    branches: [ main ]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: choam2426/AI-Algorithm-Mentor@v5
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          LLM_PROVIDER: anthropic              # openai, google, anthropic
          MODEL_NAME: claude-sonnet-4-5  # 모델명 
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          REVIEW_LANGUAGE: korean           # korean, english, etc..
```

`MODEL_NAME`은 아래와 같습니다.  
  
- Gemini : `gemini-3-pro-preview`
- GPT : `gpt-5.1`
- ClaudeCode : `claude-sonnet-4-5`
  
> 생성 전에 `ANTHROPIC_API_KEY : ${{ secrets.ANTHROPIC_API_KEY }}` 이 부분 잘 확인해보세요!  
> 저는 예시 코드 그냥 복붙해서 급하게 후다닥 했더니, `GEMINI_API_KEY : ${{ secrets.ANTHROPIC_API_KEY }}` 이렇게 작성했더라는...

##### 3️⃣ 알고리즘 풀이 코드 작성

이제 알고리즘을 열심히 풀면 되는데 리뷰를 받기 위해서는,  
‼️ **풀이 코드 "상단"에 주석으로 알고리즘 링크를 추가해야 합니다** ‼️
  
예시는 [README > 코드예시](https://github.com/choam2426/AI-Algorithm-Mentor?tab=readme-ov-file#-%EC%BD%94%EB%93%9C-%EC%98%88%EC%8B%9C)를 참고해주세용
  
이제 풀이를 작성했다면 코드를 커밋해서 올리면 됩니다.

##### 4️⃣ 리뷰 받기

![](../_images/algorithm-ai-review.png)

코드를 올리고 github action이 돌아가고 나서, 커밋 내역을 확인해보면..  
아름다운 리뷰가 코멘트로 남겨집니다!  
  
[직접 받은 리뷰](https://github.com/khyeji98/algorithm-study/commit/c836801dc51d7ca6db5e630fbbfba7924d63b95b#commitcomment-174187260)
  
두번정도 받았는데 ClaudeCode 기준 약 0.13$ 정도 사용했습니다. *(토큰에 따라 다르기 때문에 참고차!)*

---

코드를 올리기 전에 시간복잡도랑 공간복잡도를 먼저 예측하고, 리뷰를 통해 검토해보는 방식으로 메커니즘을 가져가면 좋을 것 같아요!  
아직 얼마 사용은 못했지만 개인적으로 매우매우 만족스럽다는 후기였습니다✨