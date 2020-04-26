# 자료구조는 적게, 일은 더 많이

이 장을 읽고 나서 다음 질문들에 답해보자
* 프로그램 제어의 흐름
* 코드와 데이터를 효과적으로 헤아림
* map, reduce, filter의 진면목
* 로대시JS 라이브러리와 함수 체인
* 재귀적 사고방식

----

### 3.1 애플리케이션의 흐름

제어 흐름control flow
* 프로그램이 정답에 이르기까지 거치는 경로를 제어 흐름이라고 한다
* 명령형 프로그램에서는 모든 단계를 노출해 흐름이나 경로를 자세히 서술한다
* 반면 선언적 프로그램은 독립적인 블랙박스 연산들이 최소한의 제어 구조로 연결되어 추상화 수준이 높다
* 이렇게 연결한 연산들은 각자 다음 연산으로 상태를 이동시키는 고계함수에 불과하다
* 실제로 함수형 프로그램은 데이터와 제어 흐름 자체를 고수준 컴포넌트 사이의 단순한 연결로 취급한다

### 3.2 메서드 체이닝

메서드 체이닝method chaining
* 여러 메서드를 단일 구문으로 호출하는 OOP 패턴
* 대부분 OOP에서 불변 객체에 많이 적용하나 FP에도 잘 맞는다

문자열을 다루는 예제
```js
'Functional Programming'.substring(0, 10).toLowerCase() + ' is fun';
```
  * substring과 toLowerCase는 자신을 소유한 문자열 객체에 this로 접근해 특정 작업을 한 다음 새 문자열을 반환한다
  * 문자열은 처음부터 불변값으로 설계됐기 때문에 원본과는 무관한 문자열이 생성된다

함수형으로 위의 예제 리팩토링
```js
concat(toLowerCase(substring('Functional Programming', 1, 10))),' is fun');
```
  * 매개변수는 모두 함수 선언부에 명시해서 부수효과를 없애고 원본객체를 바꾸지 않는다는 원칙을 충실히 반영하고 있다
  * 그러나 이렇게 안쪽에서 바깥쪽으로 이어지는 코드는 메서드 체이닝 방식만큼 가독성이 좋지 않다
  * 변이를 일으키지 않는 한 FP에서도 단일 객체 인스턴스에 속한 메서드를 체이닝하는 것은 쓸모가 있다

### 3.3 함수 체이닝

OOP의 코드 재사용 방식
* 주로 상속을 통해 부모형의 상태와 메서드를 물려받는다
* ex. 자바에서 List 인터페이스를 용도에 맞게 ArrayList, LinkedList, DoublyLinkedList, CopyOnWriteArrayList로 구현
* 이들은 모두 한 부모에서 출발해 특수한 기능을 덧붙인 클래스

함수형의 코드 재사용 방식
* 자료구조를 새로 생성하지 않고 배열 등 흔한 자료구조를 이용해 여러 개로 나뉜 고계 연산을 적용시킨다

고계 연산이 하는 일
* 작업을 수행하기 위해 무슨 일을 해야 하는지 기술된 함수를 인수로 받는다
* 임시 변수의 값을 계속 바꾸면서 부수효과를 일으키는 기존 수동 루프를 대체한다
* 그 결과 관리할 코드가 줄고 에러가 날 만한 코드 역시 줄어든다

### 3.3.1 람다 표현식lambda expression

람다 표현식
* 한 줄짜리 익명 함수를 보다 단축된 구문으로 만들어준다
* 여러 줄로 표기할 수도 있지만 대부분 한 줄로 쓴다

자세한 구문 구조
```js
var lambda = (param1, param2, ...) => expression;

var lambda = (param1, param2, ...) => {
                                        statement1,
                                        statement2,
                                        return finalStatement;
                                      };
```
  * 괄호 안에 0개 이상의 매개변수가 있어야 한다
  * 표현식은 한 줄로 쓸 수도 있고 여러 줄로 쓸 수도 있다
  * 여기서 lambda는 실재하는 값이 아니라 특정 작업을 하는 로직이 담긴 화살표 함수를 가리키는 느긋한 방법을 사용한다

