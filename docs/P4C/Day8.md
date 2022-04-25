---
layout: default
title: Day8
nav_order: 1

parent: Develop Project - W2
grand_parent: P4C
---

##### 해당 게시글은 빡공팟 4기(with TeamH4C)와 관련되어 있습니다
-----

# > 인증 페이지 만들기

코드를 작성하기 앞서 내가 생각한 로그인 화면은  메인화면에 로그인 기능이 있는 것이 아닌  
로그인 이후에만 메인 페이지를 확인할 수 있는 폼을 생각했다.

가장 첫 화면에는 로그인 폼이 존재하고,   
아래 회원가입으로 이동가능한 버튼을 만들 예정이다.

## login.html
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>P4CS</title>
    </head>
    <body>
        <header>
            WELCOME to " P4C Shared "
        </header>
        <form  action="login.php" method="post">
            <p>아이디 <input type="text" name="userid" placeholder="Input your ID"></p>
            <p>비밀번호 <input type="password" name="userpw" placeholder="Input your PW"></p>
            <input type="submit" value="로그인">
        </form>
            <p>아직 가입을 안했다면? <a href="signup.html">가입하기<a></p>
    </body>
</html>
```
막상 폼을 만들어 놓고 보니 도저히 디자인을 못봐주겠다는 생각이 들었다.  
그래서 css와 관련하여 조금 찾아보니 class에 대한 개념이 markdown 사용할 때랑 비슷하여  
대충 로그인 홈페이지 구상을 해보았다.

<br>

## login.php
```php
<?php
$uid = $_POST['userid'];
$upw = $_POST['userpw'];
$conn = mysqli_connect("localhost","root","","my_web") or die ("Can't access DB");
$sql = "SELECT userpw FROM userinfo WHERE userid='$uid'";

$result = mysqli_query($conn, $sql);
$row = mysqli_fetch_array($result);

if($row['num_rows'] == 1){
    if($row['userpw'] === $upw){
        echo("로그인 성공!!!");
    }else echo("로그인 실패...");
}else echo("로그인 실패...2");

?>
```
첫 주차에서 php와 mysql에 대해 조금 배웠었지만 두 기능을 연동하여 사용해보지 못했었다.  
그러나 사용자 정보를 코드에 담아둘 수 없기에 데이터베이스 연동 방법이 궁금했었는데  
찾아보니 웹 전용 언어답게 php는 MySQL 연동을 위한 함수가 존재하였다.

`mysql_connect`는 db에 접속하기 위한 함수로 서버, 계정명, 비밀번호, 데이터베이스명 을 인자로 전달받는다. 처음에 내 환경에서 PATH가 Bitnami 폴더로 지정되어 있지 않다는 사실을 잊고 있다가 도저히 접속이 되지 않아 뇌절을 거듭한 끝에 기억해냈다. 

`mysqli_query`는 db에 쿼리 요청을 통해 응답을 받아오는 함수로 `mysql_connect`에 성공하였을 때 반환되는 연결 핸들과 쿼리문을 인자로 전달하면 그에 대한 응답을 반환한다.  
그리고 그 반환값을 배열로 받아주는 것이 `mysqli_fetch_array`인데 해당 함수를 사용하지 않을 경우  
아래의 ①처럼 출력된다. ②는 함수를 이용했을 때의 반환값이다.

![image-20220425174620102](../img/image-20220425174620102.png)

해당 함수를 이용하여 쿼리 결과문을 배열로 변환한 뒤에 쿼리 결과문 갯수를 의미하는 `num_rows`와 쿼리문에서 포함한 컬럼명을 이용하여 조건문의 참과 거짓을 판별하였다.  
정말 가벼운 비밀번호 확인만 하는 코드도 직접 만드려니 조금 어려웠고 생각보다 고려해야 할 점이 많아서 신경써야 할 부분들을 정리 해보기로 했다.

- [ ] 비밀번호 DB 저장 시 비밀번호 암호화
- [ ] 미 로그인 사용자에게만 로그인 페이지 띄우기
- [ ] 가입 시 이메일 인증 이후에 정보란 띄우기
- [ ] 로그인 시 DB 저장된 비밀번호 복호화하여 인증