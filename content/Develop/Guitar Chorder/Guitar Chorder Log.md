---
title: Guitar Chorder 개발일지
---
[[2024-12-11]]

### 문제점 발견
기타 코드표의 손가락 번호를 출력하는 기능을 구현하려 했다. 
개발 중 코드의 효율성, 직관성에 문제가 있다는 것을 파악했다. 
현재 기타 코드 데이터 타입은 다음과 같이 정의되어있다. 

```typescript
type Barre = {

from_string: number; // 바레 시작 줄

to_string: number; // 바레 끝 줄

};

  

type Fret = {

string: number; // 기타 줄 번호

fret: number; // 프렛 번호

finger: number | null; // 손가락 번호 (null은 손가락 사용 안 함)

barre?: Barre; // 바레 코드 정보 (optional)

};

  

type Chord = {

name: string; // 코드 이름

frets: Fret[]; // 운지 정보 배열

description: string; // 코드 설명

omit_strings?: number[]; // 연주하지 않는 줄 배열 (optional)

open_strings?: number[]; // 개방현 배열 (optional)

};

  

type ChordData = {

chords: Chord[]; // 모든 코드 데이터 배열

};
```

지금 당장으로서는 문제 될 것이 전혀 없다. 
위 코드를 바탕으로 작성된 정보로도 아래와 같이 잘 출력된다. 

![[Screenshot 2024-12-11 at 09.33.33.png]]

하지만 사진의 `A` 코드 는 4프렛 까지 출력 될 이유가 없다. 
3개 프렛으로 표현이 충분히 가능한 경우에는 3개 프렛만 출력하도록 기능을 수정하려 한다. 

만약 2번 프렛의 2, 3, 4번 줄이 아니라, **5번 프렛의 2, 3, 4번 프렛을 운지한다면** 어떨까? 
아마 5, 6, 7, 8번 프렛이 출력될 것이다. 
**일반적인 코드 운지표는 4, 5, 6번 프렛을 출력하여 5번 프렛을 가운데에 위치하도록 할 것이다. **

이를 수정하기 위해서는 데이터 구조를 전부 바꿔야 한다. 
어떻게 하려면 할 수야 있겠지만.. 굉장히 비효율적이고 직관적이지 못할 것이라 생각한다. 
향후 유지보수에도 크게 문제가 될 부분이다. 

결국 데이터 구조를 바꾸기로 결정했다. 
데이터 구조를 바꾸면서 하드코딩 되어있던 기타 코드 표의 좌표도 수정하기로 했다. 

```typescript
// 모든 코드 데이터 배열

type ChordData = {

chords: Chord[];

};

// 코드 데이터 타입

type Chord = {

renderingInfo: renderingInfo;

stringInfo: stringInfo;

chordInfo: chordInfo;

};

// 초기 렌더링에 필요한 정보

type renderingInfo = {

usingFret: number; // 사용하는 프렛의 갯수 (렌더링 시 최소한으로 보여줄 프렛 갯수, 3 또는 4로 예상됨)

startingFret: number; // 시작 프렛 번호

};

// open string, omit string 표시 정보 [optional]

type stringInfo = {

openStrings?: number[]; // 개방현 배열

omitStrings?: number[]; // 연주하지 않는 줄 배열

};

// 코드 정보

type chordInfo = {

name: string; // 코드 이름

description: string; // 코드 설명

frets: Fret[]; // 운지 정보 배열

};

// 운지 정보

type Fret = {

string: number; // 기타 줄 번호

fret: number; // 프렛 번호

finger: number; // 손가락 번호

barre?: Barre; // 바레 코드 정보 (optional)

};

// 바레 코드 정보

type Barre = {

from_string: number; // 바레 시작 줄

to_string: number; // 바레 끝 줄

};

}
```

초기 렌더링을 위한 데이터를 추가하였다. 

[[2024-12-13]]

![[Screen Recording 2024-12-13 at 14.20.33.mov]]

