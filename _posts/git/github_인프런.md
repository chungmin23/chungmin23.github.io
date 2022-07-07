## Git 최초 설정

### Git 전역으로 사용자 이름과 이메일 주소를 설정

~~~linux
git config --global user.name "본인이름"
git config --global user.email "본인 이메일"
~~~

확인

~~~
git config --global user.name
git config --global user.email
~~~

기본 브랜츠명 변경 

~~~
git config --global init.defaultBranch main
~~~

### 프로젝트생성 초기화

~~~
git init
~~~

### 현재 폴더 깃 상태 보여줌 

~~~
git status
~~~



### git 에서 관리에서 특정파일/폴더를 배재해야 하는 겨우

.gitignore 파일 생성 

~~~
예제 

# 이렇게 #를 사용해서 주석

# 모든 file.c
file.c

# 최상위 폴더의 file.c
/file.c

# 모든 .c 확장자 파일
*.c

# .c 확장자지만 무시하지 않을 파일
!not_ignore_this.c

# logs란 이름의 파일 또는 폴더와 그 내용들
logs

# logs란 이름의 폴더와 그 내용들
logs/

# logs 폴더 바로 안의 debug.log와 .c 파일들
logs/debug.log
logs/*.c

# logs 폴더 바로 안, 또는 그 안의 다른 폴더(들) 안의 debug.log
logs/**/debug.log
~~~



---

## 프로젝트의 변경사항들을 타임캡슐(버전)에 담기

커밋 : 버전

 untracked : git관리에 간적 없는 파일 

파일 하나 담기

~~~
git add tiger.yaml
~~~



모든 파일 담기 

~~~
git add .
~~~



---

## 타임캡슐 묻기

~~~
gut commit -m "first commit"
~~~

명령어와 소스트리 확인

~~~
git log
~~~

add와 commit 한꺼번에

~~~
git commit -am "(메시지)"
~~~

🛑 단 새로 추가된(untracked) 파일이 없을 때 한정

---

## 과거로 돌아가는 두가지 방법

reset: 과거에 있는 기록을 모두 지움

revert: 이전에 작업한 부분만 해서 새로 커밋을 추가해서 이전으로 돌림



## **reset** 사용해서 과거로 돌아가기

~~~
git reset --hard (돌아갈 커밋 해시)
~~~



## **reset** 하기 전 시점으로 복원해보기

~~~
git reset --hard
~~~



## **revert** 로 과거의 커밋 되돌리기

~~~
git revert (되돌릴 커밋 해시)
~~~

:wq로 커밋메세지 저장

---

## 여러 branch 만들어보기

브런치 생성

~~~
git branch add-coach
~~~



브런치 목록 확인

~~~
git branch
~~~



add-coach브런치 이동

~~~
git switch add-coach
~~~



브런치 생성과 동시에 이동하기

~~~
git switch -c new-teams
~~~



브런치 삭제하기

~~~
git branch -d (삭제할 브랜치명)
~~~



브랜치 이름 바꾸기

~~~
git branch -m (기존 브랜치명) (새 브랜치명)
~~~



여러 브랜치 내역 확인하기

~~~
git log --all --decorate --oneline --graph
~~~

---

## 브랜치 합치기

- **merge** : 두 브랜치를 한 커밋에 이어붙입니다.
  - 브랜치 사용내역을 남길 필요가 있을 때 적합한 방식입니다.
  - 다른 형태의 merge에 대해서도 이후 다루게 될 것입니다.
- **rebase** : 브랜치를 다른 브랜치에 이어붙입니다.
  - 한 줄로 깔끔히 정리된 내역을 유지하기 원할 때 적합합니다.
  - 이미 팀원과 공유된 커밋들에 대해서는 사용하지 않는 것이 좋습니다.

###  **merge**로 합치기

~~~
//메인 브런치로 이동
git switch main

//merge 합치기 
//맥인 경우 :wq
git merge add-coach

//소스트리에서 이커밋까지 현재 브런치를 초기화를 눌러서 브랜치 되돌리기

//병합된 브랜치는 삭제
git branch -d add-coach

~~~



### **rebase**로 합치기

~~~
//new-teams 브런치로 이동 
git switch new-teams

// rebase 합치기
git rebase main

// main인 뒤쳐져 있는 상황이라서 
git merge new-teams 

//new-teams 브랜치 삭제
git branch -d new-teams

~~~

---

## 브랜치 충돌 해결하기



### `merge` 충돌 해결하기

~~~
//당장 해결이 어려운 경우 merge 중단
git merge --abort

해결 가능 시 충돌 부분을 수정한 뒤 `git add .`, `git commit`으로 병합 완료
~~~



### `rebase` 충돌 해결하기

~~~
//당장 해결이 어려운 경우 merge 중단
git rebase --abort

//해결 가능시
git add . 
git rebase --continue
//완료될때까지 반복 

git switch main
//밀린 것 복구 
git merge confliect-2 

//남은 브런치 삭제
git branch -d conflict-1
git branch -d conflict-2

~~~



### 원격 저장소 사용하기

~~~
//원격저장소 연결 추가
git remote add origin (원격 저장소 주소) 

//기본 브런치를 메인으로
git branch -M main

//로컬 저장소의 커밋 내역들 원격으로 push
git push -u origin main 

// 원격 목록 보기
git remote 

// 원격 지우기 (로컬 프로젝트와의 연결만 없애는 것. GitHub의 레포지토리는 지워지지 않음)
git remote remove (origin 등 원격 이름)

//프로젝트 받기
git clone (원격 저장소 주소)
~~~

---

## 브랜치 충돌 해결하기

~~~
//원격으로 커밋 밀어올리기 
git push

// 원격의 커밋 당겨오기
git pull
~~~



### push 할것 있을시 pull 하는 방법 

~~~
//2가지 방법 
// merge 방식
git pull -no-rebase

//rebase 방식
git pull --rebase 

push
~~~



### 로컬의 내역 강제 push해보기

~~~
git push --force
~~~

---

## 원격의 브랜치 다루기

### 로컬에서 브랜치 만들어 원격에 push 해보기

~~~
//브랜치 만들기 
git siwtch from-local

//원격의 브랜치 명시 및 기본설정
git push -u origin from-local

// 로컬과 원격의 브랜치들 확인
git branch --all
~~~



### 원격의 브랜치 로컬에 받아오기

~~~
// GitHub에서 from-remote 브랜치 만들기

// 원격의 변경사항 확인
git fetch

// 로컬에 같은 이름의 브랜치를 생성하여 연결하고 switch
git switch -t origin/from-remote
~~~



### 원격의 브랜치 삭제

~~~
git push (원격 이름) --delete (원격의 브랜치명)
~~~



### 다른 원격저장소꺼 가져오기

fork
