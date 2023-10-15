---
title: "Web Scrapping"
excerpt: "파이썬의 기초 첫번째, 웹 스크랩핑"
folder: "python"
categories:
    - Python
tags:
    - Python
header:
  teaser: /assets/posts/images/2023-10-04-22-16-15.png
toc: true
---

![](/assets/posts/images/2023-10-04-22-16-15.png)

# 스크랩핑이란?

- 어렸을 적 신문지에서 우리가 좋아하던 아이돌 또는 뉴스 기사를 오려다가 공책에 붙이는 행위를 스크랩하는 것이라고 들어봤을 것이다. 학교 숙제로도 많이 했었던 기억이 난다.
- 인터넷 상에서 우리가 필요한 정보를 어떠한 웹페이지에서 가져오는 것, 그것을 스크랩핑이라고 한다.

## 크롤링
- 스크랩은 한 페이지에서 특정 정보를 가져오는 것이라고 하면 크롤링은 그 페이지 자체를 가져오는 것이다.
```어원은 모르겠다.```


## 준비 과정
- 우리가 필요한 정보를 가진 웹 페이지
- 소스 코드 편집기(Visual Studio Code)
- python
 > https://www.python.org/downloads/
- pip
 > https://bootstrap.pypa.io/get-pip.py
 > 위 URL에서 다운받은 get-pip.py 가 있는 경로에서 cmd 명령어 입력
 > py get-pip.py
- BeautifulSoup4
 > pip install beautifulsoup4
 

## 기초 학습
 
### 1. 웹 페이지 정보 가져오기
 
- python의 requests 모듈을 이용하여 html 정보를 가져오자
 
#### requests 문법
![](/assets/posts/images/2023-10-04-22-16-35.png)

```requests.<HTTP Method>('URL')```

- 참고: https://requests.readthedocs.io/en/latest/
 
#### Content
- requests 모듈을 사용하여 원하는 정보를 추출하고자 할 때 그 정보가 어디에 있는지 아는게 중요하다.
- 보통 스크랩핑은 웹페이지 안의 **<font color="red">내용</font>**을 가져오고자 하니 content를 사용하지만 단순 HTTP Response Code가 필요한거라면 status_code를 요청할 수도 있다.

#### Source Code Sample

```python
import requests
 
req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
print(req.content)
```
 
### 2. html 소스를 시각화
- html 소스를 위 requests를 가져온다고 해도 text형태이기 때문에 가시성도 떨어질 뿐더러 그 안의 정보를 쉽게 추출하기란 어렵다. ```어렵다기 보단 눈이 아프다```
- 이를 쉽게 해주는 모듈이 스크랩핑의 꽃 <font color="red">BeautifulSoup4</font> 이다.
 
### 3. BeautifulSoup
- 사용법은 간단하다. 모듈을 import 한 상태에서 위에서 가져온 정보를 정해진 문법에 따라 우리가 보기 쉬운 html 형태로 바꾸는 것이다.

```python
import requests
from bs4 import BeautifulSoup
 
req = requests.get('https://URL')
scrap = BeautifulSoup(req.content, 'html.parser')
```

- 모듈을 사용하기 전
![](/assets/posts/images/2023-10-04-22-16-57.png)
 
- 모듈 사용 후
![](/assets/posts/images/2023-10-04-22-17-06.png)

#### find vs find_all
- 보기 좋게 만들었다고 해서 끝난 게 아니다. 내용 중에 우리가 필요한 정보를 찾아서 그 부분만 추출해야하는 작업을 해주어야 한다.
- 필요한 특정 정보를 찾기 위해 find 와 find_all을 사용할 수 있다.
- 이 둘의 차이점은 명확하다. 정해진 키워드를 1개를 찾으면 멈추던가, 키워드가 있는 모든 걸 찾거나이다. 아래의 이미지를 참고하길 바란다.
![](/assets/posts/images/2023-10-04-22-17-16.png)
- tr이라는 키워드를 find로 찾았을 때와 find_all로 찾았을 때의 차이를 그려낸 그림이다.
 
# 실습

## 목표
- KISA 보호나라의 취약점 정보를 가져와보자

## 준비 과정
- 웹 사이트
 > https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023

### 필요한 소스 확인
- 웹사이트에서 개발자모드(F12)를 진입하여 소스(Element)를 확인하자
![](/assets/posts/images/2023-10-04-22-20-57.png)

