# 파이썬 패키지 만들기

네이버 테크 셰어에서 파이썬 개발 환경에 관해 공유한 내용 중 일부를 3부로 나누어 정리하였습니다.

이 글은 둘째 편으로 (주로) setuptools로 패키지를 만들고 관리하는 법을 다루었습니다. 

----

파이썬에서 의존성 관리하는 법에 대해 이야기하면 언제나 `requirements.txt`와 `setuptools`에 관한 이야기가 나온다.
조금 더 관심이 있다면 흔히 애플리케이션을 만들 때는 `requirements.txt`를 쓰고 라이브러리를 만들 때는 `setuptools`를 쓴다는 말도 들었을 것이다.
다양한 환경에서 동작해야 하는 라이브러리와 특정 환경에서 잘 돌기를 바라는 애플리케이션의 요구사항이 다르기 때문에 나오는 이야기이다.

여기서는 조금 다른 관점으로 나누어 볼 것인데, **`requirements.txt`는 코드를 복제할 때 쓰는 방식이고, `setuptools`는 파이썬 패키지를 배포할 때 쓰는 방식** 이다.

## requirements.txt만 만들지 말자

이왕이면 패키지로써 설치할 수 있게 만들자.

`requirements.txt`만 있는 프로젝트를 실행하려면, 반드시 코드를 복제해 와야 한다. 의존성을 설치하고 코드를 실행하는 것은 모두 사용자의 몫이다. 내 프로젝트를 사용하는 고객이 내 프로젝트의 코드를 정말로 소스코드로서 받아야 하는지를 생각해 보자. 대부분의 경우 아마 아닐 것이다.

예: 파이썬은 소스코드가 곧 바이너리이므로 다소 둔감해지는 경향이 있지만, 리눅스에서 소프트웨어를 사용하려고 찾아보았더니 `aptitude`로 설치하는 대신 소스코드를 받아서 빌드하라고 한다고 생각해보자. 사용하고 싶지 않을 것이다. 파이썬 코드 역시 패키지를 만들어 둔다면 공개된 pypi든 사내에서 굴리는 pypi든 패키지만 업로드해 사용할 수도 있다. 소스를 클론하고 `requirements.txt`를 설치하는 대신 말이다.

패키지 없이 `requirements.txt`와 파이썬 파일 몇 개를 관리할 경우 다음과 같은 몇 가지 단점이 있다는 점도 기억해 두자.
- 필요할 때 패키지로 전환하는게 어려워진다. 파이썬2와 3이 다르지만, 패키지이냐 아니냐는 import 경로에 영향을 끼친다. 패키지를 만들고 싶어지는 순간에도 지옥같은 import를 풀지 못하면 원치 않게 할 수 없게 될지도 모른다.
  - `requirements.txt`는 `setuptools`와 바로 호환되지는 않고, 이런 차이는 작업자들이 새로 패키지로 만드는 것을 꺼리게 만든다.
  - 반면 패키지는 언제나 `requirements.txt`로 설치할 수 있으므로 역은 아무 문제가 없다.
- 패키지로 만드는 편이 테스트 코드와 본 코드를 분리하기 더 쉬워지는 효과가 있어서 결국은 개발을 위해서도 패키징하는 편이 더 낫다.
  - 이를테면 처음에 작게 시작한 프로젝트에서 프로젝트 루트에 `test_blah.py` 같은 파일을 늘어놓았다고 생각해 보자. 테스트 코드가 늘어나서 `tests/test_blah.py`로 옮긴다면 `PYTHONPATH` 조작 없이는 import가 잘 되지 않는다. 패키지로 만들어두었다면 항상 패키지 이름으로 참조하므로 테스트 파일의 경로에 영향을 받지 않게 된다.
- 여러 섹션을 다룰 방법이 없기 때문에 빌드에, 테스트에, 문서화에 각기 다른 설치 요구가 있거나, 특정 기능을 활성화하기 위해 추가 기능을 설치해야 하면 `requirements-dev.txt`, `requirements-doc.txt` 같은 유사한 파일이 늘어나게 된다.

