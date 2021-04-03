---
title:  "centos7에 python, uwsgi, flask, nginx 설치하기"
excerpt: ""

categories:
  - Blog
tags:
  - ["centos", "python", "uwsgi", "flask", "nginx"]

toc: true
toc_sticky: true
 
date: 2021-04-03
last_modified_at: 2021-04-03
---





-----
# read me
```
python 의 버전은 3.5를 사용하지 않아도 좋습니다. 
하지만 3.9 보다는 낮은 버전을 추천합니다. 그 이유는 너무 상위 버전의 python을 사용하다 보면 라이브러리 충돌이 쉽게 발생하기 때문입니다. 3.5 혹은 3.6버전의 파이썬 사용시 라이브러리의 충돌 및 오류가 가장 적었습니다.

centos의 버전은 7입니다. 6에서도 동일한 설정시 프로젝트의 동작에 문제가 없었지만 가장 최근에 테스트를 했던 버전은 7이었습니다.
```


# python3.5 설치

`python3.5, 기본적으로 필요한 것들을 모두 설치해줍니다. `

1.  #### repository 추가
    

```
sudo yum install -y epel-release.noarch
sudo yum install -y https://repo.ius.io/ius-release-el7.rpm
```

2.  #### python 3.x 버젼 확인
    

```
yum search python3
```

3.  #### 설치
    

```
yum install -y python35u.x86_64, python35u-debug.x86_64, python35u-devel.x86_64,python35u-libspython35u-libs.x86_64, python35u-pip.noarch, python35u-setuptools.noarch, python35u-tools.x86_64, python35u-libs, python35u-lxml 
pip3.5 install --upgrade pip
```

4.  #### 심볼릭 링크 추가
    

`python3.5 로 호출해서 사용할 예정이라면 심볼릭 링크를 추가하지 않아도 괜찮습니다.`

```
[root@centos] which python3.5
/usr/bin/python3.5
[root@centos] cd /usr/bin
[root@bin] ln -s python3.5 python3
```

---

<br>
<br>

# pip 설치
`각종 라이브러리를 설치하기 위해서 pip를 설치해줍니다.`

1.  #### repository 추가
    

```
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

2.  #### 설치
    

```
yum install -y python35u-pip
```

3.  #### 버젼 확인
    

```
[root@bin] pip3.5 --version
pip 9.0.1 from /usr/lib/python3.5/site-packages (python 3.5)
```

4.  #### 심볼릭 링크 추가
    

-   pip3.5로 호출해서 사용할 예정이라면 심볼릭 링크를 추가하지 않아도 괜찮다

```
which pip3.5
cd /usr/bin
ln -s pip pip3.5
```

5.  #### 업그레이드
    

```
pip install --upgrade pip
```

---

<br>
<br>

# 가상환경 설정

-----

`가상환경을 설치해주지 않아도 파이썬 사용은 가능하지만 각종 오류로 환경이 오염된 경우에는 가상환경만 날려주면 깔끔하게 새 가상환경에서 설치해줄 수 있기 때문에 설치해주는 것이 좋습니다.그리고 하나의 서버에서 여러개의 파이썬 프로젝트를 사용할 것이라면 가상환경은 필수로 설정이 필요합니다`

-----

1.  #### 가상환경 설치
    

```
pip install virtualenv
```

2.  #### 가상환경 만들기
    

-   내 경우에는 /home/env/로 잡았다
-   이 경로 안에서 virtualenv test\_env를 하면 test\_env가 생기고 그 안에 설치된다

```
cd /가상환경설치할 경로
virtualenv 가상환경이름
```

3.  #### 가상환경 path 설정
    

```
cd /가상환경설치경로/가상환경이름/bin
vim activate
```

-   가장 아래줄에 path 추가

```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
export PYTHONPATH=${PYTHONPATH}:/home/프로젝트경로/

