---
title : "[IntelliJ] 인텔리제이 추천 단축키"

categories :

- IntelliJ

tags :

- IntelliJ
---

해당 글은 Mac 전용 단축키들입니다. Windows를 사용하시는 분들은 해당 단축키의 이름을 Keymap에서 검색해서 사용해주세요.

이 글에서 캐럿은 문자 커서를 의미합니다.

문자 커서 참조

[https://ko.wikipedia.org/wiki/커서_(사용자_인터페이스)](https://ko.wikipedia.org/wiki/%EC%BB%A4%EC%84%9C_(%EC%82%AC%EC%9A%A9%EC%9E%90_%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4))

## 1. Editor Actions
- Delete Line [`Command + Backspace`] : 캐럿이 위치한 줄 제거
- Duplicate Line or Selection [`Command + D`] : 캐럿이 위치한 줄 혹은 블럭을 한 줄 아래에 복제

<img width="495" alt="1" src="https://user-images.githubusercontent.com/56301069/105535044-b5cfb080-5d31-11eb-8f84-963cb13464c1.png">

위 사진과 같이 복제하고 싶은 라인에 캐럿을 위치한 후 `Command + D` 를 입력하면 다음 사진과 같이 라인이 복제된다.

<img width="490" alt="2" src="https://user-images.githubusercontent.com/56301069/105534931-8d47b680-5d31-11eb-9507-0a442cf7c386.png">

- Move Caret to Line End [`Command + ⭢`] : 캐럿을 줄의  끝으로 이동
- Move Caret to Line End with Selection [`Shift + Command + ⭢`] : 캐럿부터 줄의 끝까지 블록
- Move Caret to Line Start [`Command + ⭠`] : 캐럿을 줄의 시작 지점으로 이동
- Move Caret to Line Start with Selection [`Shift + Command + ⭠`] : 줄의 시작부터 캐럿까지 블록
- Move Caret to Next Word [`Option + ⭢`] : 다음 단어로 캐럿을 이동
- Move Caret to Next Word with Selection [`Shfit + Option + ⭢`] : 캐럿부터 다음 단어까지 블록
- Move Caret to Previous Word [`Option + ⭠`] : 이전 단어로 캐럿을 이동
- Move Caret to Previous Word with Selection [`Shfit + Option + ⭠`] : 이전 단어부터 캐럿까지 블록
- Extend Selection [`Option + ⭡`] : 선택 확장
- Shrink Selection [`Option + ⭣`] : 선택 축소

## 2. Main Menu

### 2-1. File

- Preferences [`Command + ,`] : 설정

### 2-2. Edit

- Cut [`Command + X`] : 잘라내기
- Copy [`Commad + C`] : 복사
- Paste [`Command + V`] : 붙여넣기
- Past as Plaint Tesxt [`Shift + Option + Command + V`] : 일반 텍스트로 붙여넣기
- Find [`Command + F`] : 찾기
- Replace [`Command + R`] : 바꿔쓰기
- Find Usages [`Option + F7`] : 사용되는 곳 찾기
- Indent Selection [`Tab`] : 들여쓰기
- Unindent Line or Selection [`Shift + Tab`] : 들여쓰기 제거

### 2-3. View

- Quick Definition [`Command + Y`] : 클래스 구성을 빠르게 확인
- Parameter Info [`Command + P`]  : 메소드의 파라미터 정보를 보여줌

다음 사진과 같이 파라미터를 입력하는 곳에 캐럿을 두고 `Command + P` 를 입력하면 어떤 파라미터가 와야하는지 보여준다.

<img width="466" alt="스크린샷 2021-01-23 오전 2 33 10" src="https://user-images.githubusercontent.com/56301069/105535066-bb2cfb00-5d31-11eb-8d41-5bb79d5e69cb.png">


- Recent Files [`Command + E`] : 최근 사용한 파일 목록

### 2-4. Navigate

- Next Highlited Error [`F2`] : 경고 위치로 이동
- Next Method [`Ctrl + ⭣`] : 다음 메소드로 이동
- Previous Method [`Ctrl + ⭡`] : 이전 메소드로 이동
- Go to Test [`Shift + Command + T`] : 관련 테스트 코드로 이동
- File Structure [`Command + F12`] : 해당 파일 구조를 보여줌

### 2-5. Code

- Generate [`Command + N`] : 생성자, Getter/Setter 등 여러 가지 생성 가능
- Code Completion - SmartType [`Shift + Control + Space`] : 자동 완성
- Surround With.. [`Option + Command + T`] : 해당 라인을 감쌀 수 있는 여러 목록을 보여줌

원하는 라인에 캐럿을 둔 후 `Option + Command + T` 를 입력하면 사진과 같이 여러 목록들이 출력된다. 사진에선 mainController에 캐럿을 두었다.

<img width="519" alt="스크린샷 2021-01-23 오전 3 00 48" src="https://user-images.githubusercontent.com/56301069/105535067-bb2cfb00-5d31-11eb-8e95-66ff17389f89.png">

6번 try/catch를 선택하면 다음과 같이 해당 라인이 try/catch 문으로 감싸진다.

<img width="501" alt="스크린샷 2021-01-23 오전 3 02 58" src="https://user-images.githubusercontent.com/56301069/105535071-bbc59180-5d31-11eb-9cae-425d184737ab.png">

- Comment with Line Comment [`Command + /`] : 한 줄 주석
- Comment with Block Comment [`Option + Command + /` 혹은 `Shift + Command + /`] : 블록한 라인 전체 주석
- Reformat File [`Shift + Option + Command + L`] : 코드를 정리
- Move Statement Down [`Shift + Command + ⭣`] : 해당 줄과 아랫줄을 교체

위의 사진에서 mainController.run() 에 캐럿을 둔 후 `Shift + Command + ⭣` 를 입력하면 다음 사진과 같이 된다.

<img width="504" alt="스크린샷 2021-01-23 오전 3 10 29" src="https://user-images.githubusercontent.com/56301069/105535075-bc5e2800-5d31-11eb-9e47-a480870c56a2.png">

- Move Statement Up [`Shift + Command + ⭡`] : 해당 줄과 윗줄을 교체

### 2-6. Refactor

- Refactor This.. [`Contrl + T`]  : 리팩토링 기능 목록
- Rename... [`Shift + F6`] : 이름 변경, 해당 파일에서만이 아니라 사용하는 모든 곳에서 이름이 변경된다

- Introduce Variable... [`Option + Command + V`] : 해당 변수를 지역 변수로 변경
- Introduce Constant... [`Option + Command + C`] : 해당 변수를 상수로 변경
- Introduce Field... [`Option + Command + F`] : 해당 변수를 필드로 변경
- Introduce Parameter... [`Option + Command + P`] : 해당 변수를 파라미터로 변경

다음과 같이 1을 출력하는 프로그램이 있습니다.

<img width="396" alt="스크린샷 2021-01-23 오전 3 30 21" src="https://user-images.githubusercontent.com/56301069/105535077-bcf6be80-5d31-11eb-92ee-01d928fc9a87.png">

- Introduce Variable 을 하면 1을 지역 변수로 변경합니다.

<img width="395" alt="스크린샷 2021-01-23 오전 3 32 04" src="https://user-images.githubusercontent.com/56301069/105535078-bcf6be80-5d31-11eb-932a-49da61bcd514.png">


- Introduce Constant 을 하면 1을 상수로 변경합니다.

<img width="383" alt="스크린샷 2021-01-23 오전 3 32 59" src="https://user-images.githubusercontent.com/56301069/105535081-bd8f5500-5d31-11eb-8ffd-1f71bd754210.png">


- Introduce Parameter 을 하면 1을 파라미터로 변경합니다.

<img width="446" alt="스크린샷 2021-01-23 오전 3 33 56" src="https://user-images.githubusercontent.com/56301069/105535083-be27eb80-5d31-11eb-9872-5957231e6439.png">


### 2-7. Run

- Run [`Control + R`] : 실행
- Run context configuration [`Shift + Control + R`] : 현재 파일의 main 메소드 실행
- Stop [`Command + F2`] : 중지

### 2-8. Git

- Commit [`Command + K`] : 커밋
- Push [`Shift + Command + K`] : 푸시

### 2-9. Window

- Hide All Tool Windows [`Shift + Command + F12`] : 모든 도구 숨김
- Split Right [`단축키 지정 필요`] : 세로로 창 분할
- Unspli All [`단축키 지정 필요`] : 모든 창 통합

### 2-10. Help

- Find Action... [`Shift + Command + A`] : 도구 모음

## 3. Others

- Show Context Actions [`Option + Enter`] : 에러를 해결할 수 있게 도와줌

다음은 흔한 에러 발생 상황인 클래스가 import 되지 않은 상황입니다.

<img width="384" alt="스크린샷 2021-01-23 오전 4 04 24" src="https://user-images.githubusercontent.com/56301069/105535092-bec08200-5d31-11eb-9115-976d84d69520.png">


여기서 List에 캐럿을 두고 `Option + Enter` 를 누르면 해당 에러를 해결할 수 있는 방법들의 목록이 출력됩니다.

<img width="400" alt="스크린샷 2021-01-23 오전 4 05 07" src="https://user-images.githubusercontent.com/56301069/105535095-bf591880-5d31-11eb-82b7-6f21f42a8dce.png">


import class 를 누르면 자동으로 필요한 클래스가 import 됩니다.

<img width="388" alt="스크린샷 2021-01-23 오전 4 06 51" src="https://user-images.githubusercontent.com/56301069/105535096-bff1af00-5d31-11eb-8a13-bfae6f7f9c2e.png">



추후 더 좋은 단축키를 찾으면 추가하겠습니다.