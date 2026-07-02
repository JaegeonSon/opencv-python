# 4장 OpenCV GUI, I/O 처리 실습과제

## 실습과제 1

### 문제

Lenna 영상 위에서 발생하는 마우스 이벤트의 횟수를 세어서 출력하는 프로그램을 작성한다.

다음 이벤트만 발생 횟수를 출력한다.

* 왼쪽 버튼 Down
* 왼쪽 버튼 Up
* 마우스 Move

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import cv2

image = cv2.imread('lenna.bmp', cv2.IMREAD_COLOR)

if image is None:
    raise Exception("파일 읽기 오류")

title = "Mouse Event Count"

left_down_count = 0
left_up_count = 0
move_count = 0


def onMouse(event, x, y, flags, param):
    global left_down_count, left_up_count, move_count

    if event == cv2.EVENT_LBUTTONDOWN:
        left_down_count += 1
        print("왼쪽 버튼 DOWN 이벤트 횟수:", left_down_count)

    elif event == cv2.EVENT_LBUTTONUP:
        left_up_count += 1
        print("왼쪽 버튼 UP 이벤트 횟수:", left_up_count)

    elif event == cv2.EVENT_MOUSEMOVE:
        move_count += 1
        print("마우스 MOVE 이벤트 횟수:", move_count)


cv2.imshow(title, image)
cv2.setMouseCallback(title, onMouse)

while True:
    key = cv2.waitKey(100)

    if key == 27:
        break

cv2.destroyAllWindows()
```

---

## 실습과제 2

### 문제

Lenna 영상을 그레이스케일 영상으로 변환 후 화면에 출력하고, 마우스로 클릭하는 화면 좌표와 그 점에서의 화소값을 출력하는 프로그램을 작성한다.

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import cv2

gray = cv2.imread('lenna.bmp', cv2.IMREAD_GRAYSCALE)

if gray is None:
    raise Exception("파일 읽기 오류")

title = "Gray Lenna"


def onMouse(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        pixel = gray[y, x]

        print("클릭 좌표:", x, y)
        print("화소값:", pixel)


cv2.imshow(title, gray)
cv2.setMouseCallback(title, onMouse)

while True:
    key = cv2.waitKey(100)

    if key == 27:
        break

cv2.destroyAllWindows()
```

---

## 실습과제 3

### 문제

Lenna 영상을 컬러 영상으로 화면에 출력하고, 마우스로 클릭하는 화면 좌표와 해당 좌표의 화소값 `(B, G, R)`을 출력하는 프로그램을 작성한다.

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import cv2

color = cv2.imread('lenna.bmp', cv2.IMREAD_COLOR)

if color is None:
    raise Exception("파일 읽기 오류")

title = "Color Lenna"


def onMouse(event, x, y, flags, param):
    if event == cv2.EVENT_LBUTTONDOWN:
        b = color[y, x, 0]
        g = color[y, x, 1]
        r = color[y, x, 2]

        print("클릭 좌표:", x, y)
        print("BGR 화소값:", b, g, r)


cv2.imshow(title, color)
cv2.setMouseCallback(title, onMouse)

while True:
    key = cv2.waitKey(100)

    if key == 27:
        break

cv2.destroyAllWindows()
```

---

## 실습과제 4

### 문제

트랙바 이벤트를 이용하여 실행 결과가 나오도록 코드를 작성한다.

트랙바를 움직일 때마다 다음 내용을 출력한다.

* 트랙바 이벤트 횟수
* 트랙바 위치

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import cv2

image = cv2.imread('lenna.bmp', cv2.IMREAD_COLOR)

if image is None:
    raise Exception("파일 읽기 오류")

title = "src"
bar_name = "level"

trackbar_count = 0


def onChange(pos):
    global trackbar_count

    trackbar_count += 1

    print("트랙바 이벤트 횟수:", trackbar_count)
    print("트랙바 위치:", pos)


cv2.imshow(title, image)
cv2.createTrackbar(bar_name, title, 0, 10, onChange)

while True:
    key = cv2.waitKey(100)

    if key == 27:
        break

cv2.destroyAllWindows()
```

---

## 실습과제 5

### 문제

크기가 `400 × 200`이고 배경색이 백색인 컬러 영상을 생성한다.

마우스 왼쪽 버튼을 누르면 배경색이 Red가 되고, 마우스 오른쪽 버튼을 누르면 배경색이 Blue가 되도록 프로그램을 작성한다.

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import numpy as np
import cv2

image = np.full((200, 400, 3), 255, np.uint8)

title = "Mouse Color Change"


def onMouse(event, x, y, flags, param):
    global image

    if event == cv2.EVENT_LBUTTONDOWN:
        image.fill(0)
        image[:, :, 2].fill(255)
        cv2.imshow(title, image)

    elif event == cv2.EVENT_RBUTTONDOWN:
        image.fill(0)
        image[:, :, 0].fill(255)
        cv2.imshow(title, image)


cv2.imshow(title, image)
cv2.setMouseCallback(title, onMouse)

while True:
    key = cv2.waitKey(100)

    if key == 27:
        break

cv2.destroyAllWindows()
```

### 참고

OpenCV 컬러 영상은 RGB 순서가 아니라 BGR 순서이다.

```python
image[:, :, 0]  # Blue
image[:, :, 1]  # Green
image[:, :, 2]  # Red
```

따라서 Red는 2번 채널, Blue는 0번 채널을 사용한다.

---

## 실습과제 6

### 문제

크기가 `400 × 200`인 컬러 영상을 생성한다.

배경색이 1초 주기로 다음 순서로 변화하도록 프로그램을 작성한다.

```text
Blue → Green → Red
```

ESC를 누르면 프로그램을 종료한다.

### 코드

```python
import numpy as np
import cv2

image = np.zeros((200, 400, 3), np.uint8)

title = "Color Animation"

i = 0

while True:
    image.fill(0)

    if i % 3 == 0:
        image[:, :, 0].fill(255)
        print("Blue")

    elif i % 3 == 1:
        image[:, :, 1].fill(255)
        print("Green")

    else:
        image[:, :, 2].fill(255)
        print("Red")

    cv2.imshow(title, image)

    i += 1

    key = cv2.waitKey(1000)

    if key == 27:
        break

cv2.destroyAllWindows()
```

---

## 실행 시 주의사항

`lenna.bmp`를 사용하는 실습과제 1~4번은 Python 파일과 같은 폴더에 `lenna.bmp` 파일이 있어야 한다.

예시 폴더 구조는 다음과 같다.

```text
CV_SJG/
├── ex1.py
├── ex2.py
├── ex3.py
├── ex4.py
└── lenna.bmp
```

또한 ESC 키 입력은 콘솔창이 아니라 OpenCV 영상 창이 선택된 상태에서 해야 한다.
