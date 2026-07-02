# ch07-1 영역처리 실습과제

## 공통 실행 조건

모든 실습 코드는 이미지 파일이 Python 파일과 같은 폴더에 있다고 가정한다.

실습과제 1, 2, 4는 `lenna.bmp`를 사용한다.
실습과제 3은 교재 예제 흐름에 따라 `rose.bmp`를 사용한다.
만약 `rose.bmp`가 없다면 코드에서 파일명을 `lenna.bmp`로 바꾸어 실행해도 된다.

예시 폴더 구조는 다음과 같다.

```text
opencv-python/
├── ch07_1_ex1.py
├── ch07_1_ex2.py
├── ch07_1_ex3.py
├── ch07_1_ex4.py
├── lenna.bmp
└── rose.bmp
```

---

# 실습과제 1

## 문제

레나 영상에 랜덤 노이즈를 추가하고, 블러링을 수행하여 결과를 출력하라.

## 코드

```python
import numpy as np
import cv2

# 레나 영상을 그레이스케일로 읽기
image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("영상파일 읽기 오류")

# -----------------------------
# 1. 랜덤 노이즈 생성
# -----------------------------
# 평균 0, 표준편차 20인 가우시안 노이즈 생성
noise = np.zeros(image.shape, np.int16)
cv2.randn(noise, 0, 20)

# 원본 영상에 노이즈 추가
noisy = image.astype(np.int16) + noise

# 0~255 범위를 벗어나지 않도록 제한
noisy = np.clip(noisy, 0, 255).astype(np.uint8)

# 노이즈가 들어간 영상에 텍스트 표시
cv2.putText(
    noisy,
    "stddev = 20",
    (20, 40),
    cv2.FONT_HERSHEY_SIMPLEX,
    1,
    255,
    2
)

# -----------------------------
# 2. 블러링 수행
# -----------------------------
# 7x7 평균값 필터 마스크 생성
kernel = np.ones((7, 7), np.float32) / 49

# filter2D를 이용하여 블러링
blur = cv2.filter2D(noisy, -1, kernel)

# -----------------------------
# 3. 결과 출력
# -----------------------------
cv2.imshow("src", image)
cv2.imshow("dst", noisy)
cv2.imshow("dst1", blur)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

블러링은 영상을 부드럽게 만드는 필터링 기법이다.
주변 픽셀들의 평균값을 이용하기 때문에 영상의 급격한 픽셀 변화가 줄어들고, 노이즈가 완화된다.

이 코드에서는 먼저 `cv2.randn()`을 이용해 평균이 `0`, 표준편차가 `20`인 랜덤 노이즈를 만든다.

```python
cv2.randn(noise, 0, 20)
```

그 후 원본 영상에 노이즈를 더하고, `np.clip()`을 이용해 픽셀값이 `0~255` 범위를 벗어나지 않도록 제한한다.

```python
noisy = np.clip(noisy, 0, 255).astype(np.uint8)
```

블러링은 교재에서 사용한 방식처럼 평균값 필터 마스크를 만들고 `cv2.filter2D()`를 이용하여 수행한다.

```python
kernel = np.ones((7, 7), np.float32) / 49
blur = cv2.filter2D(noisy, -1, kernel)
```

블러링 결과는 노이즈가 줄어들지만, 영상 전체가 흐릿해진다.

---

# 실습과제 2

## 문제

레나 영상을 블러링 후 샤프닝을 수행하여 원본 영상이 복원되는지 확인하라.

레나 영상을 샤프닝 후 블러링을 수행하여 원본 영상이 복원되는지 결과를 비교하라.

## 코드

```python
import numpy as np
import cv2

# 레나 영상을 그레이스케일로 읽기
image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("영상파일 읽기 오류")

# -----------------------------
# 1. 블러링 마스크
# -----------------------------
blur_kernel = np.ones((5, 5), np.float32) / 25

# -----------------------------
# 2. 샤프닝 마스크
# -----------------------------
sharp_kernel = np.array([
    [-1, -1, -1],
    [-1,  9, -1],
    [-1, -1, -1]
], dtype=np.float32)

