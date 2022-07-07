# Linux

### 리눅스 명령어

~~~
//현재 위치 확인하기
pwd 

//새로운 폴더 생성
mkdir

//파일 폴더 확인 
ls

//폴더에 진입
cd 

//파일 생성
touch hi.txt

//실행 결과 파일로 저장하기 
>
echo dlcndals2005@naver.com > hi.txt

//파일의 내용을 터미널에 출력하기
cat hi.txt

//파일 삭제 
rm 
rm -rf
// r은 폴더를 지울때 사용, f는 질문을 받지 않고 지울때 사용

//폴더나 파일의 이름을 변경, 폴더나 파일의 위치 옮기기
mv
//폴더 옮기기
mv bye.txt bye/

//파일 폴더 이름 변경
mv bye.txt helloworld.txt

//폴더나 파일을 복사하기 
cp 
cp hellworld.txt hiComputer.txt

// 관리자 권한 획득
sudo 
~~~

---

### 관리자 권한

악성코드로 부터 보호하기 위해서 사용자와 관리자 권한을 두어서 관리를 한다 

---

### 패키지 매니저

~~~
//우분투 패키지 매니저 
apt 

// 관리자 권한 사용하기
sudo 
sudo apt intall wget 


//패키지 목록 갱신
// 패키지를 다운로드 할수 있는 여러 저장소의 최신 정보를 업데이트 
apt update (관리자 권한 필요)

// 업그레이드 가능한 패키지 목록 출력 
apt list --upgradable 

//전체 패키지 업그레이드
apt upgrade (관리자 권한 필요)

//특정 패키지만 업그레이드
apt --only-upgrade install 패키지 이름 (관리자 권한 필요)

//패키지 설치
apt install 패키지 이름 (관리자 권한 필요)

//설치된 패키지 보기
apt list --installed

//패키지 검색
apt search 검색어

//패키지 정보 확인
apt show 패키지 이름

//패키지 삭제
apt remove 패키지 이름 (관리자 권한 필요)

~~~



**wget**

:URL을 통해 파일을 다운로드 하는 프로그램

~~~
 wget -O goodjob.txt https://bit.ly/37sJqCo
~~~

---

### Read , Write, Execute 권한

user : 파일의 소유자 

group : 여러 유적가 포함되어 있는 그룹

other : 파일을 만들지 않은 다른 모든  user



~~~
//권항르 변경하는 명령어
chmod
chmod g-r filename
chmod 755 filename
~~~



| Premission | Number |
| ---------- | ------ |
| Read       | 4      |
| Write      | 2      |
| Execute    | 1      |

---

### 환경변수

내가 만든 프로그램의 폴더 위치를 다른 컴퓨터에서 실행했을때 위치가 달라진다 . 그래서 환경변수를 지정해주어서 다른 컴퓨터에서도 어느 위치에 있지는 알려준다