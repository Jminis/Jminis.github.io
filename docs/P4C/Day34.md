---
layout: default
title: Day34
nav_order: 4

parent: Hacking Project - W5
grand_parent: P4C
---

##### 해당 게시글은 빡공팟 4기(with TeamH4C)와 관련되어 있습니다
-----
드디어 5/20 서울 자취방에 입성했다.  
진짜 하루 종일 입주 청소를 하고 뻗어버렸다.  
아직 정리할게 산더미이지만 공부는 해야지;

# > DVWA: File upload

![image-20220521102908764](../img/image-20220521102908764.png)



## 삽질

```php
<?php

if( isset( $_POST[ 'Upload' ] ) ) {
    // Where are we going to be writing to?
    $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
    $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );

    // Can we move the file to the upload folder?
    if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
        // No
        echo '<pre>Your image was not uploaded.</pre>';
    }
    else {
        // Yes!
        echo "<pre>{$target_path} succesfully uploaded!</pre>";
    }
}

?> 
```
이전에는 들어보기만 했던 "파일 업로드 취약점"을 풀어볼 기회가 생겼다. 이전과는 다르게 php 코드를 무서워하지 않기 때문에 업로드 하려는 함수가 익숙하게 느껴진다. 내가 만들 때에도 위 코드처럼 `move_uploaded_file` 함수를 썼었는데 여기에서는 `uploaded`가 속성이름으로 설정되어 있는 듯하다. 그러나 내 기억으로는 업로드에 성공할 경우에 1을 반환하는 것으로 알고 있다. 문제에서는 업로드에 실패할 경우에 성공적으로 업로드 되었다고 하는 것을 보니 주어진 `~/hackable/uploads` 경로가 아닌 다른 곳에 업로드하라는 것 같다.

아마도 이전 디렉토리를 의미하는 `../` 기호를 활용하면 되지 않을까? 라고 생각하여 일반적인 업로드와 내가 생각한 업로드 방식을 실험해보기로 했다. 일반적인 이미지를 업로드할 경우에 다음과 같다.

![image-20220521103558090](../img/image-20220521103558090.png)

그래서 코드에 따라 업로드가 불가능하다고 판단되어 파일 이름에 경로 기호를 삽입하려고 하였다. 하지만 미처 파일 이름이 `.` 기호와 `/` 기호가 포함되도록 만드는 방법을 알지 못했다. 그래서 burp suite 를 이용해 인자 전달 시에 파일 명을 바꿔 보기로 했다. 

![image-20220521110540906](../img/image-20220521110540906.png)

근데 그래도 안됐다. 무언가 잘못됨을 느끼고 php 오류를 출력하게끔 하였더니

![image-20220521110632630](../img/image-20220521110632630.png)

권한이 없다고 한다. 아마도 bitnami를 구축할 때 설정된 폴더 권한 때문에 그런 것 같다.

<br>

![image-20220521111323305](../img/image-20220521111323305.png)

확인해보니 upload 폴더의 권한이 666이였고 내가 업로드 기능을 구현할 때 업로드 폴더에 주었던 권한 777을 부여했다. 내가 알기로는 실행 권한인 `x`가 폴더에 없을 경우 해당 폴더에 접근이 불가능한 것으로 알고 있는데, 이게 왜 이렇게 되어있었는지 잘 모르겠다. 일단 777 권한을 부여한 이후에는 php 오류 구문 없이 파일이 업로드 가능했다.

![image-20220521111639889](../img/image-20220521111639889.png)

<br><br>

이 문제를 해결하고 나니 코드 부분이 이해가 가지 않았다. 인터넷을 찾아보니 `move_uploaded_file`은 성공할 경우 true를 반환하는 것으로 나와있다. 그러나 위 코드에서 반환이 false일 경우에 업로드 문구가 뜨도록 되어있는데... 뭘까...

<br><br>

## Write up

사소한 업로드 이슈를 해결한 뒤 접근한 방식은 다음과 같다.

1. 문제에서 업로드에 성공할 경우에 업로드 된 경로를 알려준다.
2. 알려진 경로에 대한 접근이 가능하다.

![image-20220521113519206](../img/image-20220521113519206.png)

때문에 system("ls") 와 같은 코드를 실행한다면 되겠다.

```php
<?php
system("ls");
?>
```

이 코드를 삽입하여 올려도 좋겠지만 더 다양한 커맨드를 위해 아래 코드를 업로드 했다.

```php
<?php
$cmd = $_GET['c'];
system($cmd);
?>
```

그리고 언제든지 해당 경로에서 c를 통해 커맨드를 전달하면 수행가능해졌다.

![image-20220521113910279](../img/image-20220521113910279.png)

<br><br>

이 라업을 쓰다가 위 웹쉘 코드 때문에 V3랑 싸웠다. 후.

![image-20220521114134525](../img/image-20220521114134525.png)

<br><br><br>

-----

# > DVWA: Insecure recaptcha

일단 이 문제를 풀기 위한 기본적인 세팅으로 DVWA에 reCAPTCHA를 설정 해주어야 한다. 구글 로그인 이후 설정이 가능하며 공개키와 비밀키를 발급 받을 수 있다. 이후 DVWA의 `config.inc.php`에서 추가해주면 된다. 아 도메인은 `localhost`로 하였다.

