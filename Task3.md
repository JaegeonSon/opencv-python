# 3장 NumPy 실습과제

## 1. 용어 설명 및 Python 리스트와 NumPy ndarray 비교

### 1) 단위행렬, 역행렬, 전치행렬

단위행렬은 행렬 곱셈에서 1과 같은 역할을 하는 행렬이다.
즉, 어떤 행렬에 단위행렬을 곱해도 원래 행렬이 그대로 유지된다.

역행렬은 어떤 행렬과 곱했을 때 단위행렬이 되는 행렬이다.
즉, 행렬 연산에서 나눗셈과 비슷한 역할을 한다.

전치행렬은 행렬의 행과 열을 서로 바꾼 행렬이다.
예를 들어 2행 3열 행렬을 전치하면 3행 2열 행렬이 된다.

---

### 2) Python 리스트와 NumPy ndarray의 데이터 저장방식 비교

Python 리스트는 다양한 자료형을 저장할 수 있어 유연하다.
예를 들어 정수, 실수, 문자열, 논리값 등을 하나의 리스트에 함께 저장할 수 있다.

하지만 Python 리스트는 데이터가 참조형 구조로 저장된다.
즉, 실제 데이터 값이 연속적으로 저장되는 것이 아니라 각 데이터 객체의 주소를 저장하는 방식이다.
또한 연산을 수행할 때마다 데이터 타입 확인이 반복적으로 필요하기 때문에 대량의 숫자 계산에서는 속도가 느릴 수 있다.

반면 NumPy `ndarray`는 모든 원소가 같은 자료형을 가진다.
또한 데이터가 메모리에 연속적으로 저장된다.

NumPy는 C 언어 기반의 빠른 내부 연산, CPU 캐시 활용, SIMD와 같은 하드웨어 가속, 적은 메모리 오버헤드 덕분에 Python 리스트보다 빠른 처리 속도를 가진다.

---

## 2. 반복문을 이용한 행렬곱 직접 구현

다음 행렬 `A`, `B`의 행렬곱 `AB`를 라이브러리 함수를 사용하지 않고 반복문으로 직접 구현한다.

```python
A = [
    [1, 2, 3],
    [-1, 3, -10]
]

B = [
    [2, -1],
    [3, 1],
    [0, 5]
]

# A는 2x3, B는 3x2
# 결과 AB는 2x2

row_A = len(A)
col_A = len(A[0])
row_B = len(B)
col_B = len(B[0])

# 행렬곱이 가능한지 확인
if col_A != row_B:
    print("행렬곱을 계산할 수 없습니다.")
else:
    # 결과 행렬을 0으로 초기화
    AB = []

    for i in range(row_A):
        row = []

        for j in range(col_B):
            total = 0

            for k in range(col_A):
                total += A[i][k] * B[k][j]

            row.append(total)

        AB.append(row)

    print("AB =")
    for row in AB:
        print(row)
```

### 실행 결과

```text
AB =
[8, 16]
[7, -46]
```

---

## 3. NumPy를 이용한 행렬식 계산

다음 행렬 `A`에 대해 아래 식을 계산한다.

```text
A^3 - 5A^2 - 4A - 98I
```

여기서 `I`는 단위행렬이다.

```python
import numpy as np

A = np.array([
    [1, 2, 3],
    [10, -1, -5],
    [6, 7, 5]
])

I = np.eye(3, 3)

A2 = A @ A
A3 = A @ A @ A

result = A3 - 5 * A2 - 4 * A - 98 * I

print("A =")
print(A)

print("\nA^3 - 5A^2 - 4A - 98I =")
print(result)
```

### 실행 결과

```text
A =
[[  1   2   3]
 [ 10  -1  -5]
 [  6   7   5]]

A^3 - 5A^2 - 4A - 98I =
[[0. 0. 0.]
 [0. 0. 0.]
 [0. 0. 0.]]
```

---

## 4. NumPy를 이용한 연립방정식 풀이

다음 연립방정식을 NumPy를 이용하여 푼다.

```text
w + x + y + z = 10
2w - x + 2y - z = 3
3w + 2x - y + 2z = 11
w + 3x + 2y - z = 9
```

이를 행렬 형태로 표현하면 다음과 같다.

```text
AX = B
```

따라서 해는 다음과 같이 구할 수 있다.

```text
X = A^-1B
```

```python
import numpy as np

A = np.array([
    [1, 1, 1, 1],
    [2, -1, 2, -1],
    [3, 2, -1, 2],
    [1, 3, 2, -1]
])

B = np.array([10, 3, 11, 9])

A_inv = np.linalg.inv(A)

X = A_inv @ B

print("계수행렬 A =")
print(A)

print("\n상수배열 B =")
print(B)

print("\n역행렬 A_inv =")
print(A_inv)

print("\n연립방정식의 해 X =")
print(X)

w, x, y, z = X

print("\n결과")
print(f"w = {w}")
print(f"x = {x}")
print(f"y = {y}")
print(f"z = {z}")
```

### 실행 결과

```text
연립방정식의 해 X =
[1.         1.75       3.33333333 3.91666667]

결과
w = 1.0
x = 1.75
y = 3.3333333333333335
z = 3.9166666666666665
```

---

## 5. 레나 영상 밝기 증감 문제 해결

레나 영상의 밝기를 증가 또는 감소시킬 때, 단순히 픽셀값에 값을 더하거나 빼면 문제가 발생할 수 있다.

이미지 픽셀값은 일반적으로 `0~255` 범위를 가진다.
따라서 연산 결과가 `255`보다 크면 `255`를 저장하고, `0`보다 작으면 `0`을 저장해야 한다.

OpenCV 함수를 사용하지 않고 반복문을 이용하여 직접 처리한다.

```python
import cv2
import numpy as np

image = cv2.imread('lenna.bmp', cv2.IMREAD_GRAYSCALE)

if image is None:
    raise Exception("파일 읽기 오류")

height = image.shape[0]
width = image.shape[1]

bright = np.zeros((height, width), dtype=np.uint8)
dark = np.zeros((height, width), dtype=np.uint8)

value = 100

# 밝기 증가
for y in range(height):
    for x in range(width):
        pixel = int(image[y, x]) + value

        if pixel > 255:
            pixel = 255

        bright[y, x] = pixel

# 밝기 감소
for y in range(height):
    for x in range(width):
        pixel = int(image[y, x]) - value

        if pixel < 0:
            pixel = 0

        dark[y, x] = pixel

cv2.imshow('Original Image', image)
cv2.imshow('Bright Image', bright)
cv2.imshow('Dark Image', dark)

cv2.waitKey(0)
cv2.destroyAllWindows()
```

### 핵심 처리 방식

```python
if pixel > 255:
    pixel = 255
```

```python
if pixel < 0:
    pixel = 0
```

이를 통해 픽셀값이 이미지 표현 범위인 `0~255`를 벗어나지 않도록 제한할 수 있다.
