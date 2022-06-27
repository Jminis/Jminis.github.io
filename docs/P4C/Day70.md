---
layout: default
title: Day70
nav_order: 1

parent: Last Project
grand_parent: P4C
---

##### 해당 게시글은 스스로 공부하기 위한 게시글입니다.

-----

마지막으로 받은 과제는 다 끝내기로 스스로에게 약속했다.

# > dreamhack.kr: web-misconf-1

![image-20220627130557412](../img/image-20220627130557412.png)

## writeup

주어진 문제 파일에 도커파일이 있길래 빌드를 해야하나 고민하다가 일단 웹페이지에 접속을 했다. 

![image-20220627130636380](../img/image-20220627130636380.png)

로그인 정보가 하나도 없는 상태에서 로그인 페이지만 존재했고, 이 웹사이트를 대상으로 injection을 해야하나 생각했다.

아무런 힌트가 없는 상황이여서 개발자도구 탭을 이용해 소스를 계속 읽어보았지만 마땅한게 없었다.

다운로드 받은 문제 파일을 다시 확인해보다가 `default.ini`라는 파일을 발견했고 해당 파일에서 `DH{`를 검색해보았는데

![image-20220627130806055](../img/image-20220627130806055.png)

조직 이름이 플래그 값으로 설정되어 있다는 힌트만을 얻었다. 결국에는 로그인하여 계정 정보나 설정란에서 확인할 수 있다는 것인데 어떻게 로그인을 할지 막막하다.

댓글을 통해 힌트를 봤는데 다들 5초만에 풀었다,,, 이게 웹해킹 맞냐,,, 라는 반응이 많아서 나만 바보인가 생각 중이다.

Grafana에 대해 검색을 해보았는데 "다중 플랫폼 오픈 소스 분석 및 대화형 시각화 웹 애플리케이션" 이라고 한다. 음?

댓글을 좀 더 읽다가 "허술하다" 라는 말에 꽂혀서 마치 기본 아이디가 있나싶어 "grafana" 기본 계정을 검색했더니...

![image-20220627131036205](../img/image-20220627131036205.png)

이 정보로 로그인이 되었고 `~/admin/settings` 페이지에서 플래그를 확인할 수 있었다.

![image-20220627131117970](../img/image-20220627131117970.png)

나만 15분이나 걸렸다...

<br><br><br>

-----

# > dreamhack.kr: web-ssrf

## writeup

```python
#!/usr/bin/python3
from flask import (
    Flask,
    request,
    render_template
)
import http.server
import threading
import requests
import os, random, base64
from urllib.parse import urlparse

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()  # Flag is here!!
except:
    FLAG = "[**FLAG**]"


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/img_viewer", methods=["GET", "POST"])
def img_viewer():
    if request.method == "GET":
        return render_template("img_viewer.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        urlp = urlparse(url)
        if url[0] == "/":
            url = "http://localhost:8000" + url
        elif ("localhost" in urlp.netloc) or ("127.0.0.1" in urlp.netloc):
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
            return render_template("img_viewer.html", img=img)
        try:
            data = requests.get(url, timeout=3).content
            img = base64.b64encode(data).decode("utf8")
        except:
            data = open("error.png", "rb").read()
            img = base64.b64encode(data).decode("utf8")
        return render_template("img_viewer.html", img=img)


local_host = "127.0.0.1"
local_port = random.randint(1500, 1800)
local_server = http.server.HTTPServer(
    (local_host, local_port), http.server.SimpleHTTPRequestHandler
)


def run_local_server():
    local_server.serve_forever()


threading._start_new_thread(run_local_server, ())

app.run(host="0.0.0.0", port=8000, threaded=True)
```



SSRF는 Server Side Request Forgery 약자로 CSRF와는 달리 서버에 영향을 주는 공격이다. 
