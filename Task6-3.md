# ch06-3 화소처리 실습과제

## 공통 실행 조건

모든 실습 코드는 `lenna.bmp` 파일이 Python 파일과 같은 폴더에 있다고 가정한다.

예시 폴더 구조는 다음과 같다.

```text
opencv-python/
├── ch06_3_ex1.py
├── ch06_3_ex2.py
├── ch06_3_ex3.py
└── lenna.bmp
```

---

## 실습과제 1

### 문제

`lenna.bmp` 영상에 대하여 다음처럼 출력하는 프로그램을 작성하시오.

최대, 최소값은 `cv2.minMaxLoc` 함수를 사용하여 구하시오.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def calcGrayHist(image):
    images = [image]
    channels = [0]
    hsize = [256]
    ranges = [0, 256]

    hist = cv2.calcHist(images, channels, None, hsize, ranges)

    return hist


hist = calcGrayHist(image)

total_pixels = image.shape[0] * image.shape[1]

min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(image)

hist_min, hist_max, hist_min_loc, hist_max_loc = cv2.minMaxLoc(hist)

most_pixel_value = hist_max_loc[1]
most_pixel_count = int(hist_max)

pixel_80_count = int(hist[80][0])

print("영상의 전체 픽셀수:", total_pixels)
print("영상에서 픽셀값의 최소값:", int(min_val))
print("영상에서 픽셀값의 최대값:", int(max_val))
print("빈도수가 가장 많은 픽셀값과 빈도수:", most_pixel_value, most_pixel_count)
print("픽셀값 80의 빈도수:", pixel_80_count)

cv2.imshow("src", image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

`cv2.minMaxLoc(image)`는 영상에서 최소 픽셀값과 최대 픽셀값을 구한다.

히스토그램은 각 픽셀값이 영상에 몇 번 등장하는지를 저장한 배열이다.

예를 들어 `hist[80][0]`은 픽셀값 `80`의 빈도수를 의미한다.

`cv2.minMaxLoc(hist)`를 사용하면 히스토그램에서 빈도수가 가장 큰 값도 구할 수 있다.

---

## 실습과제 2

### 문제

앞 예제에서 `calcGrayHist` 함수를 `cv2.calcHist`를 사용하지 말고 직접 구현하여 같은 결과가 나오도록 작성하라.

함수명은 `mycalcGrayHist`로 정의하고 매개변수와 리턴타입은 `calcGrayHist` 함수와 같게 정의하라.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def mycalcGrayHist(image):
    hist = np.zeros((256, 1), np.float32)

    for y in range(image.shape[0]):
        for x in range(image.shape[1]):
            pixel = image[y, x]
            hist[pixel][0] += 1

    return hist


hist = mycalcGrayHist(image)

print(type(hist), hist.ndim, hist.shape, hist.dtype)

cv2.imshow("입력 영상", image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

그레이스케일 영상은 픽셀값이 `0`부터 `255`까지의 값을 가진다.

따라서 히스토그램은 총 `256`개의 빈을 가져야 한다.

```python
hist = np.zeros((256, 1), np.float32)
```

위 코드는 `256×1` 크기의 히스토그램 배열을 생성한다.

각 픽셀을 직접 접근하여 해당 픽셀값에 해당하는 히스토그램 빈을 1씩 증가시킨다.

```python
pixel = image[y, x]
hist[pixel][0] += 1
```

이렇게 하면 `cv2.calcHist()`를 사용하지 않고도 같은 형태의 히스토그램을 만들 수 있다.

---

## 실습과제 3

### 문제

`lenna.bmp` 영상의 히스토그램 그래프를 아래 그림처럼 그려보시오.

함수명은 `mydrawGrayHistImage`로 정의하고 매개변수와 리턴타입은 `drawGrayHistImage` 함수와 같게 정의하라.

힌트: `line` 함수를 이용하여 막대그래프의 끝점끼리 직선으로 이어준다.

직선을 그리는 반복문은 `i=0`에서 `i=254`까지임을 주의하라.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def mycalcGrayHist(image):
    hist = np.zeros((256, 1), np.float32)

    for y in range(image.shape[0]):
        for x in range(image.shape[1]):
            pixel = image[y, x]
            hist[pixel][0] += 1

    return hist


def mydrawGrayHistImage(hist):
    hist_h = 100
    hist_w = 256

    hist_image = np.full((hist_h, hist_w), 255, np.uint8)

    hist_norm = np.zeros(hist.shape, np.float32)
    cv2.normalize(hist, hist_norm, 0, hist_h, cv2.NORM_MINMAX)

    hist_norm = hist_norm.flatten()

    for i in range(0, 255):
        x1 = i
        y1 = hist_h - int(hist_norm[i])

        x2 = i + 1
        y2 = hist_h - int(hist_norm[i + 1])

        cv2.line(hist_image, (x1, y1), (x2, y2), 0)

    return hist_image


hist = mycalcGrayHist(image)
hist_image = mydrawGrayHistImage(hist)

cv2.imshow("입력 영상", image)
cv2.imshow("srcHist", hist_image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

히스토그램의 빈도수는 실제 값이 매우 클 수 있다.

따라서 그대로 그래프를 그리면 화면 높이를 벗어날 수 있다.

이를 해결하기 위해 `cv2.normalize()`를 사용하여 히스토그램 값을 `0~hist_h` 범위로 정규화한다.

```python
cv2.normalize(hist, hist_norm, 0, hist_h, cv2.NORM_MINMAX)
```

그 후 `i=0`부터 `i=254`까지 반복하면서 현재 픽셀값의 히스토그램 높이와 다음 픽셀값의 히스토그램 높이를 `cv2.line()`으로 연결한다.

```python
cv2.line(hist_image, (x1, y1), (x2, y2), 0)
```

이를 통해 막대그래프의 끝점을 이어 만든 선 형태의 히스토그램 그래프를 출력할 수 있다.