- 위 화면에서 우리가 필요한 정보는 "번호, 제목, CVSS, 게시일"이다.
- 소스코드를 한번 확인해보자
![](/assets/posts/images/2023-10-04-22-21-06.png)
- 위 소스코드에서 보면 table 태그안에 thead, tbody 가 있고, 우리가 필요한 정보는 tbody 안의 tr에 있는 것을 확인했다.

### Find 의 동작 방식
![](/assets/posts/images/2023-10-04-22-21-50.png)
- 위 이미지와 같이 우리가 찾는 정보는 tr에 있고 tr에 도달하려면 table과 tbody를 거쳐 tr을 찾아내야 한다.

```여기까지 이해했다면 바로 코드를 짜보자```

## 1. 스크랩핑할 웹 페이지 호출
- requests 모듈을 이용하여 웹페이지를 호출하자

```python
import requests

req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
```

- 위 코드를 print해보면 아래와 같다
![](/assets/posts/images/2023-10-04-22-22-11.png)
- Status code가 기본적으로 나온다
- 우리가 필요한 정보는 소스의 내용이므로 req.content를 출력해보면,,,
![](/assets/posts/images/2023-10-04-22-22-21.png)
- 얼추 내용처럼 보이긴 한다.

## 2. HTML 소스를 보기 좋게
- BeautifulSoup 모듈을 이용하여 내용을 가시적으로 볼 수 있게 만들어보자

```python
import requests
from bs4 import BeautifulSoup

req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
scrap = BeautifulSoup(req.content, 'html.parser')
```

![](/assets/posts/images/2023-10-04-22-22-31.png)
- 확실히 우리가 보기 좋은 형태로 변한 걸 확인할 수 있다

## 3. 정보 찾기
### 1) find, find_all을 이용하여 필요한 정보를 찾아보자

```python
table = scrap.find('table')
```

![](/assets/posts/images/2023-10-04-22-22-43.png)
- 필요한 정보까지 가까이 온 것 같다.
> 만약 해당 소스에 table이 여러개고 위 이미지와 같은 정보가 2번째의 table이라면 find_all을 이용하여 찾은 다음 인덱스를 이용하여 찾아야한다.
> ``` table = scrap.find_all('table') ```
> ``` print(table[1]) ```

### 2) thead에는 테이블의 식별? 제목같은 느낌의 내용만 있고 필요한 정보는 없다. tbody를 찾아보자

```python
tbody = table.find('tbody')
```

![](/assets/posts/images/2023-10-04-22-22-56.png)
- 변수가 여러개가 되면 지저분해질 수 있기 때문에 아래와 같이 해도 된다.

```python
table = scrap.find('table').find('tbody')
```


### 3) tbody안의 tr를 find와 find_all로 비교해보면,
- find
![](/assets/posts/images/2023-10-04-22-23-07.png)

- find_all
![](/assets/posts/images/2023-10-04-22-23-16.png)

> find는 tr의 첫번째 내용만 보이고, find_all은 모든 tr을 list 형태로 보여준다

### 4) tr 안의 내용 중 필요한 정보만 추출해보자
- td도 형태가 여러개이기 때문에 find_all을 통해 list 형태로 만들고 인덱스 형태로 바꿀 수 있다

```python
tds = trs.find_all('td')
```

![](/assets/posts/images/2023-10-04-22-23-27.png)
- 이제 list의 특정 내용만 보고 싶으면 index를 사용하여 필요한 부분을 추출하자
![](/assets/posts/images/2023-10-04-22-23-35.png)

- "번호"

```python
num = tds[0]
```

![](/assets/posts/images/2023-10-04-22-23-44.png)

### 5) .text로 내용만 추출
- .text를 이용하면 html 태그를 제외한 내용(text)만 볼 수 있다.

```python
num = tds[0].text
title = tds[1].text
severity = tds[2].text
cvss = tds[3].text
```

![](/assets/posts/images/2023-10-04-22-23-54.png)

### 6) 필요한 정보를 사전(dictionary) 형태로 보기

```python
vuln_dic = {'num':num, 'title':title, 'severity':severity, 'cvss': cvss}
```

![](/assets/posts/images/2023-10-04-22-24-05.png)

## 4. Source Code

```python
import requests
from bs4 import BeautifulSoup

req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
scrap = BeautifulSoup(req.content, 'html.parser')

table = scrap.find('table')
tbody = table.find('tbody')
trs = tbody.find('tr')

tds = trs.find_all('td')
num = tds[0].text
title = tds[1].text
severity = tds[2].text
cvss = tds[3].text
vuln_dic = {'num':num, 'title':title, 'severity':severity, 'cvss':cvss}

print(vuln_dic)
```