![image-20220521115914021](../img/image-20220521115914021.png)

캡쳐가 짤렸지만 해당 부분 아래에 2개의 키를 발급받는다.

```
$ sudo vi /opt/lampstack-8.1.4-0/apache2/htdocs/DVWA/config/config.inc.php
```
이번 과제는 bitnami와 함께하기에 다른 사람들과 설정 위치가 다르다. 사서 고생 중이다.

<div class="code-example">
# ReCAPTCHA settings<br>
#   Used for the 'Insecure CAPTCHA' module<br>
#   You'll need to generate your own keys at: https://www.google.com/recaptcha/admin<br>
$_DVWA[ 'recaptcha_public_key' ]  = '여기에 공개키';<br>
$_DVWA[ 'recaptcha_private_key' ] = '여기에 비밀키';<br>
</div>

해당 작업 이후에 확인해보면 이제야 풀 수 있겠다.

![image-20220521120509391](../img/image-20220521120509391.png)

<br><br>

## 삽질

```php
<?php

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) {
    // Hide the CAPTCHA form
    $hide_form = true;

    // Get input
    $pass_new  = $_POST[ 'password_new' ];
    $pass_conf = $_POST[ 'password_conf' ];

    // Check CAPTCHA from 3rd party
    $resp = recaptcha_check_answer(
        $_DVWA[ 'recaptcha_private_key'],
        $_POST['g-recaptcha-response']
    );

    // Did the CAPTCHA fail?
    if( !$resp ) {
        // What happens when the CAPTCHA was entered incorrectly
        $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
        $hide_form = false;
        return;
    }
    else {
        // CAPTCHA was correct. Do both new passwords match?
        if( $pass_new == $pass_conf ) {
            // Show next stage for the user
            echo "
                <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre>
                <form action=\"#\" method=\"POST\">
                    <input type=\"hidden\" name=\"step\" value=\"2\" />
                    <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" />
                    <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" />
                    <input type=\"submit\" name=\"Change\" value=\"Change\" />
                </form>";
        }
        else {
            // Both new passwords do not match.
            $html     .= "<pre>Both passwords must match.</pre>";
            $hide_form = false;
        }
    }
}

if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) {
    // Hide the CAPTCHA form
    $hide_form = true;

    // Get input
    $pass_new  = $_POST[ 'password_new' ];
    $pass_conf = $_POST[ 'password_conf' ];

    // Check to see if both password match
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the end user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with the passwords matching
        echo "<pre>Passwords did not match.</pre>";
        $hide_form = false;
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

사실 문제 이름 자체가 생소하기에 일단 무엇이든지 시도 해보기로 했다.
가장 의심한 코드는 이 부분이다.

```php
$resp = recaptcha_check_answer(
    $_DVWA[ 'recaptcha_private_key'],
    $_POST['g-recaptcha-response']
);
```
reCAPTCAH의 응답을 POST 값으로 넘겨받는다. 통과하지 않아도 조작한다면...?  
먼저 reCAPTCHA를 하고 넘어갔을 때이다.

![image-20220521121706927](../img/image-20220521121706927.png)

step 1이라는 인자와 reCAPTCHA의 응답 값이 존재하고 이를 전달한다. 그리고 나서 아래 페이지로 이동한 다음에

![image-20220521121752943](../img/image-20220521121752943.png)

![image-20220521121908972](../img/image-20220521121908972.png)

다시 change 버튼을 누르면 step 2 인자가 포함된 패킷 송신을 볼 수가 있었는데 최종적으로 해당 통신이 비밀번호를 변경하는 듯하다. 그리고 가설에 따른 시도이다.

![image-20220521122217906](../img/image-20220521122217906.png)

![image-20220521122201051](../img/image-20220521122201051.png)

이렇게 전달되려는 패킷에 1이라는 값을 한번 넣어보았다.

![image-20220521122321461](../img/image-20220521122321461.png)

응 안된다. 되겠냐고.

<br><br>

## Write up

문제에서 reCAPTCHA를 이용한 통신은 다음과 같은 과정을 거친다.

1. reCAPTCHA를 진행하여 step1 패킷을 보내는데 이 때 난수를 이용해 참/거짓을 판별한다.
2. 성실하게 통과했을 경우 step2 패킷을 보낼 수 있게 되며 비밀번호가 변경된다.

이 때 step2에서 비밀번호의 정보도 함께 넘어가는 것을 포착할 수 있는데 해당 패킷의 비밀번호만 수정하여 재요청 해보자. 동일한 요청에 대한 검증과 유효 검사가 없기에 전달 가능하다.

reCAPTCHA 통과 이후에 전달했던 패킷을 붙잡아 두고 "Send to Repeater"를 이용해 다시 전달해보자.

![image-20220521123122313](../img/image-20220521123122313.png)

burp suite에서 패킷의 내용을 수정했고, 'aaaa'로 변경하였던 요청을 '1234'로 변경하는 요청으로 다시 전달했다.

![image-20220521123032504](../img/image-20220521123032504.png)

위 요청/응답으로 비밀번호가 '1234'로 바뀐 것을 확인할 수 있다.

<br><br><br>

-----


# > DVWA: SQL injection





