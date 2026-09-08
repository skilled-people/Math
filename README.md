# 확원 (π 확원)

> 원하는 공부를, 확실하게.

중학교 다섯 과목을 수준별로 골라 개념을 익히고 문제를 푸는 학습 웹앱입니다.
서버·회원가입 없이 브라우저 localStorage에 학습 기록이 저장됩니다.

## 과목과 과정 (5과목 · 21과정 · 47단원 · 188문제)

| 과목 | 과정 | 문제 | 수준 테스트 |
| --- | --- | --- | --- |
| 수학 | 중1-1 기초·응용·심화, 중2-1 기초·응용 (5) | 60 | 12문제 |
| 영어 | 중1-1 기초·응용, 중2-1 기초·응용 (4) | 32 | 8문제 |
| 국어 | 중1-1 기초·응용, 중2-1 기초·응용 (4) | 32 | 8문제 |
| 과학 | 중1-1 기초·응용, 중2-1 기초·응용 (4) | 32 | 8문제 |
| 사회 | 중1-1 기초·응용, 중2-1 기초·응용 (4) | 32 | 8문제 |

## 실행 방법

`index.html` 을 브라우저로 열면 바로 동작합니다. (별도 설치·빌드 없음)

## GitHub Pages에 올릴 때 주의

**폴더째로 올려야 합니다.** `index.html` 하나만 올리면 `css/` 와 `js/` 를
찾지 못해서 글꼴·색·네비게이션이 전부 사라진 맨 HTML로 보입니다.

폴더를 올리기 번거로우면 함께 드린 **단일 파일 index.html** 을 쓰세요.
CSS와 JS가 전부 안에 들어 있어서 그 파일 하나만 올리면 똑같이 동작합니다.

## 파일 구조

```
index.html        홈 (과목 5개)
level-test.html   수준 테스트   ?subject=과목id
courses.html      과목 선택 / 과정 선택   ?subject=과목id
study.html        학습 페이지   ?course=과정id
quiz.html         문제 풀이     ?course=과정id&lesson=단원id
progress.html     학습 기록

css/style.css     전체 스타일 (디자인 토큰 · 반응형)

js/subjects.js    과목 레지스트리 (registerSubject)
js/data-math.js       수학  ← 콘텐츠는 data-*.js 만 수정
js/data-english.js    영어
js/data-korean.js     국어
js/data-science.js    과학
js/data-social.js     사회

js/common.js      localStorage, 진행률·추천 계산, 네비게이션 (공통)
js/main.js        홈
js/courses.js     과목·과정 선택
js/study.js       학습
js/quiz.js        문제 풀이 · 채점
js/progress.js    학습 기록
js/level-test.js  수준 테스트

assets/logo/, assets/icons/
```

## 과목 추가하기

`js/data-*.js` 를 하나 더 만들고 `registerSubject()` 를 호출한 뒤,
각 HTML 의 `<script>` 목록에 한 줄만 추가하면 끝입니다.
홈·과목 선택·문제 메뉴·학습 기록·수준 테스트에 자동으로 나타납니다.

```js
registerSubject({
  id: "history",
  name: "역사",
  mark: "史",              // 과목 배지에 들어가는 글자
  tagline: "짧은 소개",
  desc: "과목 설명",
  formulaLabel: "꼭 외울 것",  // 개념 학습의 '공식' 블록 제목
  courses: [{ id: "hi-m1-basic", title: "중1-1 기초", grade: "…", level: "기초", desc: "…" }],
  lessons: { "hi-m1-basic": [ { id: "l1", title: "…", concept: {…}, questions: […] } ] },
  placement: [ { course: "hi-m1-basic", lesson: "l1", topic: "…", q: "…", choices: [], answer: 0, explanation: "…" } ],
});
```

과정 id 는 과목별 접두사(`en-`, `ko-`, `sc-`, `so-`)를 붙여 겹치지 않게 합니다.

## 문제 추가하기

해당 과목 파일의 `questions` 배열에 객체를 넣으면 됩니다.
문제 수, 진행률, 결과 화면은 모두 자동으로 다시 계산됩니다.

```js
// 객관식
{ type: "choice", q: "...", choices: ["...", "..."], answer: 0, explanation: "..." }

// 단답형 (허용할 표기를 여러 개 쓸 수 있습니다)
{ type: "short", q: "...", answer: ["12", "12개"], explanation: "..." }

// 서술형 (모범 답안을 보여 주고 학생이 스스로 채점)
{ type: "essay", q: "...", model: "...", explanation: "채점 기준" }
```

## 수준 테스트

과목마다 별도의 테스트가 있고, 결과도 과목별로 따로 저장됩니다.
문제마다 `course` 와 `lesson` 이 달려 있어서 **틀린 문제의 단원이 그대로
'먼저 볼 단원' 추천**이 됩니다.

추천 규칙 (`js/common.js` 의 `computePlacement`)

1. 그 과목의 과정별로 정답 비율을 냅니다.
2. 기초 → 응용 → 심화 순서로 훑으면서 정답 비율이 **60% 미만인 첫 과정**을 추천합니다.
3. 그 과정에서 틀린 문제의 단원을 시작 단원으로 잡습니다.
4. 모든 과정이 기준을 넘으면 그 과목의 가장 높은 과정을 추천합니다.

기준을 바꾸려면 `PLACEMENT_PASS` 값만 고치면 됩니다.

## 진행률 계산

단원마다 `개념 학습 완료` 1칸 + `문제 풀이 완료` 1칸으로 계산합니다.
2단원 과정이면 총 4칸이므로 개념 하나를 읽으면 25%가 올라갑니다.
과목 진행률은 그 과목 모든 과정의 칸을 합해서 계산합니다.
