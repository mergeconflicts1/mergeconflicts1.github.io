---
layout: post
title:  Git의 저장소 구조 들여다 보기
description: 이번 포스트에서는 Git에서 저장소가 실제 어떻게 구현 돼 있는지 알아보겠습니다.
date:   2023-07-01 00:00:35 +0900
tags:   [저장소]
---

## 저장소 구조

{:.center}
![]({{site.baseurl}}/images/Post/local repo.png){:width="80%"}{:.centered}
[그림 1] Git의 구조

위 그림은 Git의 구조를 나타냅니다. Git을 오랫동안 사용하셨던 분들이라면 위 그림의 구조와 명령어 모두 익숙한 부분일 것 입니다.

Git은 로컬 저장소와 원격 저장소 영역으로 나눠 집니다.

로컬 저장소는 다시 작업 디렉토리(Working Directory), 스테이징 영역(Staging Area), 로컬 저장소(Local Repository) 로 나눠 집니다.

Git에서 로컬 저장소를 사용해 커밋을 생성하는 것은 보통 다음과 같은 과정으로 이루어 집니다.
> 1. 작업 디렉토리에서 코드를 수정
> 2. 수정한 내용을 스테이징 영역에 추가
> 3. 커밋을 수행하면 스테이징 영역에 있는 내용이 커밋으로 생성됨




그런데 여러분 Git의 로컬 저장소가 실제 어떻게 구현돼 있는지 궁금하지 않으시나요?

추상적으로는 알고 있지만 로컬 저장소가 실제 어떻게 구현돼 있는지 궁금해 하실 분들을 위해 이 부분을 한번 살펴 보겠습니다.

아래 그림은 로컬 저장소를 좀 더 자세히 표현한 그립입니다



{:.center}
![]({{site.baseurl}}/images/Post/repo detail.png){:width="50%"}{:.centered}
[그림 2] 로컬 저장소 구조



로컬 저장소는 위와 같이 `작업 디렉토리 - 인덱스 - 객체 데이터베이스`로 구성 돼 있습니다. 최초의 그림에서 스테이징 영역은 인덱스로 바뀌었고, 로컬 저장소는 객체 데이터베이스로 바뀌었습니다. 왜 바뀌었을까요?



아래 그림은 Git 저장소의 실제 모습입니다.



{:.center}
![]({{site.baseurl}}/images/Post/real local repo.png){:width="90%"}{:.centered}
[그림 3] Git 저장소의 실제 모습



Git에서 작업 디렉토리는 위 그림과 같이 `.git` 파일 밖에 위치해 있고, 스테이징 영역은 `index`라는 파일입니다. Git에서 생성하는 커밋는 `.git/objects` 라는 디렉토리에 저장됩니다.



참고로 `.git` 디렉토리는 숨김 설정이 돼 있어서 터미널에서 `ls -al` 명령을 사용하거나, 맥 파인더에서 숨은 파일 보기 단축키를 눌러야 볼 수 있습니다. ( 단축키 : `Command + Shift + .` )





## index 파일

Git에서 변경사항을 스테이징 영역에 추가 했을 때 `index`라는 파일이 어떻게 변하는지 살펴보겠습니다. `index` 파일은 바이너리 파일로 돼 있어서 내용을 `cat` 명령으로는 `index`의 내용을 직접 볼 수 없습니다.

```terminal
% cat index
DIRCdˈz?dˇ?7t?????????Q??.@G?n??G14???)?The Great Developer List.txtTREE1 0
??
  ????O'H???A?E?-~F??????2?]A
                             >M`?%
```
<p align="center">[cat index 결과]</p>



Git에서는 `index` 파일을 볼 수 있는 명령([git ls-files --stage](https://git-scm.com/docs/git-ls-files)) 을 제공해 줍니다.



터미널에서 명령을 입력하면 현재 스테이징 영역의 상태를 보여줍니다.

```terminal
 % git ls-files --stage