# -----------------------------
# 3. 블러링 후 샤프닝
# -----------------------------
blur = cv2.filter2D(image, -1, blur_kernel)
blur_then_sharp = cv2.filter2D(blur, -1, sharp_kernel)

# -----------------------------
# 4. 샤프닝 후 블러링
# -----------------------------
sharp = cv2.filter2D(image, -1, sharp_kernel)
sharp_then_blur = cv2.filter2D(sharp, -1, blur_kernel)

# -----------------------------
# 5. 원본과의 차이 계산
# -----------------------------
diff1 = np.abs(image.astype(np.int16) - blur_then_sharp.astype(np.int16))
diff2 = np.abs(image.astype(np.int16) - sharp_then_blur.astype(np.int16))

print("블러링 후 샤프닝 평균 차이:", np.mean(diff1))
print("샤프닝 후 블러링 평균 차이:", np.mean(diff2))

# -----------------------------
# 6. 결과 출력
# -----------------------------
cv2.imshow("image", image)
cv2.imshow("blur", blur)
cv2.imshow("sharp", sharp)
cv2.imshow("blur_then_sharp", blur_then_sharp)
cv2.imshow("sharp_then_blur", sharp_then_blur)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

블러링은 주변 픽셀의 평균값으로 영상을 부드럽게 만드는 처리이다.
따라서 블러링을 수행하면 경계선과 세부 정보가 흐려진다.

샤프닝은 주변 화소와의 차이를 크게 만들어 영상을 날카롭게 만드는 처리이다.
하지만 블러링 과정에서 이미 사라진 세부 정보를 완전히 복원할 수는 없다.

따라서 `블러링 후 샤프닝`을 수행해도 원본 영상이 완전히 복원되지는 않는다.

또한 `샤프닝 후 블러링`은 먼저 경계와 노이즈를 강조한 뒤 다시 평균값 필터로 부드럽게 만드는 과정이다.
이 경우에도 원본과 완전히 같아지지는 않는다.

정리하면 다음과 같다.

| 처리 순서     | 결과                                |
| --------- | --------------------------------- |
| 블러링 후 샤프닝 | 흐려진 세부 정보가 일부 강조되지만 원본 완전 복원은 불가능 |
| 샤프닝 후 블러링 | 강조된 경계와 노이즈가 다시 완화되지만 원본과는 다름     |

즉, 블러링과 샤프닝은 서로 반대 효과를 가지지만, 완전히 역연산 관계는 아니다.
그래서 순서를 바꾸어 적용해도 원본 영상이 그대로 복원되지는 않는다.

---

# 실습과제 3

## 문제

엠보싱 마스크를 아래 마스크로 바꾼 후 결과를 출력하라.

매개변수 `delta`를 `0` 또는 `128`로 설정하라.

아래 마스크는 어떤 역할을 수행하는가?

```text
-1   0   1
-2   0   2
-1   0   1
```

## 코드

```python
import numpy as np
import cv2

# 교재 엠보싱 예제 기준으로 rose.bmp 사용
image = cv2.imread("rose.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("영상파일 읽기 오류")

# -----------------------------
# 실습과제에서 제시한 마스크
# -----------------------------
kernel = np.array([
    [-1, 0, 1],
    [-2, 0, 2],
    [-1, 0, 1]
], dtype=np.float32)

# delta = 0인 경우
dst_delta_0 = cv2.filter2D(image, -1, kernel, delta=0)

# delta = 128인 경우
dst_delta_128 = cv2.filter2D(image, -1, kernel, delta=128)

# 결과 출력
cv2.imshow("image", image)
cv2.imshow("delta_0", dst_delta_0)
cv2.imshow("delta_128", dst_delta_128)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

이 마스크는 일반적인 엠보싱 마스크라기보다, 좌우 방향의 밝기 변화량을 강조하는 마스크이다.

```text
-1   0   1
-2   0   2
-1   0   1
```

왼쪽 열에는 음수, 오른쪽 열에는 양수가 배치되어 있다.
따라서 영상에서 왼쪽과 오른쪽 픽셀값의 차이가 클수록 결과값이 크게 나온다.

즉, 이 마스크는 다음 역할을 한다.

* 좌우 방향의 밝기 변화 검출
* 세로 방향 경계선 강조
* x방향 미분 필터 역할
* Sobel X 필터와 같은 형태

`delta=0`으로 설정하면 필터링 결과에서 음수값이 포화연산에 의해 `0`으로 처리된다.
따라서 한 방향의 경계만 강하게 보이고, 반대 방향의 경계는 잘 보이지 않을 수 있다.

`delta=128`로 설정하면 결과값 전체에 `128`을 더한다.
그러면 변화가 거의 없는 평탄한 영역은 `128` 근처의 회색으로 표현되고, 밝기 변화가 큰 경계는 흰색 또는 검은색에 가깝게 표현된다.

따라서 경계 검출 결과를 보기 좋게 확인하려면 `delta=128`을 사용하는 것이 더 적절하다.

---

# 실습과제 4

## 문제

`lenna` 영상을 그레이로 변환하여 출력하고, 마우스 이벤트를 이용하여 마우스로 드래깅한 영역만 블러링을 수행하는 프로그램을 작성하시오.

## 코드

```python
import numpy as np
import cv2

