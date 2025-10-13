# **str** 메서드 정리

## \_\_str\_\_이란?

`__str__`은 객체를 사람이 보기 좋은 문자열로 바꾸는 메서드.
`print(obj)`나 `str(obj)`를 사용할 때 자동으로 호출됨

예시:

``` python
print(obj)   # 내부적으로 str(obj)를 호출함 → obj.__str__()
```

## 예시 코드

``` python
class BankAccount:
    def __init__(self, owner, balance):
        self.owner = owner
        self.balance = balance

    def __str__(self):
        return f"[계좌 소유자: {self.owner}, 잔액: {self.balance}원]"

account = BankAccount("Alice", 10000)
print(account)   # [계좌 소유자: Alice, 잔액: 10000원]
```

`print()` 안에서는 `__str__()`이 자동으로 호출됨.
직접 `account.__str__()`를 부를 필요는 없음.

## 주의할 점

-   `print()` 안에서 `return`이 아니라 `print()`를 쓰면 안 됨.
    (`__str__`은 문자열을 반환(return)해야 함.)
-   반드시 **문자열(str)** 을 반환해야 함.
-   `__str__` 안에서는 출력(print), 파일 저장, 데이터 수정 같은 부작용을
    만들지 않는 것이 좋음.

## **str** vs **repr**

  ----------------------------------------------------------------------------------------------------
  구분         목적         출력 형태                                     주로 쓰는 상황
  ------------ ------------ --------------------------------------------- ----------------------------
  `__str__`    사용자용     `[계좌 소유자: Alice, 잔액: 10000원]`         `print(obj)`
               (읽기 쉬움)                                                

  `__repr__`   개발자용     `BankAccount(owner='Alice', balance=10000)`   REPL, 로그 등
               (디버깅용)                                                 
  ----------------------------------------------------------------------------------------------------

보통 이렇게 같이 작성.

``` python
def __repr__(self):
    return f"BankAccount(owner={self.owner!r}, balance={self.balance!r})"
```

## 팁

-   문자열 포맷을 보기 좋게 하려면 f-string을 사용하는 것이 좋음.

-   금액 등 숫자는 `:,` 포맷으로 천 단위 구분을 적용할 수 있음.

    ``` python
    f"{self.balance:,}원"  # → 10,000원
    ```

-   개인정보(비밀번호, 계좌번호 등)는 `__str__`에서 절대 노출하지 않도록
    주의함.

## 정리

`__str__`은 객체가 "사람 눈에 보일 때" 어떤 모습으로 보일지를 정하는
메서드.
`print(obj)` → `str(obj)` → `obj.__str__()` 순서로 호출됩니다.\
반드시 문자열을 반환해야 하며, 내부에서 `print()`를 사용하면 안 됨.
