---
title: "Markdown Exercise"
categories:
  - blogging
tags:
  - markdown
  - language
last_modified_at: 2019-12-31T16:37:00+09:00
toc: true
toc_sticky: true
---

# 마크다운 시작하기

### 문단

여러 개의 빈줄이 있을 경우에도 마크다운에서는 하나의 빈줄로만 인식한다.

### 제목

'#'의 갯수로 제목의 크기를 지정할 수 있다. 6개까지 사용가능하며 '#'뒤에는 공백이 하나 필요하다.

# Heading1
## Heading2
### Heading3
#### Heading4
##### Heading5
###### Heading6

## 인용

'>'를 사용하여 인용문을 나타낸다. 한 문장에 하나의 '>'만 있으면 된다. 여러 문단을 연결하고 싶을 때는 빈줄에도 '>'를 해준다.

> Change your thoughts and you change your world. - Norman Vincent Peale



> Those who realize their folly are not true fools. - Zhuangzi
>
> 
>
> Life is too important to be taken seriously. - Oscar Wilde

### 강조

**굵게하기1** __굵게하기2__ : `**굵게하기1**` `__굵게하기2__`

*기울이기*1 _기울이기2_  : `*기울이기1*` `_기울이기2_`

 **굵은데 *기울여*도 보기** :  ` **굵은데 *기울여*도 보기** `

 **굵은데 _기울여_도 보기**: ` **굵은데 _기울여_도 보기**:`

*기울었는데 **여기만** 굵어요* : `*기울었는데 **여기만** 굵어요*`

*기울었는데 __여기만__ 굵어요* : `*기울었는데 __여기만__ 굵어요*`

'*' 과 '/' 는 같다.

### 취소선

~~취소하기~~ : `~~취소하기~~`

### 목록

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



### TODO list

- [ ] 체크박스 만들기 : `- [ ] 체크박스 만들기` `-Space[Space]Space'Context'` 형식임

- [x] 체크된 박스 만들기 : `- [x] `

 

### 링크

[링크이름](url) : `[링크이름](url)` 

[목록](###목록) : `[목록](###목록)` 이러한 링크도 가능한 듯



![](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png)

`![](https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png)`

`!` 를 붙여서 image url로 이미지를 넣을 수 도 있다.

### 표

| 헤더1   | 헤더2     | 왼정렬 | 중정렬 | 우정렬 |
| ------- | --------- | :----- | :----: | -----: |
| 내용1   | **내용2** | 1      |   2    |      3 |
| _내용3_ | 내용4     | 4      |   5    |      6 |

```
| 헤더1 | 헤더2 |왼정렬|중정렬|우정렬|
|------|----|:-|:-:|-:|
| 내용1 | **내용2** |1|2|3|
| _내용3_ | 내용4 |4|5|6|
```

### 코드 포맷



수식

UML

