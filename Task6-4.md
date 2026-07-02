# ch06-4 화소처리 실습과제

## 공통 실행 조건

모든 실습 코드는 `crayfish.jpg` 파일이 Python 파일과 같은 폴더에 있다고 가정한다.

예시 폴더 구조는 다음과 같다.

```text
opencv-python/
├── ch06_4_ex1.md
├── ch06_4_ex2.py
├── ch06_4_ex3.py
├── ch06_4_ex4.py
└── crayfish.jpg
```

---

# 실습과제 1

## 문제

히스토그램 스트레칭과 히스토그램 평활화의 차이를 설명하라.

## 답안

히스토그램 스트레칭과 히스토그램 평활화는 모두 영상의 명암비를 개선하기 위한 화소처리 방법이다. 두 방법 모두 픽셀값의 분포를 더 넓은 범위로 퍼뜨려 영상이 더 선명하게 보이도록 만든다는 공통점이 있다.

하지만 두 방법은 동작 방식이 다르다.

---

## 1. 히스토그램 스트레칭

히스토그램 스트레칭은 영상의 픽셀값 분포가 그레이스케일 전체 영역에 걸쳐 나타나도록 변환하는 기법이다. 명암비가 낮은 영상은 히스토그램이 특정 구간에 집중되어 있는데, 이를 고무줄을 잡아 늘이듯이 펼쳐서 픽셀값이 `0~255` 전 구간에 분포하도록 만든다.

히스토그램 스트레칭에서는 입력 영상의 최소 픽셀값 `Gmin`과 최대 픽셀값 `Gmax`를 구한 뒤, `Gmin`은 `0`, `Gmax`는 `255`가 되도록 선형 변환을 수행한다.

즉, 다음 두 점을 지나는 직선 변환을 사용한다.

```text
(Gmin, 0)
(Gmax, 255)
```

변환식은 다음과 같다.

```text
Y = saturate(255 / (Gmax - Gmin) × (X - Gmin))
```

이 방법은 선형 변환이므로 계산이 비교적 단순하다.
그러나 전체 픽셀 분포를 단순히 최소값과 최대값 기준으로 늘리는 방식이기 때문에, 특정 밝기 구간에 픽셀이 많이 몰려 있는 경우에는 세밀한 분포 조절에는 한계가 있다.

---

## 2. 히스토그램 평활화

히스토그램 평활화는 히스토그램의 누적분포함수, 즉 CDF를 이용하여 픽셀값을 변환하는 방법이다.

히스토그램 평활화는 특정 그레이스케일 값 근처에 픽셀 분포가 많이 뭉쳐 있을 때, 그 부분을 더 넓게 펼쳐 픽셀값 분포를 조절한다. 즉, 빈도수가 많은 구간은 더 넓게 스트레칭하고, 빈도수가 적은 구간은 상대적으로 적게 스트레칭한다.

히스토그램을 `h(g)`라고 하면, `h(g)`는 픽셀값이 `g`인 픽셀의 개수를 의미한다.
히스토그램 누적 함수 `H(g)`는 다음과 같이 정의된다.

```text
H(g) = h(0) + h(1) + ... + h(g)
```

히스토그램 평활화에서는 이 누적 함수 `H(g)`를 픽셀값 변환 함수로 사용한다. 단, `H(g)`의 값은 전체 픽셀 수까지 커질 수 있으므로, 결과값이 `0~255` 범위에 들어오도록 정규화해야 한다.

그레이스케일 영상에서 최대 픽셀값은 `255`이므로, 변환식은 다음과 같이 나타낼 수 있다.

```text
Y = round(H(X) × 255 / N)
```

여기서 `N`은 영상의 전체 픽셀 수이다.

---

## 3. 차이 정리

| 구분        | 히스토그램 스트레칭              | 히스토그램 평활화               |
| --------- | ----------------------- | ----------------------- |
| 변환 방식     | 최소값과 최대값을 기준으로 선형 변환    | 히스토그램 누적함수 기반 비선형 변환    |
| 기준값       | `Gmin`, `Gmax`          | 누적 히스토그램 `H(g)`         |
| 결과        | 픽셀값 범위를 `0~255`로 확장     | 픽셀 분포가 더 고르게 퍼짐         |
| 장점        | 단순하고 빠름                 | 분포가 몰린 영역을 효과적으로 개선     |
| 단점        | 픽셀 분포 자체를 세밀하게 조절하지는 못함 | 결과가 과도하게 강해질 수 있음       |
| OpenCV 함수 | 직접 구현 필요                | `cv2.equalizeHist()` 제공 |

---

## 최종 정리