기타 코드 생성 방법을 전반적으로 바꾸었다. 
svg 파일의 좌표를 동적으로 수정했다. 
사용자가 원하는 대로 기타 코드의 너비, 폰트 크기 등을 바꿀 수 있다. 

막상 만들고 보니 또 다른 걱정이 생겼다. 
커스텀 할 수 있는 것은 좋지만.. 사용자가 이 기능을 꼭 필요로 할까? 
설정할 수 있는 항목이 너무 많으니 오히려 어려워하지 않을까? 

일단 이 문제는 천천히 생각해보기로 했다. 
실제 서비스에 적용하지 못하더라도 본인 스스로가 수정하기에(유지보수하기에) 편하기 때문이다. 

이때까지 개발한 것들 중에서 몇 가지 버그를 고쳐야 한다. 

- svg 요소의 좌표값 기준이 좌상단이다. 즉, 좌측 최상단을 (0, 0)으로 시작한다. 이 때문에 height 값이 낮아질수록 높은 위치에 표현된다. 
- 손가락 번호를 운지 표시 위에 출력하는 코드가 아직 완성되지 않았다. `<mask>` 태그에 대한 추가적인 학습이 필요하다. 
- 배경색 수정 시 폰트가 깨진다.

![[Screenshot 2024-12-13 at 16.02.26.png]]

손가락 운지 정보를 표시하도록 코드를 수정했다. 
`<mask>` 태그를 사용하여 투명한 숫자 구멍이 생기도록 만들었지만...
기타 프랫과 줄을 나타내는 표를 미처 생각하지 못했다. 
차라리 그냥 색으로 칠하는 것이 더 나을 것 같다. 
그래도 혹시 모르니 일단은 주석처리를 해 두었다. 

[[2024-12-16]]

![[Screenshot 2024-12-16 at 13.16.53.png]]

위의 손가락 번호에 대한 내용을 수정하고 바레코드의 번호 표시 기능을 구현했다. 
코드의 상세한 내용을 수정하는 부분은 추후 유지보수를 위해 남겨두되 사용자들은 이용 불가능하게 해두기로 했다. 
UI를 수정하고 나면 firebase를 통해 배포, 분석을 하려고 한다. 

현재 기능이 
1. 원하는 코드를 입력하면
2. 코드 운지표로 출력해준다. 
딱 두개 밖에 없다. 
UI를 꾸미기에는 너무 기능이 없는가 싶기도 하다. 
하지만 일단 배포하고 사용자들의 반응이 오면 추후에 기능을 점점 추가할 것이다. 
처음부터 많은 기능과 예쁜 UI에 신경쓰기 보다, MVP 개발 후 기능을 추가하는 편이 더 좋을 것이라 생각한다.
힘줘서 열심히 개발했는데 사용자가 0이면 아무런 의미가 없지 않은가. 
핵심적인 기능을 먼저 구현하고 사용자들의 피드백을 받아가며 이에 맞춰 개발할 예정이다. 

[[2024-12-23]]

위의 기능(원하는 코드 출력)을 기반으로 [배포](https://guitarchorder.web.app/)를 마쳤다. 
firebase와 supabase 중 고민했는데, 사용자 분석과 광고 기능을 위해서 firebase를 선택했다. 
지금 당장은 사용자 분석이라 할 것이 없기에(추가 할 이벤트라고는 이미지 다운로드 외에는 없다) analytics 설정은 따로 하지 않았다. 

다음으로 구현 할 기능은 다음과 같다. 

- 가사 검색 기능
- 검색한 가사를 사용해 악보 생성
- Navigation bar 구현: 코드 검색 기능과 악보 생성 기능 분리

가사 검색 기능을 위해 API를 검색해봤는데.. 의외로 찾기 어려웠다. 
몇 개의 비공식 API가 존재하긴 했으나 외국 곡의 검색만 지원했다. 
아무래도 Playwright 등을 이용해 직접 구현해야겠다. 