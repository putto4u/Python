`eval()` 함수는 문자열로 된 파이썬 코드를 실행하여 그 결과를 반환하는 내장 함수입니다. 보안상 매우 위험할 수 있는 강력한 도구이므로, 정확한 동작 원리와 위험성을 이해하는 것이 중요합니다.

---

## 1. 기본 개념

`eval(expression, globals=None, locals=None)`은 문자열(expression)을 파이썬 코드로 해석하여 실행한 뒤, 그 **결괏값을 반환**합니다.

```python
# 문자열 계산식 실행
result = eval("10 + 20")
print(result)  # 출력: 30

# 변수가 포함된 식 실행
x = 10
print(eval("x * 5"))  # 출력: 50

```

---

## 2. 주의사항 및 실전 팁

### ⚠️ 보안 위험 (Code Injection)

`eval()`의 가장 큰 문제는 **사용자로부터 입력받은 문자열을 그대로 실행할 때** 발생합니다. 악의적인 사용자가 시스템 명령어를 입력하면 그대로 실행될 위험이 있습니다.

* **위험 예시:** `__import__('os').system('rm -rf /')` 같은 문자열이 입력되면 시스템 전체가 삭제될 수 있습니다.
* **해결책:** 단순 산술 연산이나 데이터 변환이 목적이라면 `ast.literal_eval()`을 사용하는 것이 훨씬 안전합니다.

### 💡 `eval()` vs `exec()`

| 구분 | `eval()` | `exec()` |
| --- | --- | --- |
| **목적** | 식(Expression)을 실행하고 **결과 반환** | 문(Statement)을 실행하며 **반환값 없음** |
| **예시** | `eval("1 + 1")` -> 2 | `exec("a = 1 + 1")` -> None (변수 a에 2 할당) |

---

## 3. 자주 오해하는 부분

많은 개발자가 `eval()`을 사용해 문자열 형태의 리스트나 딕셔너리를 객체로 변환하려 합니다.

```python
# 나쁜 예 (보안 취약)
data = eval("[1, 2, 3]") 

# 좋은 예 (안전한 데이터 파싱)
import ast
data = ast.literal_eval("[1, 2, 3]")

```

`ast.literal_eval()`은 파이썬의 기본 자료형(리스트, 딕셔너리, 튜플 등)만 해석하며, 시스템 명령어나 함수 호출은 실행하지 않으므로 훨씬 권장됩니다.

---

## 4. 실전 활용 예제

동적인 수식을 계산해야 하는 대시보드나 설정 파일의 값을 기반으로 연산을 수행할 때 유용합니다.

```python
def dynamic_calc(a, b, operator):
    # 연산자를 문자열로 받아 즉석에서 계산
    expression = f"{a} {operator} {b}"
    return eval(expression)

print(dynamic_calc(10, 5, '*'))  # 출력: 50

```

> **참고 문서:** [Python 공식 문서 - eval()](https://www.google.com/search?q=https://docs.python.org/3/library/functions.html%23eval)

---

`eval()`의 강력한 기능은 유지하면서, 보안상의 치명적인 단점을 해결하기 위해 사용하는 것이 바로 `ast.literal_eval()`입니다. 여기서 **ast**는 **Abstract Syntax Tree**(추상 구문 트리)의 약자입니다.

---

## 1. ast.literal_eval()이란?

이 함수는 문자열을 파이썬의 **기본 자료형(Literal)**으로만 변환해 주는 안전한 함수입니다. 문자열 내에 함수 호출이나 시스템 명령이 포함되어 있으면 실행하지 않고 에러를 발생시킵니다.

### 안전한 변환 대상 (Literals)

* **Strings** (문자열)
* **Numbers** (숫자: 정수, 실수, 복소수)
* **Tuples, Lists, Dicts** (컨테이너 자료형)
* **Booleans** (True, False)
* **None**

---

## 2. 왜 `eval()` 대신 써야 할까? (실전 비교)

가장 큰 차이점은 **"실행 권한의 범위"**입니다.

### ❌ eval()의 위험성

```python
import os
user_input = "__import__('os').system('ls')" # 시스템 파일 목록을 보는 공격 코드
eval(user_input) # 그대로 실행됨 (보안 사고)

```

### ✅ ast.literal_eval()의 안전성

```python
import ast
user_input = "__import__('os').system('ls')"
try:
    ast.literal_eval(user_input)
except ValueError:
    print("위험한 입력이 감지되었습니다!") # 실행되지 않고 안전하게 에러 처리

```

---

## 3. 주요 활용 사례

주로 외부 텍스트 파일이나 데이터베이스에서 **파이썬 객체 형태의 문자열**을 읽어와서 실제 객체로 복구할 때 사용합니다.

```python
import ast

# 1. 텍스트로 된 리스트 복구
list_str = "[1, 2, 3, {'key': 'value'}]"
real_list = ast.literal_eval(list_str)
print(real_list[3]['key'])  # 출력: value

# 2. 튜플 및 불리언 복구
tuple_str = "(True, False, None)"
real_tuple = ast.literal_eval(tuple_str)
print(real_tuple[0])  # 출력: True

```

---

## 4. 자주 하는 실수 (Pitfalls)

* **수식 계산 불가:** `ast.literal_eval("1 + 1")`은 에러가 발생할 수 있습니다. 이는 '값(Literal)'이 아니라 '연산(Operation)'이기 때문입니다. (파이썬 버전에 따라 동작이 다를 수 있으나, 기본적으로 단순 값 변환용입니다.)
* **변수 참조 불가:** `x = 10; ast.literal_eval("x")`는 불가능합니다. 오직 고정된 값(Literal)만 해석합니다.

---

### 요약 테이블

| 기능 | `eval()` | `ast.literal_eval()` |
| --- | --- | --- |
| **실행 범위** | 모든 파이썬 코드 (함수, 클래스 포함) | 기본 자료형(Literal)만 |
| **보안성** | **매우 위험** (Code Injection 취약) | **안전함** |
| **용도** | 동적 코드 실행 | 문자열 데이터의 객체화 |

> **출처:** [Python Official Docs - ast.literal_eval](https://www.google.com/search?q=https://docs.python.org/3/library/ast.html%23ast.literal_eval)

---

혹시 실제 프로젝트에서 특정 텍스트 데이터를 파이썬 객체로 변환해야 하는 상황인가요? 해당 데이터의 샘플을 주시면 적합한 파싱 방법을 제안해 드릴 수 있습니다.
