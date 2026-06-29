# 4장 OpenCV GUI, I/O 처리 실습과제

## 실습과제 1

### 문제

`400×400` 영상을 생성하고 다음처럼 직선, 사각형, 원을 그린다.

* 첫 번째 창: 사각형과 X자 대각선
* 두 번째 창: 사각형 안에 원

### 코드

```python
import numpy as np
import cv2

white = (255, 255, 255)
black = (0, 0, 0)

# -----------------------------
# 1. 사각형 안에 X자 직선
# -----------------------------
image1 = np.full((400, 400, 3), white, np.uint8)

pt1 = (100, 100)
pt2 = (300, 100)
pt3 = (300, 300)
pt4 = (100, 300)

# 사각형 테두리
cv2.line(image1, pt1, pt2, black)
cv2.line(image1, pt2, pt3, black)
cv2.line(image1, pt3, pt4, black)
cv2.line(image1, pt4, pt1, black)

# 대각선
cv2.line(image1, pt1, pt3, black)
cv2.line(image1, pt2, pt4, black)

# -----------------------------
# 2. 사각형 안에 원
# -----------------------------
image2 = np.full((400, 400, 3), white, np.uint8)

# 사각형
cv2.rectangle(image2, (100, 100), (300, 300), black)

# 원
cv2.circle(image2, (200, 200), 100, black)

cv2.imshow("src1", image1)
cv2.imshow("src2", image2)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

---

## 실습과제 2

### 문제

Lenna 영상을 출력하고 아래처럼 라인을 그린다.

### 코드

```python
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_COLOR)

if image is None:
    raise Exception("lenna.bmp 파일을 불러올 수 없습니다.")

height = image.shape[0]
width = image.shape[1]

white = (255, 255, 255)

