#  람다 표현식.  
- C++11에 처음 도입된 후 C++14 및 C++17에서 확장됨.   
- C++ 컴파일러는 behind the scene에서는 람다 표현식에서 익명 함수 객체를 생성해 사용함.   
- Stateless lambda: 매개변수 리스트를 통해 전달된 정보에 대해서만 알고 있음(일반 함수처럼).   
- Stateful lambda: 실행된 환경(environment)의 element를 capture할 수 있기 때문에 더 복잡함.   
-> closure의 개념 등장. 람다 캡쳐 목록을 통해 발생.   
   
## 람다식 등장 이유.  
- C++11 이전에는 함수 객체(주로 STL과 함께)와 함수 포인터(주로 callback으로)가 자주 사용됨.   
-> 문제는 종종 알고리즘을 제어하기 위한 짧은 함수들을 많이 작성해야 한다는 것.      
-> 이러한 함수들을 작은 클래스로 캡슐화하여 함수 객체로 만들어 사용함.   
-> 클래스나 함수가 사용 시점에서 멀리 떨어져서 정의됨. 코드 수정, 유지 보수및 테스트에 문제 발생 가능.   
    ex) STL 알고리즘과 함께 함수 객체 사용한 경우 코드 이해하기 위해서는 함수 객체가 정의된 클래스를 찾고, 함수 객체를 찾아 읽어야 함. 
    또한 컴파일러는 인라인으로 정의되지 않은 함수의 경우 최적화에 능숙하지 않음.   
-> 람다식 사용하면 이러한 문제 해결 가능해짐.   
    But 람다식은 코드 양이 적을 때 사용하는 것이 좋음. 복잡한 코드라면 함수 객체 사용을 고려해야 함.   
- 람다식의 또다른 이점은 컴파일러가 람다에서 클로저 생성한다는 것.   
-> 람다가 존재하는 환경에 접근할 수 있음릉 의미함.

## 람다식의 구조.  
### [] () -> return_type specifiers {};
- [] 는 람다식의 시작을 정의. 람다식이 실행되는 컨텍스트나 클로저를 캡처할 수 있는 캡처리스트이기도 함.   
[] 안에 넣은 내용에 따라 컴파일러에게 캡처하길 원하는 요소(참조 or 값) 알릴 수 있음.   
-  ()는 매개변수 리스트임. 람다에 전달됨.   
- -> return_type 은 람다식의 반환 타입임. void일 경우 생략 가능. + 람다가 단순하면 컴파일러가 람다의 반환 타입 추론 가능 -> 생략 가능.   
- specifiers 는 선택적임. mutable or const exper.
- {} 안에는 실행하려는 코드 작성함.   
ex) [] () { std::cout << "Hi"; };
[] () { std::cout << "Hi"; }(); 처럼 호출 가능. -> 람다식에서 함수 객체를 인스턴스화 하고 오버로드된 ()를 사용해 함수 객체를 호출함.   
   
ex 2) auto l = [] () { std::cout << "Hi"; };
        l(); 처럼 auto로 컴파일러에게 람다식의 타입 추론하게 하여 변수에 저장 후 사용할 수 있음.   
   
- 이러한 람다식은 모두 stateless 람다식임. 캡처리스트가 비어있음.

## Stateful lambda   
### [captured_variables] () -> return_type specifiers {};   
 - stateless 람다로 수행하는 모든 작업은 모두 stateful 람다로 수행 가능함.   
 - scope 내에서 선언된 모든 변수 사용(캡처) 가능.   
 -> [] 안에 사용하려는 변수를 나열함으로써 매개변수로 람다에 전달할 필요 없이 변수 사용 가능해짐.
 
 ex) int y{10};   
 auto l = [y] (int x) { std::cout << x + y; };   
    
 -> 내부적으로는   
 class CompilerGenerated {   
private:    
&nbsp;&nbsp;&nbsp;int y;   
public:   
&nbsp;&nbsp;&nbsp;CompilerGenerated(int y): y{y} {};   
&nbsp;&nbsp;&nbsp;void operator() (int x) const {   
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::cout << x + y;   
&nbsp;&nbsp;&nbsp;}   
};   
 처럼 컴파일러에 의해 함수 객체화 되어 사용됨.   
-> 캡처된 변수를 멤버 변수로 사용, 매개변수로 캡처된 변수 받아 멤버로 초기화하는 생성자 사용하여 인스턴스화됨. 멤버는 수정 불가(캡처된 값은 const로 캡처됨).   
-> 변수 수정 불가능해서 특정 기능 구현히 상당히 제한적일 수 있음.   
-> C++은 캡처할 수 있는 다양한 방법 제공.   

ex) [x] () {}; -> value로 캡처. 람다 내에서 변수 수정 불가능.   
[x] () mutable {}; -> mutable 사용하여 값으로 캡처 시 람다 내에서 수정 가능해짐(non const 멤버로 생성).   
[&x] () {}; -> 참조로 캡처 시 람다 내에서 수정 가능. + 람다 외부에서도 람다 내부에 캡처된 변수 visible 함.   

### default capture
- 람다 내에서 사용된 모든 변수들 캡처.   
ex)  
[=] -> 값으로 캡처, [&] -> 참조로 캡처, [this] -> this object를 참조로 캡처.   
   
[=, &x] 처럼.default 캡처와 동시에 변수 캡처(default 캡처와 다른 방식, 같다면 컴파일 안됨) 사용 가능.  
-> default 캡처가 먼저 배치되어야 함.

## 람다는 보통 STL의 predicate or operator로 많이 사용됨.   
- predicate는 몇 개의 인자를 받고, boolean을 return하는 함수를 말함.   
- operator는 몇 개의 인자 받아 그 인자들에 어떤 연산을 적용하는 함수를 말함.   




