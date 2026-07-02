# ch06-1 화소처리 실습과제

## 공통 실행 조건

모든 실습 코드는 `lenna.bmp` 파일이 Python 파일과 같은 폴더에 있다고 가정한다.

예시 폴더 구조는 다음과 같다.

```text
opencv-python/
├── ch06_1_ex1.md
├── ch06_1_ex2.py
├── ch06_1_ex3.py
├── ch06_1_ex4.py
└── lenna.bmp
```

---

## 실습과제 1

### 문제

예제에서 보듯이 반복문을 이용한 화소처리와 OpenCV 함수를 이용한 처리의 연산시간의 차이가 크게 나는 이유를 설명하라.

### 답안

반복문을 이용한 화소처리는 Python 코드에서 모든 픽셀을 하나씩 직접 접근하여 처리한다. 이 경우 각 픽셀마다 Python 인터프리터가 반복문을 실행하고, 배열 인덱싱을 수행하며, 자료형 변환과 조건 검사를 반복해야 한다.

영상은 수많은 픽셀로 이루어져 있기 때문에, 작은 영상이라도 반복문 횟수가 매우 많아진다. 예를 들어 `512×512` 크기의 그레이스케일 영상은 총 `262,144`개의 픽셀을 가진다. 이 픽셀을 Python 반복문으로 하나씩 처리하면 실행 시간이 길어진다.

반면 OpenCV 함수는 내부적으로 C/C++ 기반으로 구현되어 있다. Python에서 `cv2.add()`와 같은 함수를 호출하면 실제 픽셀 단위 연산은 OpenCV 내부의 최적화된 코드에서 수행된다. 따라서 Python 반복문보다 훨씬 적은 오버헤드로 빠르게 처리된다.

또한 OpenCV 함수는 배열 전체를 한 번에 처리하며, 포화연산까지 내부적으로 처리한다. 예를 들어 `cv2.add()` 함수는 픽셀값이 255를 넘는 경우 자동으로 255로 제한한다.

따라서 반복문을 이용한 화소처리는 원리를 이해하기에는 좋지만 속도가 느리고, OpenCV 함수를 이용한 처리는 내부 최적화 덕분에 훨씬 빠르다.

---

## 실습과제 2

### 문제

트랙바의 값을 조절하여 영상의 밝기를 최대 100만큼 밝게 하는 코드를 작성하시오.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

title = "dst"


def onChange(pos):
    dst = cv2.add(image, pos)
    cv2.imshow(title, dst)


cv2.imshow(title, image)
cv2.createTrackbar("Brightness", title, 0, 100, onChange)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

트랙바의 위치값 `pos`는 `0~100` 사이의 값을 가진다.

```python
dst = cv2.add(image, pos)
```

위 코드는 원본 영상의 모든 픽셀값에 `pos`만큼 더한다.

`cv2.add()` 함수는 내부적으로 포화연산을 수행하므로, 결과 픽셀값이 `255`를 넘으면 자동으로 `255`로 제한된다.

예를 들어 원래 픽셀값이 `230`이고 트랙바 값이 `50`이면 계산 결과는 `280`이지만, 실제 출력 픽셀값은 `255`가 된다.

---

## 실습과제 3

### 문제

레나 영상을 그레이스케일로 출력하고 트랙바를 이용하여 `0`과 `1`을 선택하도록 하라.

트랙바의 위치가 `0`일 때는 왼쪽 마우스를 클릭할 때마다 `10`만큼 밝기를 증가시키고, 트랙바의 위치가 `1`일 때는 왼쪽 마우스를 클릭할 때마다 `10`만큼 밝기를 감소시키는 코드를 작성하시오.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

title = "dst"

dst = image.copy()
mode = 0


def onTrackbar(pos):
    global mode
    mode = pos


def onMouse(event, x, y, flags, param):
    global dst, mode

    if event == cv2.EVENT_LBUTTONDOWN:
        if mode == 0:
            dst = cv2.add(dst, 10)
        else:
            dst = cv2.subtract(dst, 10)

        cv2.imshow(title, dst)


cv2.imshow(title, dst)
cv2.createTrackbar("mode", title, 0, 1, onTrackbar)
cv2.setMouseCallback(title, onMouse)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

트랙바 값이 `0`이면 왼쪽 마우스를 클릭할 때마다 영상 전체의 밝기가 `10` 증가한다.

트랙바 값이 `1`이면 왼쪽 마우스를 클릭할 때마다 영상 전체의 밝기가 `10` 감소한다.

밝기 증가는 `cv2.add()`를 사용하고, 밝기 감소는 `cv2.subtract()`를 사용한다.

```python
dst = cv2.add(dst, 10)
```

```python
dst = cv2.subtract(dst, 10)
```

두 함수 모두 내부적으로 포화연산을 수행한다. 따라서 밝기 증가 시 `255`를 초과하지 않고, 밝기 감소 시 `0`보다 작아지지 않는다.

---

## 실습과제 4

### 문제

마우스 이벤트를 이용하여 레나 영상에서 마우스로 드래깅한 영역만 `100`만큼 픽셀값을 증가시키는 코드를 작성하라.

힌트: 마우스 이벤트와 부분행렬 추출방법을 사용하라.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

title = "src"

dst = image.copy()

dragging = False
start_x = 0
start_y = 0


def onMouse(event, x, y, flags, param):
    global dst, dragging, start_x, start_y

    if event == cv2.EVENT_LBUTTONDOWN:
        dragging = True
        start_x = x
        start_y = y

    elif event == cv2.EVENT_LBUTTONUP:
        dragging = False

        end_x = x
        end_y = y

        x1 = min(start_x, end_x)
        x2 = max(start_x, end_x)
        y1 = min(start_y, end_y)
        y2 = max(start_y, end_y)

        if x1 == x2 or y1 == y2:
            return

        roi = dst[y1:y2, x1:x2]
        dst[y1:y2, x1:x2] = cv2.add(roi, 100)

        cv2.imshow(title, dst)


cv2.imshow(title, dst)
cv2.setMouseCallback(title, onMouse)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

마우스를 누른 위치를 시작 좌표로 저장하고, 마우스를 뗀 위치를 끝 좌표로 사용한다.

```python
start_x = x
start_y = y
```

마우스를 떼면 시작 좌표와 끝 좌표를 이용해 드래그한 영역을 계산한다.

```python
x1 = min(start_x, end_x)
x2 = max(start_x, end_x)
y1 = min(start_y, end_y)
y2 = max(start_y, end_y)
```

그 후 해당 영역을 부분행렬로 추출한다.

```python
roi = dst[y1:y2, x1:x2]
```

마지막으로 해당 영역에만 `cv2.add(roi, 100)`을 적용하여 드래그한 영역만 밝게 만든다.

```python
dst[y1:y2, x1:x2] = cv2.add(roi, 100)
```
