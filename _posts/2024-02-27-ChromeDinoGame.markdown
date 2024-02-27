---
layout: post
title:  "Chrome Dino Game"
date:   2024-02-27 18:34:34 +0900
categories: science
author: "moon&blue"
---

정규 수업 전에 python 공부를 하기 위해
Chrome Dino Game 구현을 탐구해봤다.

실은 내가 게임을 만들어보고 싶었지만.

내가 어떤 게임을 만들어보려고 하다가 난항에 봉착해서 구글링을 하면
이미 완성된 프로그램들이 한가득이어서.

그럴 필요성을 잘 못느끼겠달까.

쟈. 그렇다면, 한 갈래 한 갈래씩 코드를 공부해보쟈.

우선 코드의 개괄은 아래와 같다.

## 코드 개괄
1. 게임 초기화 및 설정

2. 공룡 캐릭터 클래스, 장애물 클래스

3. 화면 업데이트 및 렌더링 설정

4. 게임 실행 및 메뉴 화면 표시

## 코드 부분 설명
- 코드 시작 전: 필요한 라이브러리와 상수 설정

- Dinosaur 클래스: 공룡 캐릭터 정의, 캐릭터 움직임 관리

- Cloud 클래스: 구름 정의, 화면 움직임 설정

- Obstacle 클래스: 모든 장애물의 기본 특성을 정의, 장애물 종류 따른 하위 클래스 상속

- main() 함수: 게임 메인 루프 정의, 이 속에서 게임 상태 업데이트 및 렌더링

- menu() 함수: 게임 시작 화면, 게임 종료 화면 담당


## 주요 로직

공룡 캐릭터는 위로 점프 or 아래 숙여 장애물 회피

장애물 형태는 다양, 플레이어는 이를 피해야 함

게임 진행 따른 속도 증가, 플레이어 점수 증가

게임 종료 시 점수 저장, 다시 시작 가능

    - 게임 종료 시 점수 저장에 버그가 좀 있음(작성시간 기준)


이제부터는 게임 내부 코드다.

## 필요 모듈 및 라이브러리 import

