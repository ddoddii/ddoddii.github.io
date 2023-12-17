+++
author = "Soeun"
title = "FastAPI로 ML Model Endpoint 설계하기"
date = "2023-08-01"
summary = "FastAPI 실제로 활용하기 "
categories = [
    "CS"
]
tags = [
    "FastAPI"
]
image = ""
+++

## ML Model 의 결과값을 API Endpoint 로 설계하기 

지금까지는 FastAPI 설계하는 방법들과 검증하는 방법들에 대해 알아봤다. 
그러면 이제 실제 ML/DL Model 을 FastAPI 를 활용해 Deploy 하는 방법을 알아보자. 

### 1. Model 설계 
- 사용모델 : [HuggingFace 의 sentiment analysis 모델](https://huggingface.co/bhadresh-savani/distilbert-base-uncased-emotion)
- `main.py` 에 모델 불러오는 코드를 넣어두었다.

HuggingFace 에서 모델의 config.json, pytorch_model.bin, special_tokens_map.json, tokenizer_config.json, vocab.txt 을 다운받아서 ./model 아래 두었다. 

main.py 의 코드는 다음과 같다. HuggingFace 의 pipeline 을 사용하여서 매우 간단하다 !

```python
from transformers import pipeline

# 감정분류하기 
def sentiment(text):
    classifier = pipeline("text-classification",model='./model', top_k = 2)
    prediction = classifier(text)
    label_1, label_2 = prediction[0][0]['label'], prediction[0][1]['label']
    score_1, score_2 = round(prediction[0][0]['score'],3), round(prediction[0][1]['score'],3)
    return (label_1, label_2 ,score_1, score_2)
```

main.py 파일에서는 label_1, label_2, score_1, score_2 값을 리턴하도록 했다. 

### 2. API 설계

-  `api.py` 파일에서 FastAPI 를 이용해서 API Endpoint 를 설계
-  `api.py` 파일은 다음과 같다. 

우선 logger 를 설정해서 error 가 발생하면 로그가 발생하도록 했다. 
그리고 SentimentRequest , SentimentResult class 를 각각 만들어서 어떤 값이 input 이고 output 값은 어떤지 지정을 해줬다. 
POST Method 를 사용해서 입력값을 입력하고 가져오도록 했다. 
만약 입력한 text 값이 3글자 이하면, 201 HTTPException 를 리턴하도록 했다. 

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import logging

from main import sentiment

logger = logging.getLogger(name='MyLog')
logger.setLevel(logging.INFO)

formatter = logging.Formatter('|%(asctime)s||%(name)s||%(levelname)s|\n%(message)s',
                            datefmt='%Y-%m-%d %H:%M:%S'
                            )

stream_handler = logging.StreamHandler() 
stream_handler.setFormatter(formatter) 
logger.addHandler(stream_handler) 

app = FastAPI(title = "Soeun's Sentiment Analysis",
            version = "1.0")

class SentimentRequest(BaseModel):
    text : str = ""
    class Config:
        schema_extra = {
            'example': {
            'text' : 'Put in text to sentiment analysis'
            }
        }


class SentimentResult(BaseModel):
    sentence : str = ""
    label_1 : str = ""
    score_1 : float = ""
    label_2 : str = ""
    score_2 : float = ""

@app.post('/sentiment')
async def root(payload: SentimentRequest):
    if len(payload.text) <= 3:
        raise HTTPException(status_code = 201, detail = "Please put in valid text")
        
    try:
        label_1, label_2 ,score_1, score_2 = sentiment(payload.text)
        return SentimentResult(sentence = payload.text,
                            label_1 = label_1,
                            score_1 = score_1,
                            label_2 = label_2,
                            score_2 = score_2
        )
    
    except Exception as e:
        logger.error("Exception:"+str(e))
        raise HTTPException(status_code = 404, detail=f"Exception Error: {e}")
    
    
```

이것이 끝이다 ! 
실제로 LocalHost/docs 안에서 테스트 해보면 정상적으로 잘 작동하는 것을 볼 수 있다 😊

### Reference
- Udemy, FastAPI - The complete course 2023

### 사용한 자료와 코드
- 사용한 자료와 코드는 모두 제 GitHub 에서 볼 수 있습니다.
- [GitHub](https://github.com/ddoddii/skills-for-DS/tree/main/week3) `sentiment-api-project` 파일 참고