리스트 처리
* FP에서는 람다 표현식과 잘 어울리는 세 주요 고계함수인 map, reduce, filter를 적극 권장한다
* 사실 대부분의 FP는 자료 리스트를 처리하는 코드다
* JS의 전신이자 함수형 언어인 LISP의 이름도 List Processor에서 비롯된 것이다
* 함수형 배열 연산을 지원하는 array의 함수들은 ES5에도 있다
* 로대시 JS를 사용하면 이와 유사한 다른 유형의 연산까지 처리할 수 있다

### 3.3.2 _.map: 데이터를 변환

학생 리스트에서 성명을 추출하는 코드
```js
const p1 = new Person('Haskell', 'Curry', '111-11-1111');
p1.address = new Address('US');
p1.birthYear = 1900;

const p2 = new Person('Barkley', 'Rosser', '222-22-2222');
p2.address = new Address('Greece');
p2.birthYear = 1907;

const p3 = new Person('John', 'von Neumann', '333-33-3333');
p3.address = new Address('Hungary');
p3.birthYear = 1903;

const p4 = new Person('Alonzo', 'Church', '444-44-4444');
p4.address = new Address('US');
p4.birthYear = 1903;

var result = [];
var persons = [p1, p2, p3, p4];

for (let i = 0; i < persons.length; i++) {
  var p = persons[i];
  if (p !== null && p !== undefined) {
    result.push(p.fullname);
  }
}
```
  * 명령형 코드에서 구체적 내용은 다르더라도 매우 자주 볼 수 있는 방식의 코드다

_.map을 사용해 함수형 스타일로 리팩토링
```js
_.map(persons,
  s => (s !== null && s !== undefined) ? s.fullname : ''
)
```
  * map은 루프를 쓰거나 스코프 문제를 신경쓸 필요 없이 컬렉션의 원소를 전부 파싱할 경우 유용하다
  * 연산이 끝나면 항상 새로운 배열을 반환하므로 불변성도 간직된다

map 연산을 수학적으로 바라보기
```js
map(f, [e0, e1, e2, ...]) -> [r0, r1, r2, ...];
// 여기서 f(en) = rn
```
  * f와 n개의 원소가 담긴 컬렉션을 받아 왼쪽에서 오른쪽으로 각 원소에 f를 적용해 크기가 n인 계산 결과를 반환한다

추상화된 내부 구현 코드를 보기
```js
function map(arr, fn) {
  const len = arr.length,
        result = new Array(len);
  for (let idx = 0; idx < len; ++idx) {
    result[idx] = fn(arr[idx], idx, arr);
  }
}
```
  * _.map 안에서도 일반 루프를 사용한다
  * _.map이 반복을 대행해주기 때문에 개발자는 루프 변수의 경계조건을 체크하는 수고를 덜 수 있다
  * 이제는 이터레이터 함수에 구현한 비즈니스 로직만 신경 쓰면 된다
  
방향 전환
* map 연산은 무조건 왼쪽 -> 오른쪽으로 진행된다
* 만약 오른쪽 -> 왼쪽으로 연산을 진행하려면 배열 원소를 거꾸로 뒤집어야 한다
* 로대시 JS에서는 Array.reverse()에 해당하는 _.reverse()를 메서드를 제공한다
* 이 함수는 원본 배열에 변이를 일으키므로 부수효과가 언제 일어날지 알고 있어야 한다
```js
_(persons).reverse().map(
  p => (p !== null && p !== undefined) ? p.fullname : ''
);
```
  * 첫줄에 _다음에 괄호로 객체나 배열을 감싸면 Lodash는 내부적으로 LodashWrapper라는 래퍼 객체로 감싼다
  * 그러면 로대시의 모든 API 함수를 점(.)으로 계속 호출할 수 있다
  * 제이쿼리에서 $(객체)로 감싸서 이용하는 것과 유사하다