일단 한 번 만들어두기만 하면 `pip install -r requirements.txt`와 `pip install -e .` 사이에는 사용성에는 큰 차이도 없다.
의존성의 나열이 중요하면 `pip freeze` 같은 도구의 도움을 받을 수도 있다.
그럼에도 불구하고 지금까지 `requirements.txt`가 선호되어 온 데는 여러 이유가 있겠으나, 단순한 리스트의 나열이라 쉽다는 점이 한 몫 할 것이고,
`setup.py`가 파이썬 소스코드여서 단순한 리스트와는 달리 디버그하는데 노력이 든다는 점도 한 몫 하였을 것이다.

Note: 한편 여전히 로컬에서 잠시 운용하거나 한두번 실행하면 그만인 스크립트 파일 몇 개를 공유하는 용도라면, 괜히 패키지에 시간을 낭비하는 대신 손쉽게 requirements.txt를 사용할 수도 있을 것이다. 만일 패키지가 아니라 `requirements.txt`조차 없는 환경이라면, 일단 의존성 정리를 위해 `requirements.txt`부터 만들고 시작하는게 심리적으로 부담이 덜 될수도 있겠다.

다행히 최근에는 setuptools 업데이트로 `setup.cfg`에 더 풍부한 표현을 쓸 수 있게 되면서 setuptools를 사용하는 난이도가 약간 떨어졌다.

## setuptools 를 써 보자

일단 들어가기 전에, 파이썬은 의존성 관리 측면에서 멀쩡한 도구를 가져본 적이 한 번도 없었다. 복잡한 케이스에서 의존서 충돌을 제대로 감지하지도 못하고, 다른 현대 개발 환경과 달리 표준화된 신뢰할만한 의존성 재현 방법도 존재하지 않는다.

하지만 지금처럼 새 도구가 심심찮게 나오는 시대도 드물었던 것 같다. 아마 조만간 이 문제는 풀리게 될 지도 모르겠다.
(조금 다르게 생각해 보면, 지금 새로운 도구를 신나게 익히고 나면 시간이 지나면 어떻게 판도가 뒤집혀 있을지 모른다는 이야기도 된다.)

setuptools는 오랫동안 표준화된 패키지 방법이므로, 아마 앞으로도 한동안은 비슷한 지위를 누릴 것으로 생각되므로 먼저 setuptools를 익혀 보자. 다음과 같은 특징이 있다. 라이브러리와 애플리케이션 모두에 적용할 수 있는 설계이므로 도구를 하나만 익혀도 된다는 장점도 있다.

### setup.py
전통적으로 setuptools의 패키지 정의 파일은 `setup.py`라는 파이썬 소스코드 파일에 작성된다. 장단점이 있는데, 자유도가 높으므로 아무리 복잡한 요구에도 어떻게든 대응할 수 있는 반면, 대체로 자유도가 상승할수록 복잡도도 상승한다. 대체로 요즈음에는 실행 없이 분석할 수 없다는 단점이 더 부각되어 이런 용도로는 선언적 언어를 더 선호한다.

다행히도 setuptools가 `setup.cfg` 지원을 강화하면서 이 문제는 어느정도 해소된 것 같다. (오래된 버전의 파이썬을 사용중이라면 setuptools를 업그레이드해야 할 수도 있다)

새 패키지를 만드는 법을 소개하는 것이 목적이므로, `setup.cfg`를 사용하기 위해 `setup.py`는 껍데기만 만들자. **모든 설정은 `setup.cfg`에 들어가면 되므로 다음 정도면 충분하다**.

```python
import setuptools

assert tuple(map(int, setuptools.__version__.split('.'))) >= (39, 2, 0), \
    'Plesase upgrade setuptools by `pip install -U setuptools`'

setuptools.setup()
```

`assert` 부분은 미래에는 필요없어지겠지만, `setup.cfg` 지원이 나름 최신 기능인 관계로 낮은 버전의 setuptools가 설치된 환경을 위해 지금은 포함시켜 두는 편이 낫다.

### setup.cfg

기본적인 파이썬 패키지는 다음 정도의 내용이면 충분하다.

```
[metadata]
name = <package_name>
version = 0.0.1
[options]
packages = <my_package>
install_requires=
    six>=1.10.0
```

