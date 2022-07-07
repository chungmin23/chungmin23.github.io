# css

웹어플리케이션 구조에 디자인을 추가해준다



---

### css 셀렉터

~~~
//클래스 .  , id # 


//셀럭터
h1 {}
div {}

//전체 셀럭터
* {}

// Tag 셀럭터
section, h1 {}

//ID 셀렉터
#only {}

//class 셀럭터 
.widget {}
.center {}

//attribute 셀럭터
// 동일한 속성이 있는것 
p[id="only"] { }


//후손 셀럭터
// header 아래 모든 h1 선택 
header h1 {}

// 자식 셀럭터
// header 바로 아래 p 태그만 선택 
header > p {}

// 인접 형제 셀럭터 
// 형제 셀럭터는 같은 선상에 있는 태그를 말한다 
// section 바로 근처에 p태그 하나만 선택 
setion + p {}

// 형제 셀럭터
// section 태그 밑에 p태그 모두 선택 
section ~ p {}


// 구조 가상 선택자 
:nth-child(N) = 부모안에 모든 요소중에 N번째 요소 
A:nth-of-type(N) = A요소중에 N번째 숫자 선택 

~~~

