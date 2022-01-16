---
layout: post
title: fast api는 멀티 프로세스일까, 멀티 스레드일까?
date: 2022-01-15 19:22:23 +0900
category: dev
---
# uvicorn의 worker는 멀티 프로세스일까, 멀티 스레드일까?

fastapi + uvicorn을 활용해 서버를 구축하던 중 궁금한점이 생겼다.
uvicorn의 worker를 늘리면 multi process일까, multi thread일까?

간단한 프로세스와 스레드의 개념을 찾아봤다.

__멀티 프로세스__
- 한개의 프로세스를 여러개로 복사해 작업을 수행
- 각 스레드마다 메모리 영역이 있어 서로 독립적
- 한 프로세스가 죽어도 다른 프로세스는 수행됨

__멀티 스레드__
- 하나의 프로세스에서 여러개의 스레드를 생성해 작업을 수행
- 이 스레드들은 하나의 공유 메모리 공간을 공유하고, context switch에 장점이 있음
- 메인 프로세스가 죽으면 모든 스레드가 종료됨

생각보다 답은 공식문서에서 바로 찾을 수 있었다..

![](../public/img/fastapi-workers.png)

실제로 multiprocessing 모듈을 이용해 간단히 테스트할 수 있다.

```
import uvicorn
import multiprocessing
from fastapi import FastAPI

app = FastAPI()


@app.get("/check")
def hello():
    process_name = multiprocessing.current_process().name
    return {"process name": process_name}


if __name__ == '__main__':
    uvicorn.run("server:app", port=8090, workers=16)
```
localhost:8090/check에 접속할때마다 process의 name이 변하는것을 알 수있다.

*__결론? uvicorn의 worker는 process 기반이다.__*
