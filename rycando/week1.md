# enum을 대체해야하는 이유
## enum은 어렵게 트랜스파일된다
### typescript
``` typescript
enum Player {
    Rudy,
    David,
    Jay 
}
```
### javascript (transpiled)
``` javascript
"use strict";
var Player;
(function (Player) {
    Player[Player["Rudy"] = 0] = "Rudy";
    Player[Player["David"] = 1] = "David";
    Player[Player["Jay"] = 2] = "Jay";
})(Player || (Player = {}));
```
### 문제점
- 디버깅이 어렵다
- tree-shaking이 불가능하다
  - enum은  JS에서 즉시함수로 트랜스파일된다
  - 번들러는 이 코드가 사용되는지 알 수가 없다
  - 냅다 번들에 포함 -> 뚱뚱한 번들 완성
## (당연하게도) enum의 기본타입은 숫자이다
``` typescript
enum Player {
    Rudy,
    David,
    Jay
}

console.log(Player.Rudy) // 0
```
### 문제점
- 디버깅이 어렵다
## enum은 literal 타입추론이 불가능하다
``` typescript
enum Player {
    Rudy = 'rudy',
    David = 'david',
    Jay = 'jay'
}

const player: Player = 'Rudy' // TS error
// Type '"Rudy"' is not assignable to type 'Player'.(2322)

```
## enum은 안전하지않다
### 같은 이름의 enum은 합쳐진다
``` typescript
enum Player {
    Rudy = 'rudy',
    David = 'david'
}

enum Player {
    Jay = 'jay'
}

console.log(Player.Rudy, Player.Jay) // rudy, jay
```
### value는 같고 key는 다른면 이상한 결과가 나온다
``` typescript
enum NumberName {
    One = 1,
    TWO = 2,
}

enum Player {
    THREE = 1
}

console.log(Player[1]) // THREE
```

## enum의 number 타입에러 (TS < 5.x.x)
``` typescript
enum NumberName {
    ONE =  1,
    TWO = 2,
    THREE = 3
}

const hundred: NumberName = 100 // no error
```
![](https://velog.velcdn.com/images/rycando/post/e85ff7f2-c48d-4644-ac4f-cc7d48375468/image.jpeg)

# enum을 대체하는 방법
## const enum
### 사용 전 - typescript
``` typescript
const enum Player {
    Rudy = 'rudy',
    David = 'daivd',
    Jay = 'jay'
}
```
### 사용 전 - javascript (transpiled)
```
"use strict";
```

### 사용시점 - typesciprt
``` typescript
const enum Player {
    Rudy = 'rudy',
    David = 'daivd',
    Jay = 'jay'
}

const player: Player = Player.Rudy
```
### 사용시점 - javascript (transpiled)
``` javascript
"use strict";
const player = "rudy" /* Player.Rudy */;
```
- const enum은 사용시점에만 트랜스파일된다
### 하지만 이런경우라면...?
- typescript
``` typescript
const enum Player {
    Rudy = 'rudy',
    Dvaid = 'daivd',
    Jay = 'jay',
    Kim = "김수한무 거북이와 두루미 삼천갑자 동방삭 치치카포 사리사리센타 워리워리 세브리깡 무두셀라 구름이 허리케인에 담벼락 담벼락에 서생원 서생원에 고양이 고양이엔 바둑이 바둑이는 돌돌이 "
}

const player: Player = Player.Rudy
```
- javascript (transpiled)
``` javascript
"use strict";
const player = "\uAE40\uC218\uD55C\uBB34 \uAC70\uBD81\uC774\uC640 \uB450\uB8E8\uBBF8 \uC0BC\uCC9C\uAC11\uC790 \uB3D9\uBC29\uC0AD \uCE58\uCE58\uCE74\uD3EC \uC0AC\uB9AC\uC0AC\uB9AC\uC13C\uD0C0 \uC6CC\uB9AC\uC6CC\uB9AC \uC138\uBE0C\uB9AC\uAE61 \uBB34\uB450\uC140\uB77C \uAD6C\uB984\uC774 \uD5C8\uB9AC\uCF00\uC778\uC5D0 \uB2F4\uBCBC\uB77D \uB2F4\uBCBC\uB77D\uC5D0 \uC11C\uC0DD\uC6D0 \uC11C\uC0DD\uC6D0\uC5D0 \uACE0\uC591\uC774 \uACE0\uC591\uC774\uC5D4 \uBC14\uB451\uC774 \uBC14\uB451\uC774\uB294 \uB3CC\uB3CC\uC774 " /* Player.Kim */
```
- 트랜스파일된 코드가 너무 더러워진다
## string literal Union
### typescript
``` typescript
type Player = 'rudy' | 'david' | 'jay'

const player: Player = 'rudy'
```
### javascript (transpiled)
``` javascript
"use strict";
const player = 'rudy';
```
- 마찬가지로 트랜스파일 된 코드가 더러워질 수 있다
## 배열 이용하기
``` typescript
const playerArr = ['rudy', 'david', 'jay'] as const
type Player = typeof playerArr[number] // 'rudy', 'david', 'jay'

const player: Player = 'jay'
```
## 객체 이용하기
``` typescript
const playerObj = {
    rudy: 'rudy',
    david: 'david',
    jay: 'jay'
} as const
type Player = typeof playerObj[keyof typeof playerObj] // 'rudy' | 'david' | 'jay'

const player: Player = playerObj.rudy
```
# 리터럴로 타입 만들어 활용하기
``` typescript
type Union<T extends ReadonlyArray<any> | Record<string, any>> = T[keyof T]

const playerObj = {
    rudy: 'rudy',
    david: 'david',
    jay: 'jay'
} as const

const player: Union<typeof playerObj> = playerObj.rudy
```

# 참고자료
- https://maxheiber.medium.com/alternatives-to-typescript-enums-50e4c16600b1
- https://engineering.linecorp.com/ko/blog/typescript-enum-tree-shaking
)
