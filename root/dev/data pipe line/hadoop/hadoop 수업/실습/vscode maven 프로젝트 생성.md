---
tags:
  - vscode
  - maven
create: 2024-06-26 13:14:51
---

https://hyunchang88.tistory.com/309

## 프로젝트

![[Pasted image 20240626132635.png]]

0. 입력: ctrl + shift + p
1. 생성: 디렉토리
	- 규칙: [스네이크 표기법](https://heytech.tistory.com/294)을 쓸 것
2. 선택: java: create java project

---
## Build Tools

![[Pasted image 20240626132344.png]]
- 선택: Maven create from archetype

---
## Archtype

![[Pasted image 20240626132409.png]]
- 선택: maven-archetype-quickstart
---
## version


![[Pasted image 20240626132434.png]]
- 선택: 1.4
---
## package

![[Pasted image 20240626132453.png]]
- 입력: com.example(기본값)

---
## Project name

![[Pasted image 20240626132656.png]]
- 입력: my_project(스네이크 표기법 사용)

---
## 경로

![[Pasted image 20240626132819.png]]
- [프로젝트](##프로젝트) 에서 만든 디렉토리 사용

---

## 마무리

![[Pasted image 20240626132932.png]]

1. 입력: 엔터
2. 입력: Y
3. 입력: 아무 키

---


## 디렉토리 구조

### 기본

![[Pasted image 20240626133026.png]]

`````ad-attention
마지막에 정하라고 하는데 그 때는 프로젝트 이름과 같은
디렉토리를 만들어서 해주자

```
.
`-- my_project
    `-- my_project
```

`````

### 전체

![[Pasted image 20240626133045.png]]
```
.
`-- my_project
    `-- my_project
        |-- pom.xml
        |-- src
        |   |-- main
        |   |   `-- java
        |   |       `-- com
        |   |           `-- example
        |   |               `-- App.java
        |   `-- test
        |       `-- java
        |           `-- com
        |               `-- example
        |                   `-- AppTest.java
        `-- target
            |-- classes
            |   `-- com
            |       `-- example
            |           `-- App.class
            `-- test-classes
                `-- com
                    `-- example
                        `-- AppTest.class
```

