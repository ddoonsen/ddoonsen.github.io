# 랭킹 서버 연결하기

랭킹은 여러 사람의 기록을 한곳에 모아야 해서, 기록을 맡아 줄 서버가 하나 필요하다.
무료인 Firebase Realtime Database 를 쓴다. 플레이하는 사람은 로그인할 필요가 없다.

## 1. 데이터베이스 만들기 (한 번만)
1. <https://console.firebase.google.com> 에서 프로젝트를 만든다. (애널리틱스는 꺼도 된다)
2. 왼쪽 메뉴 **빌드 → Realtime Database → 데이터베이스 만들기**. 위치는 아무 곳, 규칙은 "잠금 모드"로 시작.
3. 만들어지면 위쪽에 보이는 주소를 복사한다.
   예: `https://내프로젝트-default-rtdb.firebaseio.com` 또는 `https://내프로젝트-default-rtdb.asia-southeast1.firebasedatabase.app`

## 2. 규칙 넣기
**규칙** 탭의 내용을 아래로 바꾸고 **게시**를 누른다.

```json
{
  "rules": {
    "rank": {
      ".read": true,
      ".indexOn": ["served"],
      "$id": {
        ".write": "newData.exists() && $id.matches(/^[A-Za-z0-9]{16,40}$/)",
        ".validate": "newData.hasChildren(['nick', 'served', 'at'])",
        "nick": { ".validate": "newData.isString() && newData.val().length >= 1 && newData.val().length <= 24" },
        "served": { ".validate": "newData.isNumber() && newData.val() >= 1 && newData.val() <= 10000000 && (!data.exists() || newData.val() >= data.val())" },
        "at": { ".validate": "newData.val() == now" },
        "$other": { ".validate": false }
      }
    }
  }
}
```

- 누구나 순위표를 읽을 수 있고, 기록은 정해진 모양(닉네임, 손님 수, 시각)으로만 쓸 수 있다.
- 손님 수는 줄어들 수 없고, 기록을 지울 수도 없다.

## 3. 게임에 주소 넣기
`index.html` 에서 `const RANK_URL = '';` 를 찾아 복사한 주소를 넣는다. 끝에 `/` 는 붙이지 않는다.

```js
const RANK_URL = 'https://내프로젝트-default-rtdb.firebaseio.com';
```

## 알아 둘 것
- 로그인이 없으므로 기기마다 처음 켤 때 만든 번호(`save.uid`)로 사람을 구분한다.
  브라우저 저장 내용을 지우거나 다른 기기에서 하면 새 사람으로 올라간다.
- 게임이 브라우저 안에서만 돌아가서, 마음먹으면 점수를 속여 올릴 수 있다. 이상한 기록은 Firebase 콘솔의 **데이터** 탭에서 지우면 된다.
- 기록은 1분에 한 번쯤, 그리고 랭킹 창을 열 때와 닉네임을 바꿀 때 올라간다.