## 5. 실행 결과
![](/assets/posts/images/2023-10-04-22-24-20.png)
- 제목(title)에 '\n', '\r', '\t' 와 같은 개행문자가 들어가는데 다음 실습에서 개선해보도록 하겠다

# 실습 응용
- 위 실습에는 정보를 하나하나 찾아가는 방법을 익혔었다.
- 찾은 모든 정보를 가지고 가공하여 json 형태로 저장하는 방법을 배워보자.

## table의 형태
- 오늘 실습을 하기 위해선 table의 형태를 우선 알아야 한다.
- 지난 번에도 얘기했지만 table 안에는 thead(각 열의 제목)과 tbody(각 열의 내용)으로 나뉜다.
- tbody 안에는 하나의 행인 tr과 그 안에 셀 형태로 td가 있다.
- 아래 이미지를 참고하자.
![](/assets/posts/images/2023-10-04-22-28-50.png)

## 목표

```python
import requests
from bs4 import BeautifulSoup

req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
scrap = BeautifulSoup(req.content, 'html.parser')

table = scrap.find('table')
tbody = table.find('tbody')
trs = tbody.find('tr')

tds = trs.find_all('td')
num = tds[0].text
title = tds[1].text
title2 = title.replace("\n", "")
severity = tds[2].text
cvss = tds[3].text
vuln_dic = {'num':num, 'title':title2, 'severity':severity, 'cvss':cvss}

print(vuln_dic)
```

- 지난번 실습에서는 tr의 1개를 찾는 연습을 했지만 이번엔 모든 tr을 찾아 그 안의 td 내용들을 하나의 리스트형태로 저장하려고 한다.

## 과정
### 1. 모든 tr 찾기
- 지난번과의 다른 점은 tr을 find가 아닌 find_all로 찾는 것이다.

```python
trs = tbody.find_all("tr")
```

![](/assets/posts/images/2023-10-04-22-29-04.png)
- 모든 tr 안의 내용을 구분자(,)로 하여 리스트 형태로 가져오는 것을 볼 수 있다.
- ```<td> 내용 </td>``` 하나가 첫번째 내용이다.

### 2. 각 tr에 td들을 dictionary 형태로 저장
- 각 tr 들 안에도 ```<td>```가 여러개이고 이를 find_all로 찾아 필요한 내용만 추출할 수 있다.
- 동작 방식(로직)은 다음과 같다.
![](/assets/posts/images/2023-10-04-22-29-13.png)
- 우리는 for 문을 이용하여 각 tr의 td 내용들을 추출할 것이다.
- 참고로 for 문은 다음과 같은 형태로 파이썬 리스트에서 사용한다.
![](/assets/posts/images/2023-10-04-22-29-20.png)

### 3. for tr in trs:
- for 문을 이용하여 tr의 각 ```<td>```들을 리스트 순서대로 추출할 것이다.

```python
for tr in trs:
	tds = tr.find_all("td")
```

![](/assets/posts/images/2023-10-04-22-29-31.png)
- 위처럼 각 td들이 리스트형태로 분리된 것을 볼 수 있다.
- html태그는 필요하지 않으니 .text를 이용하여 내용만 가져와보자

```python
    num = tds[0].text
    title = tds[1].text
    severity = tds[2].text
    cvss = tds[3].text
```

- 각 리스트의 index를 보고 필요한 정보만 가져온 것이다. 
![](/assets/posts/images/2023-10-04-22-29-40.png)

### 4. dictionary 형태로 정보 저장

```python
dic = {'num':num, 'title':title, 'severity':severity, 'cvss':cvss}
```

- dictionary, 즉, 사전 형태로 출력할 수 있도록 함수를 작성했다.
> dictionary는 key:value 형태로 "A는 B다"라고 생각하면 된다.

![](/assets/posts/images/2023-10-04-22-30-03.png)
- 사전 형태로 가져오니 title에 \r, \n과 같은 개행문자가 보인다.
- 개행문자는 replace를 이용하여 없앨 수 있다.

```python
retitle = title.replace("\r", "").replace("\t", "").replace("\n", "")
```

- replace 문법은 ("A", "B"), A를 B로 대체(변경)하는 문법이다.
- 변경한 title 변수인 retitle을 'title': retitle로 변경해보자
![](/assets/posts/images/2023-10-04-22-30-16.png)


> 여기까지 왔다면 거의 다 끝났다고 볼 수 있다.