### 3.3.3 _.reduce: 결과를 수집

reduce
* 데이터를 변환한 후 의미있는 결과를 도출하고 싶을 때 사용한다
* reduce는 원소 배열을 하나의 값으로 짜내는 고계함수로, 원소마다 함수를 실행한 결괏값의 누적치를 계산한다

reduce 연산을 수학적으로 바라보기
```js
reduce(f, [e0, e1, e2, e3], accum) -> f(f(f(f(accum, e0), e1, e2, e3))) -> R
```
  * 앞 단계의 결괏값에 현 단계의 결괏값을 누적하고 배열 끝에 이를 때까지 반복한다
  * 반드시 하나의 값(R)으로 귀결된다

reduce 구현부
```js
function reduce(arr, fn, accumulator) {
    let idx = -1,
        len = arr.length;

    if (!accumulator && len > 0) {
      accumulator = arr[++idx];
    }

    while (++idx < len) {
      accumulator = fn(accumulator, arr[idx], idx, arr);
    }

    return accumulator;
  }
```
  * fn은 배열 원소마다 실행할 이터레이터 함수로, 매개변수는 누산치, 현재 값, 인덱스, 배열이다
  * accumulator는 계산할 초깃값을 넘겨받는 인수이고, 함수 호출을 거치며 매 호출시 계산된 결괏값을 저장하는 데 쓰인다

Person 객체 컬렉션에서 국가별 인구 등 유용한 통계치를 산출하는 프로그램 작성하기
```js
_(persons).reduce((stat, person) => {
  const country = person.address.country;
  stat[country] = _.isUndefined(stat[country]) ? 1 : stat[country] + 1;
  return stat;
}, {});

// {US: 2, Greece: 1, Hungary: 1}
```

reduce는 map과 자주 조합해 사용한다
```js
// _(persons).map(func1).reduce(func2); 대략 이런 흐름으로 진행된다
// 위에 작성한 코드를 아래처럼 리팩토링할 수 있다

const getCountry = person => person.address.country;
const gatherStats = function (stat, criteria) {
  stat[criteria] = _.isUndefined(stat[criteria]) ? 1 : stat[criteria] + 1;
  return stat;
};

_(persons).map(getCountry).reduce(gatherStats, {});
// {US: 2, Greece: 1, Hungary: 1}
```

reduce가 비효율적인 상황
* reduce는 일괄적용apply-to-all 연산이므로 배열을 순회하는 도중 그만두고 나머지 원소를 생략할 수 없다
* 잘못된 입력값이 하나라도 발견되면 나머지 값들을 체크할 필요가 없을 때, _.some, _.isUndefined, _.isNull 같은 함수를 사용하면 좀 더 효율적인 검증기를 만들 수 있다

_.some, _.every
* some은 주어진 조건을 만족하는 값이 발견되는 즉시 true를 반환
* every는 주어진 조건을 만족하지 않는 값이 발견되는 즉시 false를 반환
```js
const isValid = val => !_.isUndefined(val) && !_.isNull(val);
const isNotValid = val => _.isUndefined(val) || _.isNull(val);
const allValid = args => _(args).every(isValid);
const notAllValid = args => _(args).some(isNotValid);

allValid(['string', 0, null]); // false
allValid(['string', 0, {}]); // true
notAllValid(['string', 0, null, undefined]); // true
notAllValid(['string', 0, {}]); // false
```
  * allValid는 하나라도 false면 함수를 즉시 반환한다
  * 모든 값이 올바른지 확인할 때 유용하다
  * notAllValid는 하나라도 true면 함수를 즉시 반환한다
  * 최소한 하나의 값이라도 올바른지 확인할 때 유용하다

모든 원소를 탐색하는 낭비를 피하기
* map과 reduce는 배열 원소를 모두 탐색한다
* 그런데 원소 중에 null이나 undefined가 있을 때 연산을 건너 뛰어야 할 경우도 있다
* 이렇게 연산 이전에 특정 원소를 미리 솎아낼 수단으로 filter를 사용할 수 있다

