---
title: "마크다운 시작하기"
categories:
  - blogging
  - human
tags:
  - markdown
  - language
last_modified_at: 2020-01-02T00:00:00+09:00
toc: true
toc_sticky: true
---

## 문단

여러 개의 빈줄이 있을 경우에도 마크다운에서는 하나의 빈줄로만 인식한다.

## 제목

'#'의 갯수로 제목의 크기를 지정할 수 있다. 6개까지 사용가능하며 '#'뒤에는 공백이 하나 필요하다.

```markdown
# Heading1
## Heading2
### Heading3
#### Heading4
##### Heading5
###### Heading6
```

## 인용

'>'를 사용하여 인용문을 나타낸다. 한 문장에 하나의 '>'만 있으면 된다. 여러 문단을 연결하고 싶을 때는 빈줄에도 '>'를 해준다.

> Change your thoughts and you change your world. - Norman Vincent Peale



> Those who realize their folly are not true fools. - Zhuangzi
>
> 
>
> Life is too important to be taken seriously. - Oscar Wilde

## 강조

**굵게하기1** __굵게하기2__ : `**굵게하기1**` `__굵게하기2__`

*기울이기*1 _기울이기2_  : `*기울이기1*` `_기울이기2_`

 **굵은데 *기울여*도 보기** :  ` **굵은데 *기울여*도 보기** `

 **굵은데 _기울여_도 보기**: ` **굵은데 _기울여_도 보기**:`

*기울었는데 **여기만** 굵어요* : `*기울었는데 **여기만** 굵어요*`

*기울었는데 __여기만__ 굵어요* : `*기울었는데 __여기만__ 굵어요*`

<u>밑줄</u> : `<u>밑줄</u>` HTML 밖에 못 찾겠음

'*' 과 '/' 는 같다.

## 취소선

~~취소하기~~ : `~~취소하기~~`

## 목록

* 순서없는 목록1 : `* 순서없는 목록1 `

- 순서없는 목록2 : `- 순서없는 목록1 `

'*'과 '-'의 역할이 같다.

1. 첫번째
   1. TAB으로 들여서 쓰기
      1. 얼마나
         - 순서없이도 가능
           - 합니다.
      2. 많이
         1. 할 수 있을까?
            1. 무한대라고 함
      3. SHIFT TAB 으로 내어쓰기도 할 수 있음

## TODO list

- [ ] 체크박스 만들기 : `- [ ] 체크박스 만들기` `-Space[Space]Space'Context'` 형식임

- [x] 체크된 박스 만들기 : `- [x] `

## 링크

https://www.example.com : `https://www.example.com` 링크 적용이 안됨

<https://www.example.com> : `<https://www.example.com>`

[링크이름](url) : `[링크이름](url)` 

[첫번째 헤드1](#마크다운-시작하기) : `[첫번째 헤드1](#마크다운-시작하기)` 이러한 링크도 가능하다

[세번째 헤드2](#인용) : `[세번째 헤드2](#인용)` 이러한 링크도 가능하다

![google](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png)

`![google](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png)`
`!` 를 붙여서 image url로 이미지를 넣을 수 도 있다.
`[google]` 부분에 글을 넣어 alter text를 설정할 수 있다.

![google](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png){: width="20%" height="20%"}

`![](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png){: width="20%" height="20%"}`
화면에 나타내는 이미지 사이즈를 조절할 수 있다.
`%`를 지우면 `px` 단위로 인식한다. `px`를 직접 명시해도된다.

[reference-link][ref] : `[reference-link][ref]` reference 형식의 link를 설정한다. 어떤 문자나 문자열을 사용할 수 있지만 소문자만 사용 가능하다.
`[ref]: http://chae-yc.github.io` 이렇게 reference 할 link를 설정해 주어야 한다.

[ref]: http://chae-yc.github.io

`<a name="id-value"></a>` Anchor tag 를 삽입하여
`[go](#id-value)` 와 같이 사용할 수 도 있다.

## 표

| 헤더1   | 헤더2     | 왼정렬 | 중정렬 | 우정렬 |
| ------- | --------- | :----- | :----: | -----: |
| 내용1   | **내용2** | 1      |   2    |      3 |
| _내용3_ | 내용4     | 4      |   5    |      6 |

```markdown
| 헤더1 | 헤더2 |왼정렬|중정렬|우정렬|
|------|----|:-|:-:|-:|
| 내용1 | **내용2** |1|2|3|
| _내용3_ | 내용4 |4|5|6|
```

## Code

### In-line

`` ` ``를 글의 양 옆에 묶으면 한 줄 단위의 코드 를 작성할 수 있다.

코드 내부에는 마크다운 문법이 적용되지 않고 plain text로 보이게된다.
아래의 코드를 보면 굵기와 기울기가 적용되지 않고 있다.

`code **code** __code__ _code_ *code*`

`` ` code **code** __code__ _code_ *code* ` `` 이렇게 작성하면 된다.

### Multi-line

멀티라인 코드를 작성하고 싶으면 '```'
혹은 `~~~` 를 코드 블록 시작과 끝에 사용한다.

```javascript
let cnt = 0;
console.log("count: "+ cnt);
```

~~~
```javascript
let cnt = 0;
console.log("count: "+ cnt);
```
~~~

'```'와 같이 여러 줄의 코드를 작성할 시 language를 지정하여 syntax highlighing 기능을 사용 할 수 있다.
위에 작성한 예시에는 javascript를 사용한다고 지정해주었다.

### Language list
다음은 지원하는 language의 목록이다.
- actionscript3
- apache
- applescript
- asp
- brainfuck
- c
- cfm
- clojure
- cmake
- coffee-script, coffeescript, coffee
- cpp - C++
- cs
- csharp
- css
- csv
- bash
- diff
- elixir
- erb - HTML + Embedded Ruby
- go
- haml
- http
- java
- javascript
- json
- jsx
- less
- lolcode
- make - Makefile
- markdown
- matlab
- nginx
- objectivec
- pascal
- PHP
- Perl
- python
- profile - python profiler output
- rust
- salt, saltstate - Salt
- shell, sh, zsh, bash - Shell scripting
- sql
- scss
- sql
- svg
- swift
- rb, jruby, ruby - Ruby
- smalltalk
- vim, viml - Vim Script
- volt
- vhdl
- vue
- xml - XML and also used for HTML with inline CSS and Javascript
- yaml

## 수식, UML

당분간은 사용할 일이 없어서 추후에 필요할 때 정리하도록 할 것임