100644 ec51cbdf2e4047d06ea6ec473134b3fd03c129ed 0	The Great Developer List.txt
```

<p align="center">[git ls-files --stage 결과]</p>



스테이징 영역에 다음과 같은 변경 사항을 추가해 보겠습니다.

{:.center}
![]({{site.baseurl}}/images/Post/repo staging.png){:width="100%"}{:.centered}
[그림 4] 변경된 파일을 스테이징 영역에 넣은 모습



`git ls-files --stage` 명령을 입력하면 다음과 같이 해시가 바뀐 데이터(`729d081`)가 스테이징 영역에 변경사항이 들어있는 것을 확인할 수 있습니다.

```terminal
git ls-files --stage
100644 729d081228239f226fc19ccde6361b23c4f45be2 0	The Great Developer List.txt
```

현재 커밋 로그는 다음과 같습니다.

```terminal
% git log --oneline
77edeb2 (HEAD -> master) WWW
9a55a21 Unix and C
b6fd775 Python
d94305b 리눅스 Git
4f8cf86 Java
771049f StackOverflow
f60dfef Slack
2e37ac4 Github
324f320 Init
```

이 상태에서 스테이징 영역에 있는 내용을 커밋으로 생성해 보겠습니다.

"깃미남" 이라는 메시지를 갖는 커밋이 하나 생성 됐습니다. 이 커밋의 ID는 df00403(`df0040314160da4ee1c9e6acc894f96883e347ac`) 입니다.

```terminal
% git log --oneline
df00403 (HEAD -> master) 깃미남
77edeb2 WWW
9a55a21 Unix and C
b6fd775 Python
d94305b 리눅스 Git
4f8cf86 Java
771049f StackOverflow
f60dfef Slack
2e37ac4 Github
324f320 Init
```



## 객체 데이터베이스

방금 전 생성한 깃미남 커밋(`df0040314160da4ee1c9e6acc894f96883e347ac`)이 Git에서 어떻게 저장돼 있는지 살펴보겠습니다. `/.git/objects` 디렉토리를 보면 이상한 두 개의 문자로 구성된 여러 디렉토리 리스트를 볼 수 있습니다.



{:.center}
![]({{site.baseurl}}/images/Post/repo objects.png){:width="100%"}{:.centered}
[그림 5] `.git/objects` 디렉토리 내용



방금 전 생성한 커밋의 ID는 `df0040314160da4ee1c9e6acc894f96883e347ac` 입니다. ID의 앞 두글자는 `df` 입니다. 위 [그림 5]의 `/.git/objects` 디렉토리 리스트에 `df`로 시작하는 디렉토리가 존재하는 것을 확인 할 수 있습니다.



{:.center}
![]({{site.baseurl}}/images/Post/repo commit.png){:width="100%"}{:.centered}
[그림 5] `df` 디렉토리 안에 파일



디렉토리에 들어가보면 `0040314160da4ee1c9e6acc894f96883e347ac`라는 파일이 있는 것을 확인 할 수 있습니다.

뭔가 감이 오시나요?

Git은 커밋을 커밋의 ID인 40자리 해시 중 **앞에서 두 글자로 디렉토리**로 생성하고 해시의 **나머지 38자리를 이름으로 하는 파일**로 저장합니다. 커밋 뿐만 아니라 Git에서 데이터를 저장하기 위해 사용하는 다른 객체인 트리(tree), 블랍(blob), 태그(tag) 모두 동일한 방법으로 이곳에 저장합니다.



파일을 확인한 김에 파일의 내용도 살펴볼까요? Git에서 저장하는 객체 파일은 gzip으로 압축돼 있어서 이 파일 역시 cat명령으로 내용을 볼 수 없습니다.



Git에서는 Git 객체의 내용을 볼 수 있는 [git cat-file](https://git-scm.com/docs/git-cat-file) 이라는 명령을 제공해 줍니다. `-p` 옵션을 사용해 객체의 해시를 명시하면 해당 객체의 내용을 확인할 수 있습니다.

```terminal
% git cat-file -p df00403
tree eedd805bff038311ea99e642ecf3dd9bbec1be05
parent 77edeb281ccc6a5e1ba759a1dd2bf8e45710c911
author 깃미남 <mergeconflicts1@gmail.com> 1691148535 +0900
committer 깃미남 <mergeconflicts1@gmail.com> 1691148535 +0900

깃미남
```

객채의 내용을  보면 저희가 앞에서 커밋한 내용을 담고 있는 것을 확인할 수 있습니다.



## 정리

이번 포스트에서 알아본 내용을 정리해 보겠습니다.



• Git의 로컬 저장소는 작업 디렉토리, 인덱스, 객체 데이터베이스로 구성된다.

• 인덱스인 `.git/index` 파일은 스테이징 역할을 수행하고 커밋을 수행하면 객체 데이터 베이스인 `.git/objects` 디렉 토리에 저장된다.

• 스테이징 영역의 상태는 `git ls-files --stage` 명령을 사용해 확인할 수 있다.

• 객체가 저장 될 때에는 40자리 해시 중 앞 두 자리는 디렉토리 이름으로 사용되고 나머지 38자리는 파일 이름으로 사용된다.

• 객체의 내용은 `git cat-fiel -p (HASH)` 명령을 통해 확인할 수 있다.



<br/>

배웠던 내용 모두 이해하셨죠? 이해 하셨다면 잊어버리셔도 됩니다. 나중에 다시 찾아보면 되니까요.



👨🏻‍💻 Git 지식이 +10 늘었다. 다음 포스트에서 또 만나요 🚀😄