`<package_name>`에 패키지 이름을, `<my_package>`에 이 패키지를 설치할 때 설치할 소스코드가 담긴 디렉터리를 추가하자.
의존성은 참고차 six 하나 넣어두었고, 필요하면 여러 줄로 쓰면 된다.

파이썬 버전 별로 필요한 의존성이 다르거나, 더 많은 정보를 넣고 싶다면 [setup.py를 setup.cfg로 옮긴 예](https://github.com/youknowone/itunes-iap/pull/60/files#diff-380c6a8ebbbce17d55d50ef17d3cf906)를 참고하거나, [setuptools 공식 문서](https://setuptools.readthedocs.io/en/latest/setuptools.html)의 표를 참고하자.

잡담: `setup.cfg`는 원래 `setup`의 설정을 덮어쓰는 용도로 만들어졌고, 다른 파이썬 도구(pytest라던가)의 설정을 저장해 두는 용도로도 널리 쓰였다. 현재는 `setup.py`의 기능을 많이 흡수하여 거의 대신할 수 있을 정도가 되었고, 설정 저장의 기능은 setuptools에 속하는 `setup.cfg` 대신 표준 프로젝트 설정은 `pyproject.toml`을 쓰는 방향으로 바뀌어 가고 있다.

## 개발 모드로 설치하기

사용자가 패키지를 설치해서 쓸 요량이라면 처음부터 설치한 패키지를 테스트하는 것이 올바른 관리 전략일 것이다.
먼저, 사용자가 설치할때와 똑같은 방법으로 설치하려면 그냥 현재 디렉터리를 설치하면 된다.

```shell
$ pip install .  # 현재 pwd가 setup.py가 있는 디렉터리일 경우 (.는 현재 디렉터리이므로)
```

하지만 이 방식으로는 이 패키지가 정말로 site-packages에 설치되어 버리고, 그럼 코드를 고칠 때마다 고친 코드 적용을 위해 패키지를 새로 설치해야 한다.

편의를 위해 개발용으로 쓰기 위해서는 editable 모드로 설치하면 현재 디렉터리가 그대로 설치되므로, 코드를 변경하면 즉각 반영되게 될 것이다.
```shell
$ pip install --editable .  # 또는 pip install -e .
```

Note: editable로 설치하게 되면 몇몇 설치 과정을 건너뛰게 된다. 따라서 패키지 배포 전에는 반드시 일반 설치를 테스트해 보는 것이 좋다. CI에서 일반 설치를 하게 되면 사용자와 동일한 환경을 테스트하게 될 것이다.

## 용도에 따라 다른 의존성 부여하기

복잡한 소프트웨어라면 활성화할 기능 별로 의존성이 다른 경우들도 있고, 그렇지 않더라도 대부분 배포된 패키지에 필요한 의존성과 그 제품을 개발하기 위한 환경에서는 서로 다른 의존성을 요구하는 경우가 많다. 하다 못해 개발 중에는 `pytest` 같은 테스트 도구라도 하나 더 사용하게 된다.

이런 경우 `options.extras_require`에 이름을 달아 필요한 의존성을 추가하고, 설치할 때는 설치 옵션을 줄 수 있다.

예를 들어 위 `setup.cfg`를 좀 더 고쳐 다음과 같이 test라는 이름으로 테스트용 설치에 필요한 추가 의존성을 만들어 보자.
```
[metadata]
name = <package_name>
version = 0.0.1
[options]
packages = <my_package>
install_requires=
    six>=1.10.0
[options.extras_require]
test =
    pytest==5.2.2
```

이제 로컬에서 테스트 의존성을 설치하기 위해서는 다음과 같이 할 수 있다.

```shell
$ pip install -e '.[test]'  # 따옴표에 유의하자. []는 셸 문법이어서 따옴표 없이는 셸 문법으로 간주된다.
```

잡담: setuptools 사용자들은 `python setup.py test` 같은 명령이 기억날 것이다. 현재 setuptools의 test 명령은 deprecate 되었고 따라서 `tests_require`는 더 이상 권장되지 않는다. 모두 `extras_require`를 쓰면 된다. 기억할 게 하나 더 줄었다!

## requirements.txt로 의존성 버전 고정하기

이 파트는 파이썬 표준 패키지에서 가장 취약한 부분 중 하나이다. 정말로 새로 나온 다른 서드파티 도구들이 더 잘하고 있는 부분이다. 조만간 불필요한 글이 되기를 바란다 :)