# 세로선 3개
cv2.line(image, (width // 4, 0), (width // 4, height), white)
cv2.line(image, (width // 2, 0), (width // 2, height), white)
cv2.line(image, (width * 3 // 4, 0), (width * 3 // 4, height), white)

# 가로선 3개
cv2.line(image, (0, height // 4), (width, height // 4), white)
cv2.line(image, (0, height // 2), (width, height // 2), white)
cv2.line(image, (0, height * 3 // 4), (width, height * 3 // 4), white)

cv2.imshow("Line", image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

---

## 실습과제 3

### 문제

RGB 각 영역이 정사각형이 되도록 영상을 생성하고, 다음처럼 화면을 세 영역으로 나눈다.

* 왼쪽 영역: 파란색 배경 + 흰색 사각형
* 가운데 영역: 초록색 배경 + 흰색 원
* 오른쪽 영역: 빨간색 배경 + 흰색 X자

전체 행렬에서 슬라이싱한 부분 행렬은 얕은 복사임을 이용한다.

### 코드

```python
import numpy as np
import cv2

# 색상 정의, OpenCV는 BGR 순서
blue = (255, 0, 0)
green = (0, 255, 0)
red = (0, 0, 255)
white = (255, 255, 255)

# 각 영역을 400x400 정사각형으로 만들기 위해
# 전체 영상 크기를 높이 400, 너비 1200으로 생성
image = np.zeros((400, 1200, 3), np.uint8)

height = image.shape[0]

# 한 영역의 크기
box_size = height  # 400

# 슬라이싱한 부분 행렬
left = image[:, 0:box_size]
center = image[:, box_size:box_size * 2]
right = image[:, box_size * 2:box_size * 3]

# 슬라이싱 영역은 얕은 복사이므로, 각 영역을 채우면 원본 image도 바뀜
left[:] = blue
center[:] = green
right[:] = red

# 왼쪽 영역: 흰색 사각형
cv2.rectangle(left, (100, 100), (300, 300), white, 10)

# 가운데 영역: 흰색 원
cv2.circle(center, (200, 200), 100, white, 10)

# 오른쪽 영역: 흰색 X
cv2.line(right, (100, 100), (300, 300), white, 10)
cv2.line(right, (300, 100), (100, 300), white, 10)

cv2.imshow("img", image)
cv2.waitKey(0)
cv2.destroyAllWindows()
```

---

## 실습과제 4

### 문제

다음 조건을 만족하는 초시계를 만든다.

* 윈도우 사이즈는 `300×300`
* 초기 상태는 `0`을 출력하고 대기
* `s` 키를 누르면 카운트 시작
* `t` 키를 누르면 정지
* `r` 키를 누르면 0으로 리셋
* `q` 키를 누르면 프로그램 종료
* 시간 제어 및 문자 입력은 `waitKey` 함수 사용
* 문자열 출력은 `putText` 함수 사용
* 반복문은 1번만 사용

### 코드

```python
import numpy as np
import cv2

white = (255, 255, 255)
black = (0, 0, 0)

title = "Stop Watch"

count = 0
running = False

while True:
    image = np.full((300, 300, 3), white, np.uint8)

    cv2.putText(
        image,
        str(count),
        (120, 170),
        cv2.FONT_HERSHEY_SIMPLEX,
        3,
        black,
        3
    )

    cv2.imshow(title, image)

    delay = 1000 if running else 100
    key = cv2.waitKey(delay)

    if key == ord('s'):
        running = True

    elif key == ord('t'):
        running = False

    elif key == ord('r'):
        count = 0
        running = False

    elif key == ord('q'):
        break

    elif running:
        count += 1

cv2.destroyAllWindows()
```

---

## 실습과제 5

### 문제

키보드를 이용하여 파란색 버튼을 움직이는 프로그램을 작성한다.

조건은 다음과 같다.

1. 초기 화면은 폭 `500`, 높이 `500` 픽셀의 3채널 컬러 영상으로 설정한다.
2. 화면은 바둑판 모양으로 직선을 출력한다.
3. 파란색 버튼은 바둑판 한 칸을 채우는 정사각형 형태로 만든다.
4. 초기 위치는 화면의 가운데 칸으로 설정한다.
5. 왼쪽 화살표 키를 누르면 현재 위치에서 왼쪽으로 한 칸 이동한다.
6. 위쪽 화살표 키를 누르면 현재 위치에서 위쪽으로 한 칸 이동한다.
7. 오른쪽 화살표 키를 누르면 현재 위치에서 오른쪽으로 한 칸 이동한다.
8. 아래쪽 화살표 키를 누르면 현재 위치에서 아래쪽으로 한 칸 이동한다.
9. 윈도우를 벗어나는 경우 현재 위치를 유지한다.
10. `q` 키를 누르면 종료한다.

### 코드

```python
import numpy as np
import cv2

white = (255, 255, 255)
black = (0, 0, 0)
blue = (255, 0, 0)

title = "src"

size = 500
cell = 100

# 버튼의 현재 위치를 행, 열로 저장
row = 2
col = 2


def draw_screen():
    image = np.full((size, size, 3), white, np.uint8)

    # 파란색 버튼 먼저 그리기
    x1 = col * cell
    y1 = row * cell
    x2 = x1 + cell
    y2 = y1 + cell

    cv2.rectangle(image, (x1, y1), (x2, y2), blue, cv2.FILLED)

    # 바둑판 선 그리기
    for i in range(0, size + 1, cell):
        cv2.line(image, (i, 0), (i, size), black)
        cv2.line(image, (0, i), (size, i), black)

    return image


while True:
    image = draw_screen()
    cv2.imshow(title, image)

    key = cv2.waitKeyEx(0)

    # 왼쪽 화살표
    if key == 2424832:
        if col - 1 >= 0:
            col -= 1

    # 위쪽 화살표
    elif key == 2490368:
        if row - 1 >= 0:
            row -= 1

    # 오른쪽 화살표
    elif key == 2555904:
        if col + 1 < size // cell:
            col += 1

    # 아래쪽 화살표
    elif key == 2621440:
        if row + 1 < size // cell:
            row += 1

    # q 키 종료
    elif key == ord('q'):
        break

cv2.destroyAllWindows()
```

---

## 실습과제 6

### 문제

키보드를 이용하여 직선을 그리는 프로그램을 작성한다.

조건은 다음과 같다.

1. 초기 화면은 폭 `500`, 높이 `500` 픽셀의 3채널 컬러 영상으로 설정한다.
2. 직선의 시작 위치는 화면의 중심으로 설정한다.
3. `q` 키를 누르면 종료한다.
4. 왼쪽 화살표 키를 누르면 현재 위치에서 왼쪽으로 길이 50픽셀인 직선을 그린다.
5. 위쪽 화살표 키를 누르면 현재 위치에서 위쪽으로 길이 50픽셀인 직선을 그린다.
6. 오른쪽 화살표 키를 누르면 현재 위치에서 오른쪽으로 길이 50픽셀인 직선을 그린다.
7. 아래쪽 화살표 키를 누르면 현재 위치에서 아래쪽으로 길이 50픽셀인 직선을 그린다.

### 코드

```python
import numpy as np
import cv2

white = (255, 255, 255)
black = (0, 0, 0)

title = "Draw Line"

size = 500
step = 50

image = np.full((size, size, 3), white, np.uint8)

x = size // 2
y = size // 2

cv2.imshow(title, image)

while True:
    key = cv2.waitKeyEx(0)

    if key == ord('q'):
        break

    old_x = x
    old_y = y

    # 왼쪽 화살표
    if key == 2424832:
        if x - step >= 0:
            x -= step

    # 위쪽 화살표
    elif key == 2490368:
        if y - step >= 0:
            y -= step

    # 오른쪽 화살표
    elif key == 2555904:
        if x + step < size:
            x += step

    # 아래쪽 화살표
    elif key == 2621440:
        if y + step < size:
            y += step

    if old_x != x or old_y != y:
        cv2.line(image, (old_x, old_y), (x, y), black)
        cv2.imshow(title, image)

cv2.destroyAllWindows()
```

---

## 실습과제 7

### 문제

다음을 만족시키는 마우스로 곡선을 그려주는 프로그램을 작성한다.

조건은 다음과 같다.

1. `500×500` 크기의 윈도우를 생성한다.
2. 바탕은 흰색으로 설정한다.
3. 트랙바를 생성하고 `0`, `1`, `2` 중 하나를 선택하도록 한다.
4. 트랙바 위치가 `0`이면 마우스를 드래그할 때 파란색으로 선을 그린다.
5. 트랙바 위치가 `1`이면 마우스를 드래그할 때 초록색으로 선을 그린다.
6. 트랙바 위치가 `2`이면 마우스를 드래그할 때 빨간색으로 선을 그린다.
7. 왼쪽 버튼이 눌리면 그리기를 시작하고, 버튼을 떼면 종료한다.
8. `MOUSEMOVE` 이벤트가 발생할 때마다 이전 위치와 현재 위치를 직선으로 이어준다.
9. `q`를 누르면 프로그램을 종료한다.

### 코드

```python
import numpy as np
import cv2

white = (255, 255, 255)
blue = (255, 0, 0)
green = (0, 255, 0)
red = (0, 0, 255)

title = "img"
bar_name = "Color"

image = np.full((500, 500, 3), white, np.uint8)

drawing = False
old_x = 0
old_y = 0
color_pos = 0


def onChange(pos):
    global color_pos
    color_pos = pos


def getColor():
    if color_pos == 0:
        return blue
    elif color_pos == 1:
        return green
    else:
        return red


def onMouse(event, x, y, flags, param):
    global drawing, old_x, old_y, image

    if event == cv2.EVENT_LBUTTONDOWN:
        drawing = True
        old_x = x
        old_y = y

    elif event == cv2.EVENT_MOUSEMOVE:
        if drawing:
            color = getColor()
            cv2.line(image, (old_x, old_y), (x, y), color, 2)
            old_x = x
            old_y = y
            cv2.imshow(title, image)

    elif event == cv2.EVENT_LBUTTONUP:
        drawing = False


cv2.imshow(title, image)
cv2.createTrackbar(bar_name, title, 0, 2, onChange)
cv2.setMouseCallback(title, onMouse)

while True:
    key = cv2.waitKey(100)

    if key == ord('q'):
        break

cv2.destroyAllWindows()
```

---