### 3.3.4 _.filter: 원하지 않는 원소를 제거

계산하지 않을 원소를 사전에 제거하기
```js
filter(p, [d0, d1, d2, d3, ..., dn]) -> [d0, d1, ..., dn](원래 집합의 부분집합)
```
  * filter는 배열 원소를 반복하면서 술어 함수 p가 true를 반환하는 원소만 추려내어 새 배열에 담아 반환하는 고계함수다
  
filter 구현부
```js
function filter(arr, predicate) {
  let idx = -1,
      len = arr.length,
      result = [];

  while (++idx < len) {
    let value = arr[idx];
    if (predicate(value, idx, this)) {
      result.push(value);
    }
  }

  return result;
}
```
  * 대상 배열과 원소를 결과에 포함할지 결정하는 술어 함수 두 가지를 인자로 받는다
  * 술어 함수 결과가 true인 원소는 남기고 그렇지 않은 원소는 내보낸다

filter의 사용처
```js
_(persons).filter(isValid).map(fullname);
```
  * 배열에서 오류 데이터를 제거하는 용도
```js
const bornIn1903 = person => person.birthYear === 1903;

// 책에는 fullname 함수는 빠져 있어서 직접 구현했다
const fullname = person => person.fullname;

_(persons).filter(bornIn1903).map(fullname).join(' and '); 
// "John von Neumann and Alonzo Church"
```

축약comprehension
```js
[for (x of 이터러블) if (조건) x]
```
* 위에서 본 것처럼 map, filter 등을 조합하는 대신 축약이란 개념을 적용하는 방법도 있다
* 배열 혹은 리스트 축약comprehension은 map, filter를 각각 for..of와 if 키워드를 이용해 단축된 구문으로 캡슐화한다
* 검색해서 안 것인데 아직 표준이 아니므로 사용하지 않는 것이 좋다

### 3.4 코드 헤아리기

코드 헤아리기
* 프로그램의 일부만 들여다봐도 무슨 일을 하는 코드인지 멘털 모델을 쉽게 구축할 수 있다는 의미
* 멘털 모델이란 전체 변수의 상태와 함수 출력 같은 동적인 부분뿐만 아니라
* 설계 가독성 및 표현성 같은 정적인 측면까지 포괄하는 개념이다
* 불변성과 순수함수는 이러한 멘털 모델 구축을 용이하게 해준다
* 함수형에서는 고수준 연산을 서로 연결해 프로그램 로직을 직접 파헤치지 않아도 윤곽을 잡기 쉽다
* 따라서 프로그래머는 코드 뿐만 아니라, 결과를 내기 위해 서로 다른 단계를 드나드는 데이터의 흐름까지 더 깊게 헤아릴 수 있다

### 3.4.1 선언적 코드와 느긋한 함수 체인

전체 프로그램을 구성하는 방법
* FP는 단순 함수들로 프로그램을 구성한다
* 개별 함수는 많은 일을 하지 않지만, 함께 뭉치면 복잡한 작업도 해낼 수 있다
* FP의 선언적 모델에 따르면, 프로그램은 개별적인 순수함수들을 평가하는 과정이다
* FP는 필요시 코드의 표현성 향상을 위한 추상화 수단을 지원한다
* 따라서 개발하고 있는 애플리케이션의 실체를 명확히 표현하는 온톨로지ontology 또는 어휘집vocabulary을 만들 수 있다
* 앞서 본 map, filter, reduce 등 순수함수를 쌓아가면 자연스레 한눈에 봐도 흐름이 읽히는 코드가 완성된다

자료구조보다 연산에 중점을
* 추상화를 거치면 기반 자료구조에 영향을 끼치지 않는 방향으로 연산을 바라볼 수 있다
* 배열, 연결 리스트, 이진 트리 등 어떤 자료 구조를 쓰더라도 프로그램 자체의 의미가 달라져서는 안된다
* FP는 자료구조보다 연산에 더 중점을 둔다