```python
import datetime
import os
import random
import threading
import pygame
```
[threading](https://wikidocs.net/124143)관련 페이지

> threading은 스레드를 이용하여 한 프로세스에서 2가지 이상의 일을 동시에 실행할 수 있게 하는 모듈이다.

## Pygame 초기화
``` python
pygame.init()
```
[pygame](https://www.jbmpa.com/pygame/1)
> pygame은 GUI를 구현해주는 모델이다

## 화면 크기 설정
```python
SCREEN_HEIGHT = 600
SCREEN_WIDTH = 1100
SCREEN = pygame.display.set_mode(SCREEN_HEIGHT, SCREEN_WIDTH)
```
오우. 깔끔하게 상수(관례적)에 가로세로 창 사이즈를 정의하고, 매개변수로 SCREEN 설정

## 게임 창 제목 설정

```python
pygame.display.set_caption("Chrome Dino Runner")
```
아까 쟤는 `set_mode` 였고, 지금은 `set_caption`

## 아이콘 설정
```python
Ico = pygame.image.load("assets\DinoWallapaer.png")
pygame.display.set_icon(Ico)
```
python... 넌 정말 미스테리한 존재야.
어떻게 이미지를 담을 수 있지?

python의 변수 선언 방식은 모두 간접참조 형식이라고 언뜻 공부했던 기억이 나는 것도 같고...

여튼 이번엔 pygame.display의 `set_icon`을 사용
그리고 이미지를 불러올 때는 `pygame.image.load("주소\이름.확장자")`를 사용

## 캐릭터 이미지 로드, 장애물 이미지 로드
```python
RUNNING = [
    pygame.image.load(os.path.join("assets\Dino","Dinorun1.png")),
    pygame.image.load(os.path.join("assets\Dino", "Dinorun2.png")),
]
```
우선 `RUNNING`변수에 대해서 먼저 설명해보자면

`os.path.join()`은 요소로 입력받은 **디렉토리** 와 **파일** 이름을 결합하여 이미지 파일에 대한 전체 경로를 만든다.

그리고 리스트의 요소들이 쉼표로 묶일 때, 마지막 요소에 대해서도 쉼표가 부가된 것을 확인할 수 있는데

이는 코드의 일관성 유지에 도움이 되는 요소로서 권장되는 스타일 중 하나다.
별도로 코드를 추가하거나 지워도 문제가 되지 않게끔 도와주기 때문!

남은 코드도 이어서 적어보겠다.

### 캐릭터와 장애물 이미지 로드 cont'd

```python
JUMPING = pygame.image.load(os.path.join("assets\Dino", "DinoJump.png"))
DUCKING = [
    pygame.image.load(os.path.join("assets\Dino", "DinoDuck1.png")),
    pygame.image.load(os.path.join("assets\Dino","DinoDuck2.png")),
]
```

뜬금없을 수 있는 질문이지만 한 번에 전체 경로를 입력해서 사용하지 않고, 디렉토리 경로와 파일 경로를 구분해서 join시켜 사용하는 이유는 뭘까?

학교 수업시간에도 java에서 파일 경로를 표현하는 too much 해보이는 방법들을 보며 생각했던 의문인데, 이번에 모사코딩(?)을 해보면서 알았다.

반복되는 코드 구조에 대해 달라지는 부분만 수기작업을 하고, 이런 작업 방식을 협력하는 개발자들과 묵시적으로 공유하기 위함인듯 하다.

### 캐릭터와 장애물 이미지 로드 cont'd

```python
SMALL_CACTUS = [
    pygame.image.load(os.path.join("assets\Cactus", "SmallCactus1.png")),
    pygame.image.load(os.path.join("assets\Cactus", "SmallCactus2.png")),
    pygame.image.load(os.path.join("assets\Cactus", "SmallCactus3.png")),
]
LARGE_CACTUS = [
    pygame.image.load(os.path.join("assets\Cactus", "LargeCactus1.png")),
    pygame.image.load(os.path.join("assets\Cactus", "LargeCactus2.png")),
    pygame.image.load(os.path.join("assets\Cactus", "LargeCactus3.png")),
]
BIRD = [
    pygame.image.load(os.path.join("assets\Bird", "Bird1.png")),
    pygame.image.load(os.path.join("assets\Bird", "Bird2.png")),
]
CLOUD = pygame.image.load(os.path.join("assets\Other", "Cloud.png"))
BG = pygame.image.load(os.path.join("assets\Other", "Track.png"))
FONT_COLOR=(0,0,0)
```

여기서 `FONT_COLOR=(0,0,0)`은 생소해서 뭔가 했는데 튜플이었다. 근데 어차피 000이면 List로 하면 안되나

아. 상수표현. 오키.
여러모로 코딩 방식에 대해서도 공부해보게 되는 것 같다.

모사코딩(?)이란.

~~살사소-스~~

## 공통 클래스

이제 위에서 말한 다양한 클래스에 대해서 기술할 거다.
### Dinosaur 클래스 & 하위 초기화 함수
> 어릴 적~ 내 꿈에 나온 Dinosaur~~

```python
class Dinosaur:
    X_POS = 80
    Y_POS = 310
    Y_POS_DUCK = 340
    JUMP_VEL = 8.5

    def __init__(self): #초기화 함수
        self.duck_img = DUCKING
        self.run_img = RUNNING
        self.jump_img = JUMPING

        self.dino_duck = False
        self.dino_run = True
        self.dino_jump = False
        self.step_index = 0

        self.jump_vel = self.JUMP_VEL
        self.image = self.run_img[0]
        
        self.dino_rect = self.image.get_rect()
        self.dino_rect.x = self.X_POS
        self.dino_rect.y = self.Y_POS
```

입장시 자기 변수를 초기화하는 구문인가보다.
그런데 설마... 미리 선언해놓지 않은 변수도 이때 선언해서 정의할 수 있는건가...?

코드를 좀 더 공부해야겠다.

### 공룡 클래스의 캐릭터 업데이트 함수
```python
        def update(self, userInput):
            if self.dino_duck:
                self.duck() #duck 함수요?!
            if self.dino_run:
                self.run()  #함수예고편인가보다
            if self.dino_jump:
                self.jump() #너도 함수 예고편?
            
            if self.step_index >= 10:
                self.step_index = 0
            
            if (userInput[pygame.K_UP] or userInput[pygame.K_SPACE]) and not self.dino_jump:
                self.dino_duck = False
                self.dino_run = False
                self.dino_jump = True
            elif userInput[pygame.K_DOWN] and not self.dino_jump:
                self.dino_duck = True
                self.dino_run = False
                self.dino_jump = False
            elif not (self.dino_jump or userInput[pygame.K_DOWN]):
                self.dino_duck = False
                self.dino_run = True
                self.dino_jump = False
```

`up key나 sapce를 누르고 dino가 jump상태가 아닐 때`
- 수그리기 = False
- 달리기 = False
- 점프하기 = True

또는

`down key를 누르고 jump 상태가 아닐 때`
- 수그리기 = True
- 달리기 = False
- 점프하기 = False

또는

`dino_jump가 아니고 K_DOWN이 아닐 때`
- 수그리기 = False
- 달리기 = True
- 점프하기 = False


python에서 && 이나 ||은 and와 or로 작성하나보다.
else if는 elif

### 캐릭터 숙이기
```python
def duck(self):
    self.image = self.duck_img[self.step_index // 5] # 헉 이렇게 줄인 이미지였다니
    self.dino_rect = self.image.get_rect()
    self.dino_rect.x = self.X_POS
    self.dino_rect.y = self.Y_POS_DUCK
    self.step_index += 1
```
`get_rect()`에 대하여!

game box 같은 걸 가져오는 함수라고 함

[get_rect()](https://www.jbmpa.com/pygame/4)

duck_img를 변형한 것을 image에 할당.

그 후 get_rect()를 통해 self.dino_rect에 이미지 위치 및 크기 객체를 반환!

### 계속해서 dinosaur 클래스 내부 함수

```python
def run(self):
    self.image = self.run_img[self.step_index //5]
    self.dion_rect = self.image.get_rect()
    self.dino_rect.x = self.X_POS
    self.dino_rect.y = self.Y_POS
    self.step_index += 1
```
x좌표 위치 설정해주고~

y좌표 위치 설정해주고~

```python
def jump(self):
    self.image = self.jump_img
    if self.dino_jump:
        self.dino_rect.y -= self.jump_vel * 4
        self.jump_vel -= 0.8
    
    if self.jump_vel < -self.JUMP_VEL:
        self.dino_jump = False
        self.jump_vel = self.JUMP_VEL

```

오오... 객체에 저걸 때에 따라서 대입하는 식으로 코딩하는구만

1. 점프 중 이미지로 변경

2. dino_jump가 True인 상황에서 캐릭터 효과 변경

3. 점프 속도 < 최대 점프 속도:
    dino_jump를 false로 설정,
    self.jump_vel을 다시 최대 점프 속도로 설정!

-> 캐릭터의 점프 동작 제어, 중력과 최대 점프 높이 제한 위한 설정.

### 캐릭터 그리기
```python
def draw(self, SCREEN):
    SCREEN.blit(self.image, (self.dino_rect.x, self.dino_rect.y))

```
self.image를 self.dino_rect.x, self.dino_rect.y에 그리는 명령!

그런데, bilt함수는 무엇이냐!

얘는 pygame에서 나온 함수!

아래 두 개 사이트를 참고하면서 공부하면 좋을 것 같다.

[jbmpa](https://www.jbmpa.com/pygame/9?sst=wr_hit)

[NaverBlog](https://m.blog.naver.com/scyan2011/221980550330)


## 장애물 클래스
이번에는 한번에 다 적어버리겠다!
```python
class Obstacle:
    def __init__(self, image, type):
        self.image = image
        self.type = type
        self.rect = self.image[self.type].get_rect()
        self.fect.x = SCREEN_WIDTH
    
    #장애물 업데이트
    def update(self):
        self.rect.x -= game_speed
        if self.rect.x < -self.rect.width:
            obstacles.pop()
    
    #장애물 그리기
    def draw(self. SCREEN):
        SCREEN.blit(self, image[self.type], self.rect)
```

## 작은 선인장 클래스
```python
class SmallCatus(Obstacle):
    def __init__(self, image):
        self.type = random.randint(0.2)
        super().__init__(image, self.type)
        self.rect.y = 325
```

## 큰 선인장 클래스
```python
class LargeCactus(Obstacle):
    def __init__(self, image):
        self.type = random.randint(0, 2)
        super().__init__(image, self.type)
        self.rect.y = 300
```

## 새 클래스
```python
class Bird(Obstacle):
    BIRD_HEIGHTS = [250, 290, 320]
    def __init__(self, image):
        self.type = 0
        super().__init__(image, self.type)
        self.rect.y = random.choice(self.BIRD_HEIGHTS)
        self.index = 0
    # 새 그리기
    def draw(self, SCREEN):
        if self.index >= 9:
            self.index = 0
        SCREEN.blit(self.image[self.index // 5], self.rect)
        self.index += 1

```

마지막 메인 함수는 다음 포스트에 업로드!