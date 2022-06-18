---
layout: default
title: Day63
nav_order: 3

parent: Hacking Project - W9
grand_parent: P4C
---

##### 해당 게시글은 빡공팟 4기(with TeamH4C)와 관련되어 있습니다

-----

오늘 면접봤는데 이번 11기는 무조건 떨어진 것 같다 ㅋㅋㅋㅋㅋㅋㅋㅋㅋ   
진짜 진짜 아쉽긴 한데 좋은 경험이 였다고 생각하고 다음 지원을 위해서 열심히 해야겠다.  
한 문제라고 풀고 자야 내일의 내가 무기력한 상태로 있지 않을 거라고 믿는다.

# > Webhacking.kr:old-14(js)

## 삽질

```javascript
function ck(){
  var ul=document.URL;
  ul=ul.indexOf(".kr");
  ul=ul*30;
  if(ul==pw.input_pwd.value) { location.href="?"+ul*pw.input_pwd.value; }
  else { alert("Wrong"); }
}
```

자바스크립트에서 처음 보는 함수가 나왔다. `indexOf` 라는 함수는 문자열 뒤에 `.` 기호를 붙여 사용하며, 해당 문자열이 몇 번째 위치에 존재하는지를 반환한다고 한다. 그래서 현재 주소를 적어보면

`https://webhacking.kr/challenge/js-1/` 이고 여기서 ".kr" 문자의 위치는 18번째 이후이다.  그래서 18\*30dls 540을 넣었더니 풀렸다.

![image-20220618211226900](../img/image-20220618211226900.png)

