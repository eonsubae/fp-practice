# 함수형 자바스크립트, 루이스 아텐시오 내용정리
---
## 1장 함수형 길들이기
* 이 장에서 다루는 내용들
  * 함수형 사고방식
  * 함수형 프로그래밍의 정의와 필요성
  * 불변성, 순수함수 원리
  * 함수형 프로그래밍 기법 및 그것이 설계 전반에 미치는 영향

* 복잡성의 증가
  * 현재 웹과 관련된 JS진영은 과거와 달리 복잡성이 크게 증가
  * 대규모 서버 애플리케이션, 대부분의 비즈니스 로직을 클라이언트 측에 두기가 기술적으로 가능
  * 여러 저장소와 통신하며 비동기 작업을 생성하고 이벤트를 처리하는 등 다양한 기술이 필요해짐
  
* 함수형 프로그래밍
  * 이런 복잡성들은 객체지향 방식으로도 해결 가능
  * 하지만 자바스크립트는 상태 공유가 보편적이라 관리하기 어려운 코드가 되기 쉬움
  * 최근 리액티브 프로그래밍의 유행도 데이터흐름과 변경 전파의 중요성을 반증
  * 데이터와 데이터를 다루는 함수에 대해 진지하게 고민하는 패러다임의 필요성 대두

* 애플리케이션의 필수 설계 요소
  1. 확장성 : 추가 기능 지원을 위해 계속 리팩토링이 필요한가
  2. 모듈화 용이성 : 파일 하나를 고치면 다른 파일도 영향을 받는가
  3. 재사용성 : 중복이 많은가
  4. 테스트성 : 함수를 단위 테스트하기 어려운가
  5. 헤아리기 쉬움 : 체계도 없고 따라가기 어려운 코드인가
  * 함수형은 특정 API가 아닌 위의 문제들을 해결하려는 하나의 패러다임이다

### 1.1 함수형 프로그래밍은 과연 유용한가?
* 자바스크립트의 단점 보완
  * FP는 깔끔하고 모듈화하기 좋으며 테스트가 쉬운 간결한 JS코드 작성에 도움이 된다
  * JS는 상태관리를 개발자에 떠넘기는 동적인 언어이므로 버그를 양산해왔다
  * JS코드를 함수형으로 작성하면 대부분의 문제가 해결된다
  * 순수함수로 구현한 코드는 전역상태를 깨뜨리지 않아 복잡하고 거대한 앱도 통제가 쉽다
  * 따라서 전체 애플리케이션 품질도 향상시키며 동시에 JS자체의 이해도 증진시킨다

### 1.2 함수형 프로그래밍 
* 정확한 개념 정의의 중요성
  * FP는 함수 사용을 강조하는 개발 패러다임
  * 많은 사람들은 과거에 함수를 사용해온 경험을 말하며 차이점이 무엇인지 인식하기 어려워함
  * 함수형의 진짜 목표
    * 부수효과(Side effect)를 방지하고 상태 변이(Mutation of state)를 감소시키기 위해
    * 데이터의 제어 흐름과 연산을 추상(abstract)화하는 것이다