히스토그램 스트레칭은 영상의 최소 픽셀값과 최대 픽셀값을 기준으로 픽셀값 범위를 선형적으로 넓히는 방법이다.

반면 히스토그램 평활화는 히스토그램 누적함수를 이용하여 픽셀 분포가 전체 그레이스케일 영역에 더 고르게 나타나도록 조절하는 방법이다.

따라서 히스토그램 스트레칭은 단순한 명암비 확장에 적합하고, 히스토그램 평활화는 픽셀값이 특정 구간에 몰려 있는 영상을 더 적극적으로 개선하는 데 적합하다.

---

# 실습과제 2

## 문제

`crayfish.jpg` 영상의 히스토그램 스트레칭을 수행한 후 입력, 출력 영상을 출력하라.

히스토그램 스트레칭 전, 후의 히스토그램 그래프를 그려라.

## 코드

```python
import numpy as np
import cv2


def calcGrayHist(image):
    images = [image]
    channels = [0]
    hsize = [256]
    ranges = [0, 256]

    hist = cv2.calcHist(images, channels, None, hsize, ranges)

    return hist


def drawGrayHistImage(hist):
    win_shape = (100, 256)
    hist_img = np.full(win_shape, 255, np.uint8)

    cv2.normalize(hist, hist, 0, win_shape[0], cv2.NORM_MINMAX)

    gap = hist_img.shape[1] / hist.shape[0]

    for i, h in enumerate(hist.flatten()):
        x = int(round(i * gap))
        cv2.line(hist_img, (x, 0), (x, int(h)), 0, 1)

    return cv2.flip(hist_img, 0)


def saturate(value):
    if value > 255:
        return 255
    elif value < 0:
        return 0
    else:
        return value


image = cv2.imread("crayfish.jpg", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

# 입력 영상의 최소값, 최대값 구하기
minVal, maxVal, minLoc, maxLoc = cv2.minMaxLoc(image)

print("Gmin:", minVal)
print("Gmax:", maxVal)

# 히스토그램 스트레칭 계수 계산
alpha = 255 / (maxVal - minVal)
beta = -minVal * alpha

# 반복문을 이용한 히스토그램 스트레칭
dst = np.zeros(image.shape, image.dtype)

for y in range(image.shape[0]):
    for x in range(image.shape[1]):
        tmp = alpha * int(image[y, x]) + beta
        dst[y, x] = saturate(tmp)

# 히스토그램 계산
src_hist = calcGrayHist(image)
dst_hist = calcGrayHist(dst)

# 히스토그램 영상 생성
src_hist_image = drawGrayHistImage(src_hist)
dst_hist_image = drawGrayHistImage(dst_hist)

# 결과 출력
cv2.imshow("src", image)
cv2.imshow("dst", dst)
cv2.imshow("srcHist", src_hist_image)
cv2.imshow("dstHist", dst_hist_image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

히스토그램 스트레칭은 입력 영상의 최소 픽셀값 `Gmin`과 최대 픽셀값 `Gmax`를 구한 뒤, 이 범위를 `0~255` 범위로 늘리는 방법이다.

```python
minVal, maxVal, minLoc, maxLoc = cv2.minMaxLoc(image)
```

위 코드로 입력 영상의 최소값과 최대값을 구한다.

그 후 변환식의 계수 `alpha`, `beta`를 계산한다.

```python
alpha = 255 / (maxVal - minVal)
beta = -minVal * alpha
```

각 픽셀에 대해 다음 식을 적용한다.

```text
Y = alpha × X + beta
```

이때 결과값이 `0`보다 작거나 `255`보다 커질 수 있으므로 `saturate()` 함수를 이용하여 픽셀값을 `0~255` 범위로 제한한다.

```python
dst[y, x] = saturate(tmp)
```

히스토그램 스트레칭 후에는 입력 영상의 좁은 픽셀값 범위가 넓어지므로, 출력 영상의 명암비가 증가한다.

---

# 실습과제 3

## 문제

`crayfish.jpg` 영상의 히스토그램 평활화를 수행한 후 입력, 출력 영상을 출력하라.

히스토그램 평활화 전, 후의 히스토그램 그래프를 그려라.

## 코드

```python
import numpy as np
import cv2


def calcGrayHist(image):
    images = [image]
    channels = [0]
    hsize = [256]
    ranges = [0, 256]

    hist = cv2.calcHist(images, channels, None, hsize, ranges)

    return hist


def drawGrayHistImage(hist):
    win_shape = (100, 256)
    hist_img = np.full(win_shape, 255, np.uint8)

    cv2.normalize(hist, hist, 0, win_shape[0], cv2.NORM_MINMAX)

    gap = hist_img.shape[1] / hist.shape[0]

    for i, h in enumerate(hist.flatten()):
        x = int(round(i * gap))
        cv2.line(hist_img, (x, 0), (x, int(h)), 0, 1)

    return cv2.flip(hist_img, 0)


