# 널리 쓰이는 파이썬 보조 도구

네이버 테크 셰어에서 파이썬 개발 환경에 관해 공유한 내용 중 일부를 3부로 나누어 정리하였습니다.

이 글은 셋째 편으로 널리 쓰이는 파이썬 보조 도구를 다루어야 하지만 쓰다 말았습니다.
도구 소개 시간이니 제목이 중요한게 아닐까요? 라고 변명하는 마음입니다.

## flake8 - 코드 스타일 통일

널리 알려진 사실: 모두들 선호하는 코드 스타일이 다르고 나와 다른 코드 스타일은 나쁜 취향으로 보인다

함께 코딩하기 위해서 어떻게 코드 스타일을 조율할까?
이 문제에서 파이썬은 평균보다는 나은 환경을 갖고 있는데, 표준(주로 [PEP-8](https://www.python.org/dev/peps/pep-0008/))이 정의한 스타일이 있기 때문이다.

주의: pep8은 오래 전에 작성되어 버전 관리에 덜 친화적인 부분이 있고, 최신 기능에 대한 제안은 포함되어 있지 않다.

[flake8](https://pypi.org/project/flake8/)을 쓰면 pep8 위반에 대해 경고해주고, 팀에서 별도로 필요한 규칙을 플러그인으로 추가하기도 쉽다.

예시 코드:
```python

class   uglyClass:
   # 3-space intent! yay!
   def\
        UNACCEPTABLE_NAME(this):
    print(     "aligned"      );
```

flake8 실행 결과
```shell
$ flake8 ugly.py
ugly.py:2:6: E271 multiple spaces after keyword
ugly.py:3:4: E114 indentation is not a multiple of four (comment)
ugly.py:4:4: E111 indentation is not a multiple of four
ugly.py:6:11: E201 whitespace after '('
ugly.py:6:30: E202 whitespace before ')'
ugly.py:6:32: E703 statement ends with a semicolon
ugly.py:6:33: W292 no newline at end of file
```

Note: pep8은 절대적 규칙이 아니라 협업을 돕기 위한 도구입니다. [`--ignore` 설정과 `noqa` 주석](http://flake8.pycqa.org/en/3.1.1/user/ignoring-errors.html)을 적절히 사용해 쾌적환 환경 조성에 도움이 되게 사용합시다

## pytest - 테스트 도구

파이썬 표준에는 unittest 모듈이 포함되어 있지만, 주로 다음 이유로 개인적으로 pytest 사용을 권하고 있다.

- 전용 assert 함수 대신 내장 assert 구문 쓸 수 있다.
- parametrize: 여러 input의 조합을 테스트하기 위해 루프 만들지 않아도 된다
- fixture: 일관성 있는 테스트 객체 제공을 setup/teardown을 사용하지 않고도 쉽게 할 수 있다.
- 플러그인 지원이 다양하여 찾아보면 필요한건 대강 다 있다.
- 테스트 리포트 지원이 잘 되어서 coverage 등의 도구와 함께 사용하기 쉽다.

## futurize - 파이썬2/3 호환 도구

파이썬3를 모두가 쓰면 좋겠지만, 이미 만들어진 파이썬2 코드는 어떻게 하겠는가!

현재 [futurize](https://python-future.org/automatic_conversion.html)를 쓰는 것이 가장 쉬운 방법이다.

현실적으로 테스트 일정이나 동료와의 협업을 고려하면 파이썬2 코드를 하루아침에 파이썬3로 모두 바꾸기는 어려운데, 
2to3 같은 도구는 파이썬2와 3을 동시에 쓰는 환경에 대한 배려가 부족하다.

futurize를 실행하면 파이썬2/파이썬3 반 호환 코드를 생성해준다. 다음과 같은 문제가 있으니 참고하자.
- 대체로 파이썬3에서는 잘 돌고 파이썬2에서는 조금 느린 코드를 만들게 된다. 넘어가는 중에 감내할 수 있다면 괜찮음.
- 한국어 서비스의 난제인 str/unicode와 bytes/str 변환은 거의 지원되지 않는다고 보아야 한다. 수동으로 옮길 각오를 하자.

대체로 문자열 문제만 해결하면 적용에 큰 문제는 없는 편이다.
문자열 문제는 충분한 테스트를 작성하고 부분 부분 정복하는 전략을 잘 세워서 시도해 보자.

## mypy - typing

없는거 보단 나으니 내 코드에라도 붙여 쓰자. 파이썬2가 퇴역하였으므로 파이썬3만 지원하는 코드가 늘면 상황이 다소 좋아질 것으로 생각된다.

IDE를 쓰면 자동완성도 잘 되는 효과가 있으니 내 개발환경을 위해서 쓴다고 생각하고 조금씩 붙여 보자.