- 우리는 이제 하나의 리스트 1번을 완성했으니 나머지 tr행들의 정보도 가져와보자.
- 그러기 전에 빼먹은 게 하나 있다.
- 위의 1개 정보를 리스트에 저장을 해야하는데 저장할 변수를 지정하지 않았다.
- 리스트 타입의 변수 하나를 for문 전에 하나 만들어서 for문 마지막에 해당 정보를 하나씩 넣어보자.

### 5. list 정보 삽입
- list에 정보를 추가하기 위해선 우리는 append 라는 것을 사용한다.
- 리스트 타입의 변수 하나를 먼저 지정하자.

```python
vuln = []
```

- 그리고 위에 for문에서 하나의 정보가 만들어질 때마다 리스트에 저장하려면,,

```python
vuln.append(dic)
```

> vuln 변수는 꼭 for 문 전에 지정해줘야 한다. for문 안에 vuln 변수가 있다면 계속해서 리셋되어 마지막 정보만 vuln 변수에 저장될 것이다.

### 6. for문 완성

```python
vuln = []

for tr in trs:
    tds = tr.find_all("td")
    num = tds[0].text
    title = tds[1].text
    retitle = title.replace("\r", "").replace("\t", "").replace("\n", "")
    severity = tds[2].text
    cvss = tds[3].text
    dic = {'num':num, 'title':retitle, 'severity':severity, 'cvss':cvss}
    vuln.append(dic)
```

- 출력해보자
![](/assets/posts/images/2023-10-04-22-30-31.png)
- 위처럼 한 웹페이지 안에 모든 정보를 가져왔다.

### 7. json 형태로 만들기
- json 형태로 만들기 위해선 json 모듈을 삽입하여 json.dump(s)를 사용하면 된다.
- 메모리(일시적)형태로 json 형식으로 잘 만들어졌는지 확인해보자.

```python
jsondump = json.dumps(vuln, indent=4)
print(jsondump)
```

- jsondump라는 변수를 만들고 json.dumps(vuln(변수), indent(들여쓰기)=4)로 지정하고 출력해보면,,
> indent는 보통 4를 이용한다. 구분자(,)에 따라 줄바꿈을 해주어 이쁘게 작성된다.
> ![](/assets/posts/images/2023-10-04-22-30-41.png)
- 위와 같이 잘 만들어진 것을 볼 수 있다.

### 8. json 저장
- json을 저장하려면 우선은 json을 open 함수로 w(write:쓰기), 그리고 encoding을 지정하여 열어준다.

```python
with open('vuln.json', "w", encoding="utf-8") as make_file:
```

> with ~ as 변수: 는 with 하위의 동작이 끝나면 닫는다.

- json 파일(없을 시 생성됨)을 열어주고 write를 할 준비가 끝났다.
- 그렇다면 json 파일에 저장할 내용을 적어주자

```python
json.dump(vuln, make_file, ensure_ascii=False, indent=4)
```

- json.dump(입력해줄 내용, 어디에 저장할지(변수), 아스키코드 설정, 들여쓰기 지정) 문법 형태
- ensure_ascii=True일 경우 ascii코드(예: 0x00)가 아닐 경우 이스케이프 문자로 표기된다.

### 9. 코드 완성

```python
import requests
from bs4 import BeautifulSoup
import json

req = requests.get('https://www.krcert.or.kr/kr/bbs/list.do?bbsId=B0000302&menuNo=205023')
scrap = BeautifulSoup(req.content, 'html.parser')

table = scrap.find('table')
tbody = table.find('tbody')
trs = tbody.find_all('tr')

vuln = []

for tr in trs:
    tds = tr.find_all("td")
    num = tds[0].text
    title = tds[1].text
    retitle = title.replace("\r", "").replace("\t", "").replace("\n", "")
    severity = tds[2].text
    cvss = tds[3].text
    dic = {'num':num, 'title':retitle, 'severity':severity, 'cvss':cvss}
    vuln.append(dic)

with open('vuln.json', "w", encoding="utf-8") as make_file:
    json.dump(vuln, make_file, ensure_ascii=False, indent=4)
```


### 10. 저장된 json 파일 확인
![](/assets/posts/images/2023-10-04-22-31-10.png)

# 결론
- 본 실습의 경우 한 페이지에서만 스크랩하여 json 형태로 저장하는 법을 다루어보았다.
- 파이썬에서 스크랩핑은 사실 여기까지만 알고 있어도 거의 다 한거라고 볼 수 있다.
- 이를 어떻게 활용하고 어떤 방식으로 가져올 것인지는 역량에 있다고 생각한다.
