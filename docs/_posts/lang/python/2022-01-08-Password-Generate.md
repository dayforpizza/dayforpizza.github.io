---
title: "Study Coding: Password Generate"
excerpt: "Make your own password in cycle"
categories:
  - Python
tags:
  - Python
---

# Concept
- 패스워드 생성

# Rule
- URL 입력 (ex. naver.com)
- URL 정보 축략 (ex. naver)
- 사용자 식별자 입력 (ex. m3yh)

## Output
- 식별자 + @ + URL축략정보 + 축략자 글자수

# code
```python
#!/usr/bin/python3
import os, hashlib

# 안내 문구 출력
print("Password is generated as an under format\n > [Default Word]@[Website Name]][Website Count]")

# URL 입력
url = input("URL: ")

# URL 축약
pw_info = url.replace("http://", "")
pw_info = pw_info.replace("https://", "")
pw_info = pw_info.replace("www.", "")
pw_info = pw_info[:pw_info.index(".")]

# 기본으로 들어갈 단어 입력
default = input("Default Word: ")

# 패스워드 생성
password = str(default) + "@" + str(pw_info[0].upper()) + str(pw_info[1:]) + str(len(pw_info))

# URL의 이름
name = pw_info[0].upper() + pw_info[1:]

# 비밀번호 해쉬값
pw_hash = hashlib.md5(str(password).encode('utf-8')).hexdigest()

print("{0}'s password : {1}".format(name, password))
print("<Hash: {}>".format(pw_hash))

os.system("pause")
```