image = cv2.imread("crayfish.jpg", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

# 히스토그램 평활화
dst = cv2.equalizeHist(image)

# 히스토그램 계산
src_hist = calcGrayHist(image)
dst_hist = calcGrayHist(dst)

# 히스토그램 영상 생성
src_hist_image = drawGrayHistImage(src_hist)
dst_hist_image = drawGrayHistImage(dst_hist)

# 결과 출력
cv2.imshow("src", image)
cv2.imshow("dst", dst)
cv2.imshow("srcHist", src_hist_image)
cv2.imshow("dstHist", dst_hist_image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

히스토그램 평활화는 특정 픽셀값 근처에 몰려 있는 픽셀 분포를 더 넓게 펼쳐주는 방법이다.

OpenCV에서는 그레이스케일 영상에 대해 히스토그램 평활화를 수행하는 함수 `cv2.equalizeHist()`를 제공한다.

```python
dst = cv2.equalizeHist(image)
```

이 함수는 8비트 1채널 영상, 즉 일반적인 그레이스케일 영상에 적용할 수 있다.

히스토그램 평활화 후에는 픽셀값 분포가 더 넓게 퍼지고, 영상의 명암비가 개선된다. 다만 영상에 따라 대비가 강해져 거칠게 보일 수 있다.

---

# 실습과제 4

## 문제

`crayfish.jpg` 영상의 히스토그램 누적함수의 그래프를 그리시오.

주의: 히스토그램 객체의 타입은 `float32`이다.

## 코드

```python
import numpy as np
import cv2


def calcGrayHist(image):
    images = [image]
    channels = [0]
    hsize = [256]
    ranges = [0, 256]

    hist = cv2.calcHist(images, channels, None, hsize, ranges)

    return hist


def drawGrayHistImage(hist):
    win_shape = (100, 256)
    hist_img = np.full(win_shape, 255, np.uint8)

    cv2.normalize(hist, hist, 0, win_shape[0], cv2.NORM_MINMAX)

    gap = hist_img.shape[1] / hist.shape[0]

    for i, h in enumerate(hist.flatten()):
        x = int(round(i * gap))
        cv2.line(hist_img, (x, 0), (x, int(h)), 0, 1)

    return cv2.flip(hist_img, 0)


def calcCumulativeHist(hist):
    cdf = np.zeros(hist.shape, np.float32)

    total = 0.0

    for i in range(hist.shape[0]):
        total += hist[i, 0]
        cdf[i, 0] = total

    return cdf


def drawCumulativeHistImage(cdf):
    win_shape = (100, 256)
    cdf_img = np.full(win_shape, 255, np.uint8)

    cdf_norm = cdf.copy()
    cv2.normalize(cdf_norm, cdf_norm, 0, win_shape[0], cv2.NORM_MINMAX)

    gap = cdf_img.shape[1] / cdf_norm.shape[0]

    for i, h in enumerate(cdf_norm.flatten()):
        x = int(round(i * gap))
        cv2.line(cdf_img, (x, 0), (x, int(h)), 0, 1)

    return cv2.flip(cdf_img, 0)


image = cv2.imread("crayfish.jpg", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

# 히스토그램 계산
hist = calcGrayHist(image)

# 히스토그램 누적함수 계산
cdf = calcCumulativeHist(hist)

print("hist dtype:", hist.dtype)
print("cdf dtype:", cdf.dtype)

# 그래프 생성
hist_image = drawGrayHistImage(hist)
cdf_image = drawCumulativeHistImage(cdf)

# 결과 출력
cv2.imshow("src", image)
cv2.imshow("srcHist", hist_image)
cv2.imshow("CDF", cdf_image)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

히스토그램은 각 픽셀값이 영상에 몇 번 등장하는지를 나타낸다.

```text
h(g) = 픽셀값이 g인 픽셀의 개수
```

히스토그램 누적함수는 현재 픽셀값까지의 히스토그램 값을 모두 더한 것이다.

```text
H(g) = h(0) + h(1) + ... + h(g)
```

코드에서는 다음 부분이 누적합을 계산하는 역할을 한다.

```python
total += hist[i, 0]
cdf[i, 0] = total
```

히스토그램 객체의 타입은 `float32`이므로, 누적함수 배열도 `np.float32` 타입으로 생성한다.

```python
cdf = np.zeros(hist.shape, np.float32)
```

이후 `cv2.normalize()`를 사용하여 누적함수 값을 화면 높이 범위로 정규화하고, `cv2.line()`을 이용해 그래프를 그린다.

---