이름 리스트를 읽고 데이터를 정제한 후 중복을 제거하고 정렬하기
```js
var names = ['alonzo Church', 'Haskell curry', 'stephen_kleene', 'John von Neumann', 'stephen_kleene'];

// 명령형 코드
var result = [];

for (let i = 0; i < names.length; i++) {
  var n = names[i];
  if (n !== undefined && n !== null) {
    var ns = n.replace(/_/, ' ').split(' ');
    for (let j = 0; j < ns.length; j++) {
      var p = ns[j];
      p = p.charAt(0).toUpperCase() + p.slice(1);
      ns[j] = p;
    }
    if (result.indexOf(ns.join(' ')) < 0) {
      result.push(ns.join(' '));
    }
  }
}
result.sort(); // ["Alonzo Church", "Haskell Curry", "John Von Neumann", "Stephen Kleene"]

// 함수형 코드
_.chain(names)
 .filter(isValid)
 .map(s => s.replace(/_/, ' '))
 .uniq() 
 .map(_.startCase)
 .sort()
 .value();
// ["Alonzo Church", "Haskell Curry", "John Von Neumann", "Stephen Kleene"]
```
  * 명령형의 단점은 특정 문제의 해결만을 목표로 한다는 점이다
  * 이렇게 추상화 수준이 낮으면 코드를 재사용하기 어렵고 에러 가능성과 복잡도가 증가한다
  * FP는 블랙박스 컴포넌트를 서로 연결만 해주고, 뒷일은 테스트를 마친 검증된 API에게 모두 맡긴다
  * names 배열을 정확한 인덱스로 순회하는 등 버거운 일은 모두 filter, map 등이 대행한다
  * 프로그래머는 그저 나머지 단계에 대한 프로그램 로직을 구현하면 된다

앞서 국가별 인구를 계산한 코드를 리팩토링 하기
```js
// gatherStats 함수 리팩토링
const gatherStats = function (stat, country) {
  if (!isValid(stat[country])) {
    stat[country] = {'name': country, 'count': 0};
  }
  stat[country].count++;
  return stat;
};

// Person 데이터 추가
const p5 = new Person('David', 'Hilbert', '555-55-5555');
p5.address = new Address('Germany');
p5.birthYear = 1903;

const p6 = new Person('Alan', 'Turing', '666-66-6666');
p6.address = new Address('England');
p6.birthYear = 1912;

const p7 = new Person('Stephen', 'Kleene', '777-77-7777');
p7.address = new Address('US');
p7.birthYear = 1909;
  
_.chain(persons)
 .filter(isValid)
 .map(_.property('address.country'))
 .reduce(gatherStats, {})
 .values()
 .sortBy('count')
 .reverse()
 .first()
 .value()
 .name;
```
  * 인구가 가장 많은 국가를 반환하기 위해 여러 함수형 장치들을 _.chain으로 연결했다
  * _.chain은 복잡한 프로그램을 느긋하게 작동시키는 장점이 있다
  * 제일 끝에서 value() 함수를 호출하기 전에는 아무것도 실행되지 않기 때문이다
  * _.chain은 반드시 끝에 value() 함수를 호출해야 값이 풀려(unwrapped) 반환된다
  * _.()은 암시적 연쇄로서 끝에 value()가 없이도 단일 값으로 리듀스되는 함수가 연쇄될 경우 체인을 종결시킨다

### 3.4.2 유사 SQL 데이터: 데이터로서의 함수

FP에서 자주 사용하는 함수들과 닮은 SQL 구문
* 앞서 본 map, reduce, filter 등의 이름을 잘 보면 함수가 데이터에 하는 일을 쉽게 추측할 수 있다
* 그런데 조금만 관점을 틀어보면 이 함수들은 SQL 구문과 많이 닮아있다
  ```SQL
  SELECT p.firstname FROM Person p
  WHERE p.birthYear > 1903 and p.country IS NOT 'US'
  GROUP BY p.firstname
  ```