# 레나 영상을 그레이스케일로 읽기
image = cv2.imread("lenna.bmp", cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("영상파일 읽기 오류")

title = "src"

# 출력용 영상
dst = image.copy()

# 마우스 드래그 상태 저장 변수
dragging = False
start_x = 0
start_y = 0

# 블러링 마스크
kernel = np.ones((15, 15), np.float32) / 225


def onMouse(event, x, y, flags, param):
    global dst, dragging, start_x, start_y

    # 왼쪽 버튼을 누르면 시작 좌표 저장
    if event == cv2.EVENT_LBUTTONDOWN:
        dragging = True
        start_x = x
        start_y = y

    # 왼쪽 버튼을 떼면 드래그 영역 계산 후 블러링
    elif event == cv2.EVENT_LBUTTONUP:
        dragging = False

        end_x = x
        end_y = y

        x1 = min(start_x, end_x)
        x2 = max(start_x, end_x)
        y1 = min(start_y, end_y)
        y2 = max(start_y, end_y)

        # 영역 크기가 0이면 처리하지 않음
        if x1 == x2 or y1 == y2:
            return

        # 드래그한 영역만 부분행렬로 추출
        roi = dst[y1:y2, x1:x2]

        # 해당 영역만 블러링
        blur_roi = cv2.filter2D(roi, -1, kernel)

        # 블러링 결과를 원래 위치에 삽입
        dst[y1:y2, x1:x2] = blur_roi

        cv2.imshow(title, dst)


cv2.imshow(title, dst)
cv2.setMouseCallback(title, onMouse)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

## 설명

먼저 레나 영상을 그레이스케일로 읽고, 결과를 저장할 `dst` 영상을 복사한다.

```python
dst = image.copy()
```

마우스 왼쪽 버튼을 누르면 시작 좌표를 저장한다.

```python
start_x = x
start_y = y
```

마우스 왼쪽 버튼을 떼면 시작 좌표와 끝 좌표를 이용해 드래그한 영역을 계산한다.

```python
x1 = min(start_x, end_x)
x2 = max(start_x, end_x)
y1 = min(start_y, end_y)
y2 = max(start_y, end_y)
```

그 후 해당 영역만 부분행렬로 추출한다.

```python
roi = dst[y1:y2, x1:x2]
```

그리고 해당 영역에만 블러링 마스크를 적용한다.

```python
blur_roi = cv2.filter2D(roi, -1, kernel)
```

마지막으로 블러링된 영역을 원래 영상의 같은 위치에 다시 넣는다.

```python
dst[y1:y2, x1:x2] = blur_roi
```

이렇게 하면 마우스로 드래그한 영역만 흐릿하게 처리된다.

---

영역처리는 현재 픽셀 하나만 보는 것이 아니라 주변 픽셀까지 함께 고려한다.
따라서 블러링, 샤프닝, 엠보싱, 에지 검출처럼 영상의 공간적인 특징을 다루는 처리에 사용된다.
