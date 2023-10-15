---
title: "샤오미 노트 10 프로 Internal Storage 0mb 해결 방법"
excerpt: "TWRP와 Pixel Experience ROM을 설치한 이후 만족하게 사용하다가..."
header:
  teaser: /assets/posts/images/2023-10-03-10-17-05.png
categories:
  - Android
tags:
  - Xiaomi Note 10 Pro
  - OrangeFox
---

## 개요
- TWRP와 Pixel Experience ROM을 설치한 이후 만족하게 사용하다가 시스템 업데이트를 하는데 Recovery mode에서 Internal Storage 0MB로 나오고 Decrypt Data를 하려고 해도 비밀번호(PIN)이 제대로 먹히지 않아 ```Failed to mount /data``` 때문에 업데이트를 하지 못했다.
- Reddit과 XDA Developers 와 같은 해외 커뮤니티에서 아무리 찾아봐도 결국엔 Wipe(포맷) 하거나 Data의 Format 을 Ext2 로 변경했다가 Ext4로 변경하라는 옛날 글 뿐이었다.
- 이 글을 올리는 이 시점에 찾다 찾다 결국엔 발견하여 공유하고자 한다.

## 이야기
 처음 Pixel Experience 를 설치하고 깔끔하고 빨라서 마음에 들어 사용하고 있었다. 근데 며칠 지났나 시스템 업데이트가 생겨서 업데이트를 하려고 리커버리 모드(TWRP)를 들어가 설치하려고 보니 Internal Storage가 0MB로 나오고 Decrypt Data도 안되는 것이다.
 
 어차피 Data도 Mount 되어있지 않아 Backup도 안되고 혹시나 하는 마음에 Data 파티션을 Ext2 Format으로 변경하고 다시 Ext4로 변경했다. 원래는 F2FS 포맷이었다. Ext4로 변경하니 Internal Storage의 용량이 제대로 보여서 너무 기쁜 와중에 Reboot을 했더니 부팅이 제대로 안되는 것이다. 지금 생각해보면 파티션의 포맷이 변경되다보니 내 ROM이 설치된 포맷이 아니어서 그랬나보다.
 
 F2FS로 포맷을 다시 변경하니 역시나 Internal Storage는 또 ```0MB``` 이고 이것저것 만지다보니 결국 내가 이미 세팅해뒀던 앱이라던지 자료들이 전부 삭제된 상태였다. 깊은 빡침과 함께 어차피 데이터 다 지워진거 Wipe하고 업데이트된 ROM을 새로 설치했다.
 
 다시는 시스템 업데이트가 나와도 하지 않겠다고 결심했지만 2주가 지난 후 또 새로운 업데이트가 나왔다.
 
 ![](/assets/posts/images/2023-10-03-10-17-05.png)
  
 당시에 알림도 없고 아무것도 없는데 가만히 납둬도 화면이 자꾸 켜지는 것이다. 이 문제도 계속 찾고 있지만 아직 해결 방법은 찾지 못했다. 근데 업데이트 내용이 뭔가 이 화면 켜지는 버그의 Fix 내용인가하는 기대감이 있었다.
 
>  시스템 업데이트를 하기로 결심했다.

 하지만 업데이트를 하려면 Internal Storage 0MB부터 해결해야 했다. 구글에 열심히 검색을 했다. ```xiaomi note 10 pro internal storage 0```라고 검색하니 최상단에 ```Fix Internal Storage 0 MB on Android 13``` 라는 유튜브 영상이 있었다. Orangefox 12 버전이라는 비공식 버전을 설치하면 해결된다고 하는 것이다. 진짜 마지막 지푸라기라도 잡는 심정으로 유튜브를 보고 따라하며 설치를 했다.
 
 > 백업 과정은 data가 mount되지 않아 하진 않고 바로 Install 을 했다.
 
 설치하고 나서 Recovery mode로 들어가는데 평소와 달랐다! 비밀번호를 입력하라는 것이다. 난 곧바로 내가 설정한 PIN을 입력했다. (안드로이드 설정 - 보안 - 화면 잠금에서 설정한 PIN 번호) 뭔가 달랐다. 뭔가 Data 파티션이 마운트되는 것처럼 보였다. 마운트가 되버렸다..! sh*t..
 
 너무 기쁘고 들뜬 마음으로 시스템 업데이트를 설치했더니 잘되는 것이다.
 
 ```그랬다```

## 설치 방법
- 아래 참고에 있는 유튜브를 참고하길 바란다.

### 다운로드
[OrangeFox-R12.1_2-Unofficial-sweet.zip](https://sourceforge.net/projects/orangefox-releases/files/sweet/OrangeFox-R12.1_2-Unofficial-sweet.zip/download)

## 참고
[Fix Internal Storage 0 MB on Android 13 | Install OrangeFox Recovery 12.1 on Redmi Note 10 Pro](https://www.youtube.com/watch?v=nu1dZuO75bE)