* 쿼리 언어를 구사해 개발하는 것과 FP에서 배열에 연산을 적용하는 것은 일맥상통한다
  ```js
  // 믹스인으로 라이브러리에 함수를 추가해 확장한다
  _.mixin({'select': _.map,
           'from': _.chain,
           'where': _.filter,
           'sortBy': _.sortByOrder});

  _.from(persons)
   .where(p => p.birthYear > 1900 && p.address.country !== 'US')
   .sortBy(['firstname'])
   .select(p => p.firstname)
   .value();
  ```
* FP는 흔히 사용되는 어휘집이나 대수학 개념을 활용해서 데이터 자체의 성격과 구조 체계를 더 깊이 추론할 수 있게 한다

데이터로서의 함수
* 위에서 본 것처럼 JS 코드도 SQL처럼 데이터를 함수 형태로 모형화 할 수 있다
* 이를 functions as data라는 개념으로 부르기도 한다
* 여기서는 선언적으로 어떤 데이터가 출력되어야 할지 서술할 뿐 출력을 어떻게 얻는지는 논하지 않는다

### 3.5 재귀적 사고방식

복잡한 문제들
* 해결책이 떠오르지 않는 복잡한 문제를 해결하려면 문제를 분해할 방법을 찾아야 한다
* 재귀가 여기에 하나의 해답이 될 수 있다
* 하스켈, 스킴, 얼랭 등 순수 함수형 언어는 처음부터 루프 구조가 없기 때문에 배열 등을 탐색할 떄 재귀가 필수다
* JS에서도 XML, HTML 문서, 그래프 등을 파싱할 때 재귀를 다양하게 활용한다

### 3.5.1 재귀란?

재귀
* 주어진 문제를 자기 반복적인 문제들로 잘게 분해한 다음, 다시 조합해 원래의 정답을 찾는 기법
* 재귀 함수의 주된 구성 요소는 두 가지로 나뉜다
  * 기저 케이스base case(종료 조건terminating condition이라고도 한다)
  * 재귀 케이스recursive case

기저 케이스base case
* 재귀 함수가 구체적인 결괏값을 바로 계산할 수 있는 입력 집합

재귀 케이스recursive case
* 함수가 자신을 호출할 때 전달한 입력 집합(최초 입력 집합보다 점점 작아져야 한다)
* 입력 집합이 작아지지 않으면 재귀가 무한 반복되며 프로그램이 종료될 것이다

### 3.5.2 재귀적으로 생각하기

재귀적 사고
* 자기 자신 또는 그 자신을 변형한 버전을 생각하는 사고방식
* 재귀적 객체는 스스로를 정의한다

배열의 원소를 모두 더하기
```js
// 명령형 코드
var acc = 0;
for (let i = 0; i < nums.length; i++) {
  acc += nums[i];
}

// reduce를 이용해 명령형 코드의 수동 반복을 자동화하는 방식으로 추상화
_(nums).reduce((acc, current) => acc + current, 0);

// 재귀
function sum(arr) {
  if (_.isEmpty(arr)) {
    return 0;
  } 
  return _.first(arr) + sum(_.rest(arr)); // _.first, _.rest로 입력을 점점 줄여가며 자신을 호출한다
}

sum([]); //0
sum([1,2,3,4,5,6,7,8,9]); //45
```
  * 재귀는 변이가 없으므로, 더 강력하고 우수하며 표현적인 방식으로 반복을 대체할 수 있다
  * 재귀는 내부적으로 호출 스택에 겹겹이 쌓인다
  * 알고리즘이 종료 조건에 이르면 쌓인 스택이 런타임에 의해 즉시 풀리면서 반환문이 모두 실행되어 덧셈을 한다