* 간단한 예제로 시작하기
  ~~~
  <body> // 이후 코드에선 body 태그 생략
    <div id="msg"></div>
    <script>
      document.querySelector('#msg').innerHTML = '<h1>Hello World</h1>';
    </script>
  </body>
  ~~~
  * 이 예제는 하드코딩되어서 메시지를 동적으로 표시할 수 없다

  ~~~
  function printMessage(elementId, format, message) {
      document.querySelector(`#${elementId}`).innerHTML =
          `<${format}>${message}</${format}>`;
  }

  printMessage('msg', 'h1', 'Hello World');
  ~~~
  * 재사용가능한 코드가 되었으나 HTML이 아닌 다른 형식에 사용할 수 없어(ex. 파일) 완벽하지 않다
  * 이 문제를 해결하려면 특정 기능을 함수에 추가해 매개변수로 전달하는  함수의 매개변수화가 필요
  * FP에서는 여러 함수를 서로 합성해 더 많은 기능을 탑재해야 한다

  ~~~
  var printMessage = run(addToDom('msg'), h1, echo);
  printMessage('Hello World');
  ~~~
  * h1은 스칼라 값이 아닌 addToDom ,echo와 같은 함수가 되었다
  * 이와 같이 FP에서는 작은 함수들을 재료로 새로운 함수로 만들어낸다
  * FP에서는 더 작은 조각들로 프로그램을 나눈 후 더 쉬운 형태로 다시 조합하는 과정을 거친다
  * run 함수는 addToDom, h1, echo 함수를 연결해 한 함수의 반환값을 다른 함수의 입력으로 전달
  * echo함수가 Hello World를 반환하면 h1에 전달되고, h1의 결과값이 addToDom에 전달되는 구조다

  ~~~
  var printMessage = run(console.log, repeat(2), h1, echo);
  printMessage('Get Functional');
  ~~~
  * 앞선 코드들의 흐름을 익히면 위 코드의 흐름을 예측하는 것이 어렵지 않다
  * 이처럼 시각적인 명료함은 우연의 산물이 아니다
  * FP는 FP가 아닌 코드와 근본적으로 스타일이 다르다
  * 이는 FP 특유의 선언적 개발 방식 때문이다
  * 이를 온전히 이해하려면 다음과 같은 개념을 숙지해야 한다
    * 선언적 프로그래밍
    * 순수함수
    * 참조 투명성
    * 불변성

#### 1.2.1 함수형 프로그래밍은 선언적(declarative) 
* 선언적이라는 말은 내부 구현과 데이터 흐름을 감추고 연산/작업을 표현하는 사상을 의미한다
* 기존의 익숙한 방식은 명령형(imperative) 또는 절차적(procedural) 방식
  * 특정 결과를 위해 시스템의 상태를 변경하는 구문을 위에서 아래로 늘어놓는 수열방식
  ~~~
  // 명령형 또는 절차적 방식의 코드의 예(배열의 모든 원소 제곱)
  var array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
    for(let i = 0; i < array.length; i++) {
        array[i] = Math.pow(array[i], 2);
    }
  console.log(array);
  ~~~
* 선언적 프로그래밍은 서술부(description)와 평가부(evaluation)를 분리
  * 제어흐름, 상태변화를 특정하지 않고도 로직이 무엇인지 표현식(expression)으로 나타냄
  * 내부 메커니즘은 추상화하고 데이터를 가져오는 SQL 구문도 선언적 프로그래밍의 한 예
  ~~~
  // 앞서 보았던 코드를 선언적 프로그래밍으로 변화시키면 다음과 같다
  [0, 1, 2, 3, 4, 5, 6, 7, 8, 9].map(num => Math.pow(num, 2));
  // 람다 표현식을 사용하는 습관을 들이자
  // 람다 표현식은 함수 호출 구조의 가장 중요한 부분만 남기므로 코드를 덜 쓴다
  ~~~
  * 선언적 코드는 루프 제어를 시스템의 다른 파트(Array.map)에 맡기고 각 요소의 처리에만 집중한다
  * 이전 코드와 비교하면 루프 카운터 관리, 정확한 배열 인덱스 접근같은 신경을 쓸 부담이 없다
  * 제어문은 코드가 길어지면 버그 가능성도 높고 함수로 추상하지 않는한 재사용도 안된다
  * 수동 루프를 제거하고 함수를 매개변수로 받는 map, reduce, filter 같은 고계함수를 사용하자
    * 재사용성, 확장성을 획득하는 장점이 있다
    * 앞서 run함수를 이용해 한 일도 같은 맥락에 있다
* 루프를 제거해야 하는 이유
  * 루프는 재사용이 어렵고 다른 연산에 끼워 넣기도 힘든 명령형 제어 구조물
  * 또 성격상 반복할 때마다 값이나 상태를 계속 변경시킨다
  * FP는 무상태성(statelessness)과 불변성(immutability)을 지향한다
  * 무상태코드는 전역 상태를 바꾸거나 혼선을 일으킬 가능성이 없다
  * 상태를 두지 않으려면 부수효과와 상태 변이를 일으키지 않는 순수함수(pure function)를 써야한다

#### 1.2.2 순수함수와 부수효과