이때 프로젝트의 경로가 조금 혼란스러울 수 있습니다.
/home/python-project 안에 wsgi.py가 존재하며,
나머지 파일들은 /home/python-project/test/application.py가 있다면
최상위 경로인 /home/python-project/가 설정할 경로입니다. 
```

---

<br>
<br>

# MySQLdb 설치
`가상환경 activate 상태로 진행합니다.`

-----

`MySQLdb는 설치 도중 오류가 굉장히 많이 발생하는 라이브러리입니다.
어느정도냐면 3년간 python project를 centos 서버에 설치해왔는데 오류가 아예 나지 않은 일은 없습니다. 때문에 따로 정리합니다. `

-----

1.  #### 설치 파일 다운로드
    

```
wget https://pypi.python.org/packages/6f/86/bad31f1c1bb0cc99e88ca2adb7cb5c71f7a6540c1bb001480513de76a931/mysqlclient-1.3.12.tar.gz#md5=dbf1716e2c01966afec0198d75ce7e69 
```

2.  #### 압축 해제 후에 설치
    

```
tar xvfz mysqlclient-1.3.12.tar.gz
cd mysqlclient-1.3.12 
yum -y install mariadb-devel gcc python35u-devel 
python3 setup.py clean 
python3 setup.py build
python3 setup.py install
```

3.  #### 오류 발생시
    

`이 아래 내용까지 실패하면 현재 환경은 깨끗하게 버리고 가상환경 새로 파는게 빠릅니다. 아니면 다른 서버에서 설치하고 라이브러리만 복사해오는게 빠를수도 있습니다`

```
yum -y install gcc-c++ zlib zlib-devel libffi-devel
wget https://www.python.org/ftp/python/3.5.0/Python-3.5.0.tgz tar xzf Python-3.5.0.tgz cd Python-3.5.0.tgz 
./configure --enable-optimizations
make
make install
pip install mysqlclient, MySQL-db
```

---

<br>
<br>

# 라이브러리 설치

```
pip install uwsgi, flask, flask_restful
```

---

<br>
<br>

# nginx 설치


1.  #### repository 추가
    

-   vim /etc/yum.repos.d/nginx.repo

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

2.  #### 설치
    

```
yum -y install nginx
```

3.  #### 설정 파일 수정
    

-   vim /etc/nginx/conf.d/default.conf

```
server {
    listen       80;
    server_name  localhost;

    location / {
        try_files $uri @test;
    }

    location @test {
        include uwsgi_params;
        uwsgi_pass unix:/tmp/test.sock;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

4.  #### 시작
    

-   systemctl enable nginx
-   systemctl start nginx

---


<br>
<br>

# 테스트

1.  #### 테스트 수행 조건
    

`제대로 설치 되었는지, 제대로 동작하는지 확인이 필요하다면 3단계의 테스트가 필요합니다`

* python project
* python project + uwsgi
* python project + uwsgi + nginx

`이렇게 3단계 테스트를 진행하는 이유는 3번만 덜렁 테스트 진행시 오류가 발생하면 누가 바보인지 알기 힘들기 때문입니다. 차라리 깔끔하게 3단계로 진행하는것이 오류 파악도 더 빠릅니다. `

2.  #### 테스트 프로젝트 생성
    

-   /home/test 경로에 파일을 생성하고 테스트를 수행한다

-   wsgi.py

```
from app import app

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)

```

-   app.py

```
from flask import Flask
import requests

app = Flask(__name__)
@app.route("/test/<test>", methods=["GET"])
def test1(test):
    print(test)
    return test
```

-   test-uwsgi.ini

```
[uwsgi]
chdir = /home/test/
module = wsgi:app
master = true
processes = 5
socket = /tmp/test.sock
chown-socket = nginx:nginx
chmod-socket = 666
vacuum = true
die-on-term = true
pyargv=123
socket-timeout = 3000
http-timeout = 3000
ignore-sigpipe = true
ignore-write-errors = true
disable-write-exception = true
```

3.  #### 테스트
    

-   python project
    -   테스트용 프로젝트를 설치해둔 경로로 이동 후에 백그라운드로 실행해서 확인한다

```
(test_env) [root@test] nohup python3 wsgi.py &
(test_env) [root@test] curl -X GET http://localhost:5000/test/testProjectPrint
(test_env) [root@test] curl -X GET http://localhost:5000/test/testProjectPrint
testProjectPrint(test_env)
```

-   python + uwsgi

```
(test_env) [root@test] uwsgi --socket :5000 --protocol=http --wsgi-file wsgi.py --callable app
(test_env) [root@test] curl -X GET http://localhost:5000/test/testProjectPrint
testProjectPrint(test_env)
```

-   python + uwsgi + nginx

```
systemctl restart nginx
uwsgi test.ini &
```

---

<br>
<br>

# 오류 확인하기

1.  #### permission denied
    

```
2021/03/29 08:31:45 [crit] 7306#7306: *1 connect() to unix:/tmp/test.sock failed (13: Permission denied) while connecting to upstream, client: 127.0.0.1, server: localhost, request: "GET /test/test HTTP/1.1", upstream: "uwsgi://unix:/tmp/test.sock:", host: "localhost"
```


`여기서 이상함을 느껴야 합니다. 왜냐면 uwsgi.ini 에서 분명 아래의 2개의 설정을 추가했기 때문입니다.`


```
chown-socket = nginx:nginx
chmod-socket = 666
```

`혹시나 모르니까 권한을 한번 확인해봅니다`

-   ls -al /tmp

```
srw-rw-rw-.  1 nginx nginx   0 Mar 29 08:31 test.sock
```

-   selinux 문제입니다.
-   nginx는 centos 6.6 이상에서 httpd\_t 컨텍스트로 레이블이 지정됩니다. 
-   허용 도메인 httpd\_t 를 목록에 추가하기
    -   semanage permissive -a httpd\_t
-   허용 도메인 httpd\_t 를 목록에서 삭제
    -   semanage permissive -d httpd\_t

```
[root@] ps auZ | grep nginx
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 7397 0.0  0.0 112816 980 pts/2 S+ 08:49   0:00 grep --color=auto nginx
[root@] setsebool -P httpd_can_netword_connect on
[root@] semanage permissive -a httpd_t
```


<br>
<br>

# 참고 문헌

[www.nginx.com/blog/using-nginx-plus-with-selinux/](https://www.nginx.com/blog/using-nginx-plus-with-selinux/)