라이브러리를 배포할 때는 잘 동작하는 버전을 `setup.cfg`에 느슨한 조건으로 써 두게 된다. 정확한 버전을 요구할 경우 같은 라이브러리의 다른 버전을 요구하는 다른 라이브러리와 충돌할 가능성이 높아지기 때문이다.

반면 애플리케이션을 배포할 때는 개발할 때 사용한 것과 정확히 같은 버전을 배포환경에서도 똑같이 사용하는 것이 안전하다. 안타깝게도 파이썬 생태계에는 아직 이 역할을 돕는 표준 방법이 존재하지 않는 것처럼 보인다. (여러 도구가 이런 역할을 지원하고는 있다)

고전적인 방법은 [`pip freeze`를 그대로 저장해두었다가 설치해 쓰는 것](https://pip.pypa.io/en/stable/reference/pip_freeze/
)이다. `venv`나 `pyenv-virtualenv`를 이용해 잘 격리해 두었다면 큰 군더더기 없는 목록이 나오므로 충분히 실용적이다.
```shell
$ pip freeze > requirements.lock  # 개발 장비에서
$ git commit requirements.lock
...
$ pip install -r requirements.lock  # 배포 또는 다른 개발 장비에서
```

앞에서 `requirements.txt`로만 의존성 관리를 하지 말자고 하였다. 위와 같이 사용할 경우 애플리케이션 프로젝트에는 `setup.py`와 `requirements.txt`가 공존할 수 있다. 이 경우 `requirements.txt`가 의존성 잠금 기능을 대신한다. 이 때 `requirements.txt`는 손으로 만든 파일이 아니라는 점을 유의하자.

만일 배포할 장비에서 내 코드 저장소에 접근할 수 없으면 자동으로는 되지 않을 수도 있다. `pip freeze` 가 내 프로젝트에 대해 올바른 의존성을 생성해내지 못하거나, 접근할 수 없는 저장소에 대한 의존성을 생성한다면 `-e .`을 추가하는 등의 방법으로 약간 손을 보자.

## pyproject.toml

비교적 최근(~2016)에 추가된 새 파이썬 프로젝트 정의 방식. 표준으로는 build system을 정의하는 방법을 정의하고 있고, 그 외에 다른 도구가 읽을 수 있는 설정을 저장하는 역할도 겸하고 있다.

PEP-517에는 다음과 같이 setuptools 대신 flit을 사용하도록 하는 예시가 포함되어 있다.

```toml
[build-system]
# Defined by PEP 518:
requires = ["flit"]
# Defined by this PEP:
build-backend = "local_backend"
backend-path = ["backend"]
```

앞에서는 `setup.py`에 assert 구문을 추가해 setuptools의 버전을 검사하였는데, `pyproject.toml`로도 특정 버전 이상의 `setuptools`를 요구할 수 있다.

```toml
[build-system]
requires = ["setuptools>=42.0"]
```

## poetry
[poetry](https://python-poetry.org/)는 `pyproject.toml`로 빌드 시스템을 지정할 수 있게 된 덕분에, 새롭게 쓸 수 있게 된 빌드 도구 가운데 하나이다.

신뢰 할 수 있는 dependency resolution과 lock을 제공하여 현재 시점에서는 애플리케이션 개발에 상당한 인기를 누리고 있다.

주의: 아직 매우 흔한 사용례 밖에서는 불안정하다는 보고가 있고, 새로운 도구들이 여럿 등장하는 시대이므로 특별히 다른 도구보다 더 권장하는 것은 아니다. 위의 PEP 인용에서 flit이 언급되었지만 고작 몇 년 지난 지금은 아무도 flit을 쓰지 않는다는 점을 기억하자.

주의: 그렇다고 아무 도구나 다 비슷하다는 얘기는 아니다. pipenv는 쓰지 말자. 쓰지 말아야 할 이유는 [여기](https://velog.io/@doondoony/pipenv-101)에 잘 설명되어 있으니 링크로 대신한다.
