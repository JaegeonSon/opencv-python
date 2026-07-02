# ch06-2 화소처리 실습과제

## 공통 실행 조건

모든 실습 코드는 `lenna.bmp` 파일이 Python 파일과 같은 폴더에 있다고 가정한다.

예시 폴더 구조는 다음과 같다.

```text
opencv-python/
├── ch06_2_ex1.py
├── ch06_2_ex2.py
├── ch06_2_ex3.py
└── lenna.bmp
```

---

## 실습과제 1

### 문제

다음을 수행하시오.

1. `lenna.bmp` 영상을 그레이스케일로 변환 후 픽셀값의 최대, 최소값을 구하시오.
2. 예제의 기본적인 명암비 조절 후 픽셀값의 최대, 최소값을 구하시오.
3. 예제의 효과적인 명암비 조절 후 픽셀값의 최대, 최소값을 구하시오.
4. 3가지 경우의 명암비를 계산하고 개선효과를 설명하시오.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def saturate(value):
    if value > 255:
        return 255
    elif value < 0:
        return 0
    else:
        return value


def basic_contrast(src, scale):
    dst = np.zeros(src.shape, src.dtype)

    for y in range(src.shape[0]):
        for x in range(src.shape[1]):
            tmp = scale * int(src[y, x])
            dst[y, x] = saturate(tmp)

    return dst


def effective_contrast(src, alpha):
    beta = -128 * alpha + 128
    dst = np.zeros(src.shape, src.dtype)

    for y in range(src.shape[0]):
        for x in range(src.shape[1]):
            tmp = alpha * int(src[y, x]) + beta
            dst[y, x] = saturate(tmp)

    return dst


min_val, max_val, min_loc, max_loc = cv2.minMaxLoc(image)
contrast_original = max_val - min_val

basic = basic_contrast(image, 2)
basic_min, basic_max, _, _ = cv2.minMaxLoc(basic)
contrast_basic = basic_max - basic_min

effective = effective_contrast(image, 2)
effective_min, effective_max, _, _ = cv2.minMaxLoc(effective)
contrast_effective = effective_max - effective_min

print("원본 영상")
print("최소값:", min_val)
print("최대값:", max_val)
print("명암비:", contrast_original)

print("\n기본적인 명암비 조절")
print("최소값:", basic_min)
print("최대값:", basic_max)
print("명암비:", contrast_basic)

print("\n효과적인 명암비 조절")
print("최소값:", effective_min)
print("최대값:", effective_max)
print("명암비:", contrast_effective)

cv2.imshow("original", image)
cv2.imshow("basic contrast", basic)
cv2.imshow("effective contrast", effective)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

명암비는 영상에서 가장 밝은 값과 가장 어두운 값의 차이이다.

```text
명암비 = 최대 픽셀값 - 최소 픽셀값
```

기본적인 명암비 조절은 모든 픽셀값에 일정한 배율을 곱하는 방식이다.

```text
Y = saturate(sX)
```

이 방식은 밝은 영역이 쉽게 `255`로 포화되어 세부 정보가 사라질 수 있다.

효과적인 명암비 조절은 그레이스케일 중간값인 `128`을 기준으로 밝은 픽셀은 더 밝게 만들고, 어두운 픽셀은 더 어둡게 만드는 방식이다.

```text
Y = saturate(alpha * (X - 128) + 128)
```

따라서 단순 곱셈 방식보다 자연스럽게 명암비를 개선할 수 있다.

---

## 실습과제 2

### 문제

예제2에서 `alpha`가 너무 크면 어떻게 되는가? 실행결과를 첨부하고 이유를 설명하라.

예제2에서 `alpha`가 너무 작으면 어떻게 되는가? 실행결과를 첨부하고 이유를 설명하라.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def saturate(value):
    if value > 255:
        return 255
    elif value < 0:
        return 0
    else:
        return value


