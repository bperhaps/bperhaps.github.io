---
title: "카카오톡 저장기간 만료 파일 복구"
date: 2018-02-10 05:28:00 +0900
categories: project kakaotalk-recover
tags: kakaotalk
sidebar:
  nav: "projects"
---

카카오톡으로 사진들을 주고받다 보면 저장기간이 만료되서 사진을 다운로드 하지 못하는 경우가 있다.
{:refdef: style="text-align: center;"}
![fig1](/assets/images/posts/kakao-recover/fig1.jpg)
{: refdef}
*<center>사진1. 저장 기간 만료로 파일을 불러올 수 없는 사진.</center>*

하지만 잘 보면, 다운로드는 불가능하지만 흐릿하게 해당 사진을 볼 수 있는것을 확인할 수 있다.
여기서 저 사진을 가져올 수 있으면 저장 기간을 놓쳐 다운받지 못한 사진도 받을 수 있지 않을까? 하는 의문을 품게 되었다.

먼저 카카오톡의 사진들의 위치를 알아내야 했고, 경로를 찾아 내었다.



>카카오톡Media파일 저장 위치 : /Android/data/com.kakao.talk

해당 경로를 들어가면 다음과 같은 폴더들을 볼 수 있다.
{:refdef: style="text-align: center;"}
![fig2](/assets/images/posts/kakao-recover/fig2.png)
{: refdef}
*<center>사진2. 카카오톡 Media파일 위치</center>*

우리가 사용할 폴더는 contents 폴더이다. 대화방에서 주고받은 사진들의 정보들은 저 폴더안에 다 들어있다고 보면 된다.

해당 파일을 컴퓨터도 이동시킨 후 안의 내용물을 보면 ???? 가 나올 수 밖에 없다.
{:refdef: style="text-align: center;"}
![fig3](/assets/images/posts/kakao-recover/fig3.png)
{: refdef}
*<center>사진3. Contents 폴더의 내부</center>*


파일과 폴더들의 이름들이 해쉬화 되어있는것을 볼 수 있으며 심지어 파일들은 확장자도 존재하지 않는다.
처음에는 파일들이 암호화돼 있나? 하는 끔찍한 상상을 하였지만 다행이 그것은 아니었다.
{:refdef: style="text-align: center;"}
![fig4](/assets/images/posts/kakao-recover/fig4.png)
![fig5](/assets/images/posts/kakao-recover/fig5.png)
{: refdef}
*<center>그림4. HXD를 이용해 확인한 파일 형식</center>*

HXD를 이용해 파일들을 열어보니 파일 시그니처가 해당 파일이 무엇인지를 정확하게 명시해 주고 있었다.
필요한 정보는 모두 얻었다. 코딩만 하면 된다. 파이썬을 이용하여 코딩할 것인데, 파일 시그니처를 이용해 파일들의 확장자를 붙여줄 것이다.
원래는 사진, 동영상 정도의 파일만 있을 것이라 생각하여 라이브러리를 안쓰고 직접 시그니처들을 리스트에 넣어 비교할 예정이었으나 본인의 데이터(약 1년 6개월)를 분석한 결과
{:refdef: style="text-align: center;"}
![fig6](/assets/images/posts/kakao-recover/fig6.png)
{: refdef}
*<center>사진5. 파일들의 앞 4바이트 리스트</center>*

사진뿐만 아니라 다른 잡다한 파일의 시그니처도 확인되는걸 볼 수 있었다.
물론, 사진 동영상의 시그니처만 처리하고 나머지는 예외처리시켜버리면 라이브러리를 쓰지 않고도 충분히 가능하긴 하지만.. 그건 성격상 안맞기도 하고 (귀찮..) 그냥 라이브러리를 사용하기로 했다.
사용한 라이브러리는 python-magic이다.

```
# -*- coding: cp949 -*-

import sys
import os
import time
import shutil
import magic


# Referenced from "https://wikidocs.net/39"
def search(dirname):
    filenames = os.listdir(dirname)
    for filename in filenames:
        full_filename = os.path.join(dirname, filename)
        if os.path.isdir(full_filename):
            search(full_filename)
        else:
            path = time.strftime("%Y.%m.%d" , time.localtime(os.path.getmtime(full_filename)))
            extention = magic.from_file(full_filename, mime=True).split("/")[1]

            if not os.path.exists("./result/" + path):
                os.mkdir("./result/" + path)

            shutil.copy(full_filename, "./result/" + path + "/" + filename + "." + extention)     

if not os.path.exists("result"):
    os.mkdir("result");

search("/mnt/g/com.kakao.talk/contents")
```


{:refdef: style="text-align: center;"}
![fig7](/assets/images/posts/kakao-recover/fig7.png)
![fig8](/assets/images/posts/kakao-recover/fig8.png)
{: refdef}
*<center>사진6. 결과</center>*



실행시켰을 경우 결과는 다음과 같고, 다운로드 기간이 지난 사진이라도 가져올 수 있는 것을 볼 수 있다.
물론, 원본사진은 아니기에 해상도가 떨어지는 부분은 어쩔 수 없다. 하지만 요즘 기술이 발달하여 딥러닝으로 해상도 올려주는 것이 있으니 확인해보면 좋을 것 이다. 아래 링크를 참고하시길..
참고 : http://waifu2x.udp.jp/
채팅방 관련 db가 존재하므로 채팅방별 정렬을 시도하려 했지만, 해당 db를 가져오기 위해선 루팅이 필요하고 안드로이드 버전이 올라감에 따라 퍼미션이 많이 견고해 져서 루팅없이 adb로 가져오는 것이 불가능해 졌고, 현재 나와있는 루팅방법은 폰을 초기화 시켜야 하는 것 이므로.. 초기화 없이 하는 루팅방법이 나오지 않는 이상은 사실상 불가능한 것 같다.
.
