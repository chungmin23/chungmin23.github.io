web-inf-appServlet-servlet-context.xml

~~~
<view-controller path="/register/add" view-name="registerForm"/>
~~~

view 컨트롤러 추가

view-controller에 등록한것은 get요청만 가능

---

RequestMapping 대신 @GetMapping 이나 @PostMapping 으로 짧게 대처가능하다

- 메서드가 다르면 같은 url이어도 충돌이 안난다

---

url 패턴 매핑

? 한글자

*여러글자

** 하위경로 포함

---

url 인코딩

url에 문자열을 변환을 해줄때 사용 

url에 포함된 ascll문자를 문자코드 문자열로 변환한다