def effective_contrast(src, alpha):
    beta = -128 * alpha + 128
    dst = np.zeros(src.shape, src.dtype)

    for y in range(src.shape[0]):
        for x in range(src.shape[1]):
            tmp = alpha * int(src[y, x]) + beta
            dst[y, x] = saturate(tmp)

    return dst


alpha_big = 5.0
alpha_small = 0.2

dst_big = effective_contrast(image, alpha_big)
dst_small = effective_contrast(image, alpha_small)

cv2.imshow("original", image)
cv2.imshow("alpha_big", dst_big)
cv2.imshow("alpha_small", dst_small)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 실행결과 및 설명

`alpha`가 너무 크면 변환 함수의 기울기가 매우 커진다.

이 경우 `128`보다 작은 픽셀값은 빠르게 `0`에 가까워지고, `128`보다 큰 픽셀값은 빠르게 `255`에 가까워진다.

따라서 결과 영상은 검은색과 흰색 영역이 과도하게 강해지고, 많은 픽셀이 `0` 또는 `255`로 포화된다. 이로 인해 명암비는 강해지지만 세부 정보가 손실될 수 있다.

`alpha`가 너무 작으면 변환 함수의 기울기가 매우 작아진다.

이 경우 픽셀값들이 `128` 근처로 모이게 된다. 따라서 전체 영상이 회색에 가까워지고, 밝은 부분과 어두운 부분의 차이가 줄어든다.

결과적으로 명암비가 낮아져 영상이 흐릿하고 밋밋하게 보인다.

| 경우             | 결과                                   |
| -------------- | ------------------------------------ |
| `alpha`가 너무 큼  | 명암비 과도 증가, 0과 255 포화 영역 증가, 세부 정보 손실 |
| `alpha`가 너무 작음 | 명암비 감소, 픽셀값이 128 근처로 모임, 영상이 흐릿해짐    |

---

## 실습과제 3

### 문제

`lenna.bmp` 영상을 그레이스케일로 변환 후 다음 그래프를 이용하여 명암비를 조절하고 결과 영상을 출력하시오.

OpenCV 함수를 사용하지 말고 화소값을 직접 접근하는 방식으로 코드를 작성하시오.

### 그래프 해석

그래프는 다음 점들을 기준으로 하는 구간별 선형 변환으로 해석할 수 있다.

```text
(0, 50)
(128, 128)
(200, 255)
(255, 255)
```

즉, 입력 픽셀값이 `0`일 때 출력은 `50`이고, 입력 픽셀값이 `128`일 때 출력은 `128`이다.

입력 픽셀값이 `200` 이상이면 출력은 `255`로 포화된다.

### 코드

```python
import numpy as np
import cv2

image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")


def saturate(value):
    if value > 255:
        return 255
    elif value < 0:
        return 0
    else:
        return value


def transform_pixel(x):
    if x <= 128:
        y = 50 + (128 - 50) / 128 * x

    elif x <= 200:
        y = 128 + (255 - 128) / (200 - 128) * (x - 128)

    else:
        y = 255

    return saturate(int(round(y)))


dst = np.zeros(image.shape, image.dtype)

for y in range(image.shape[0]):
    for x in range(image.shape[1]):
        src_value = int(image[y, x])
        dst[y, x] = transform_pixel(src_value)

cv2.imshow("original", image)
cv2.imshow("result", dst)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 설명

주어진 그래프는 입력 픽셀값 `X`를 출력 픽셀값 `Y`로 변환하는 함수이다.

낮은 픽셀값 영역에서는 출력값이 `50`부터 시작하므로 어두운 영역이 약간 밝아진다.

중간 이후 영역에서는 기울기가 커지므로 밝은 영역이 더 밝아진다.

입력값이 `200` 이상이면 출력값은 `255`가 되어 포화된다.

OpenCV의 명암비 조절 함수를 사용하지 않고, 모든 화소값에 직접 접근하여 변환을 수행하였다.