꼬리 호출 최적화tail-call optimization
* ES6에서는 꼬리 호출 최적화가 추가되면서 사실상 재귀와 수동 반복의 성능 차이는 미미해졌다
  ```js
  function sum(arr, acc = 0) {
    if (_.isEmpty(arr)) {
      return 0;
    }
    return sum(_.rest(arr), acc + _.first(arr)); 
  }
  ```
    * 함수 본체의 가장 마지막 단계, 즉 꼬리 위치에서 재귀 호출을 한다
    * 이후 7장에서 함수형 최적화를 언급하면서 어떤 이점이 있는지를 언급할 것이다

### 3.5.3 재귀적으로 정의한 자료구조

노드 기반의 단순한 자료구조
* 트리는 다양한 분야에 쓰이는 아주 일반적인 자료구조다
* 그런데 배열처럼 평탄한 자료구조를 파싱할 때 사용하는 함수형 기법은 트리 구조의 데이터에는 적절하지 않다
* 자바스크립트는 언어 자체로 내장 트리 객체를 지원하지는 않으므로 노드 기반의 단순한 자료구조를 만들어야 한다
* 노드는 값을 지닌 객체로 자신의 부모와 자식 배열을 레퍼런스로 참조한다

노드 객체
```js
class Node {
  constructor(val) {
    this._val = val;
    this._parent = null;
    this._children = [];
  }

  isRoot() {
    return isValid(this._parent);
  }

  get children() {
    return this._children;
  }

  hasChildren() {
    return this._children.length > 0;
  }

  get value() {
    return this._val;
  }

  set value(val) {
    this._val = val;
  }

  append(child) {
    child._parent = this;
    this._children.push(child);
    return this;
  }

  toString() {
    return `Node (val: ${this._val}, children: ${this._children.length})`;
  }
} 

// 노드는 이렇게 생성한다
const church = new Node(new Person('Alonzo', 'Church', '111-11-1111')); 
// (...) 다른 객체들을 같은 방식으로 생성했다고 가정하자

church.append(rosser).append(turing).append(kleene);
kleene.append(nelson).append(constable);
rosser.append(mendelson).append(sacks);
turing.append(gandy);
```
  * 노드의 메인 로직은 append 메서드에 있다
  * 특정 노드에 자식을 붙일 때 자식이 부모 레퍼런스로 해당 노드를 가리키게 하고 자식 노드를 리스트에 추가한다
  * append를 이용해 루트부터 연결하면 트리가 완성된다

트리 객체
```js
class Tree {
  constructor(root) {
    this._root = root;
  }

  static map(node, fn, tree = null) {
    node.value = fn(node.value);
    if (tree === null) {
      tree = new Tree(node);
    }

    if (node.hasChildren()) {
      _.map(node.children, function (child) {
        Tree.map(child, fn, tree);
      });
    }
    return tree;
  }

  get root() {
    return this._root;
  }
} 

Tree.map(church, p => p.fullname);
```
  * 루트 원소의 데이터를 표시하고
  * 전위 함수를 재귀 호출해 왼쪽 하위 트리를 탐색한다
  * 같은 방법으로 오른쪽 하위 트리를 탐색한다

데이터 캡슐화
* 변이 및 부수효과 없는 자료형을 다룰 때 데이터 자체를 캡슐화해 데이터에 접근하는 방법을 통제하는 것이 FP의 관건이다
* 자료 구조 파싱은 소프트웨어의 기본기이자 FP의 주특기이다

### 3.6 마치며

내용 정리
* 고계함수 map, reduce, filter를 쓰면 코드를 확장할 수 있다
* 로대시JS는 데이터 흐름과 변환 과정이 명확히 구획된 제어 체인을 통해 데이터 처리 및 프로그램 작성을 도모한다
* FP의 선언적 스타일로 개발하면 코드를 파악하기 쉽다
* 고수준의 추상화를 SQL 어휘로 매핑하면 더 심도있게 데이터를 이해할 수 있다
* 재귀는 자기 반복적 문제를 해결하는 데 쓰이며, 정의된 자료구조를 재귀적으로 파싱해야 한다