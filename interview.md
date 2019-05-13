
### 경력 설명

#### 라이머스 
##### 설명

##### 질문


#### 에이피엑스 소프트
##### 설명

##### 질문

#### 로지웨어
##### 설명

##### 질문


#### 제니브레인
##### 설명

##### 질문


#### 펜타비전 
##### 설명

##### 질문


###  질문



#### C++ 

#####  C++ r-value란 
- 우측값 참조
- C++ 11에서 추가 됨.
- C++에서 모든 표현식은 Lvalue 또는 Rvalue 
- Lvalue는 단일 표현식 이후에도 없어지지 않고 지속되는 객체
- Rvalue는 표현식이 종료된 이후에는 더이상 존재하지 않는 임시적인 값
- ++x 는  Lvalue , x++는 Rvalue.
- C++ Rvalue를 참조할 수 있는 Rvalue 참조자가 추가
- Move Semantics란 객체의 리소스(동적으로 할당 된 메모리와 같은)를 또 다른 객체로 전송(이동)하는 것을 의미
- 불필요한 복사를 줄이는 것을 목적
- std::move()는 의미 이동을 위한 도구중 하나입니다.


```
MyObj b = a; 는 복사 생성을 하게 됩니다.
MyObj b = std::move(a); 라고 하게 되면, b는 이동 생성을 하게 됩니다.
이런 식으로 경우에 따라서 복사를 이동으로 변환하여 처리 할 수도 있습니다.
이런 강제 이동의 이면에는 이에 따른 불안 요소를 남기게 됩니다.
즉, b로 강제 이동을 했지만, a는 현재 살아있는 상태(변수이기 때문에)이기 때문에 만약 a를 사용하려 한다면, 
a.__data 는 nullptr 상태이기 때문에 문제가 생깁니다.

즉, std::move() 를 사용할 수 있는 경우는 a를 더이상 사용 안해야 합니다.
```



##### C++ 11에서 주로 써본것


##### 가상함수/소멸자
먼저 상속을 받은 클래스의 생성과 소멸 과정을 보자. 생성자는 부모 클래스의 생성자가 먼저 불려지고, 
소멸자는 자식 클래스의 소멸자가 먼저 불려지고 나서 부모 클래스의 소멸자가 불려진다.

그런데 다형성 이용을 위해 부모 클래스의 포인터로부터 자식 클래스를 호출할 때, 
가상 함수로 정의되지 않은 자식 클래스의 오버라이딩된 함수를 호출하면 부모 클래스의 멤버 함수가 호출된다.
소멸자도 자식 클래스에서 오버라이딩된 함수라고 볼 수 있기 때문에 만약 부모 포인터로 객체를 삭제하면 부모 클래스의 소멸자가 호출된다. 

따라서 소멸자를 가상 함수로 선언하지 않으면 이 경우 자식 클래스의 소멸자는 결코 호출되지 않는다.
가상 함수 키워드 virtual이 사용되었다면 이것은 자식 클래스에서 재정의될 수 있음을 명시하기 때문에 
포인터의 종류에 상관없이 항상 자식 클래스의 메서드가 호출된다. 즉, 자식 클래스의 소멸자가 호출되고 나서 부모 클래스의 소멸자가 호출된다.

따라서 상속 관계가 있고 소멸자에서 리소스를 해제해야 하는 경우 반드시 소멸자를 가상 함수로 선언해야 한다.

##### 오버로딩/오버라이딩
- 오버로딩(Overloading) : 같은 이름의 메소드를 여러 개 가지면서 매개변수의 유형과 개수가 다르도록 하는 기술

```java
public class Overloadingtest {

    // test() 호출
    void test(){
        System.out.println("매개변수 없음");
    }
    
    // test에 매개변수로 int형 2개 호출
    void test(int a, int b){
        System.out.println("매개변수 "+ a + "와 " + b);
    }
    
    // test에 매개변수 double형 1개 호출
    void test(double d){
        System.out.println("매개변수 " + d);
    }
}
```

- 오버라이딩(Overriding) : 상위 클래스가 가지고 있는 메소드를 하위 클래스가 재정의 해서 사용한다


```java
public class Employee{
    
    public String name;
    public int age;
    
    // print() 메소드
    public void print(){
        System.out.println("사원의 이름은 "+this.name+ "이고, 나이는" + this.age+"입니다.");
    }   
}


// Employee 상속
public class Manager extends Employee{
    
    String jobOfManage;
    
    // print() 메소드 오버라이딩
    public void print(){
        System.out.println("사원의 이름은 "+this.name + "이고, 나이는" + this.age + "입니다.");
        System.out.println("관리자 "+this.name+"은 "+this.jobOfManage+" 담당입니다.");
    }
}
```

##### git rebase 관련 기능

#####  deque, vector, map 

##### C++ 과 C#의 차이점은?

#####  share_ptr, auto_ptr
- auto_ptr은 포인터의 소유권 문제가 있다
- auto_ptr 객체 자체는 복사가 필요한 곳에 사용을 할 수 없다
```cpp
class AAA;
 
// RAII 방식으로... AAA 객체 생성
std::auto_ptr<AAA> AAAObject(new AAA());
 
// 복사가 되는 순간, AAAObject는 NULL이 되고, 이제 BBBObject 만이 객체를 가리킨다.
std::auto_ptr<AAA> BBBObject(AAAObject);
 
// 역시 대입이 되는 순간, BBB는 NULL, 이제 AAA가 객체를 가리킴.
AAAObject = BBBObject;

```
- share_ptr 객체 소유권을 이곳 저곳에서 관리가능하다
- 객체의 소유권을 가지고 있기 때문에 객체의 소유권이 올바르게 회수되었는지를 확인하기 위해 Reference Counting 기법을 사용한다. 
- 레퍼런스 카운팅 기반이기에 순환 참조에 대한 잠재적인 문제가 있을 수 있다.
- A와 B가 서로에 대한 shared_ptr을 들고 있으면 레퍼런스 카운트가 0이 되지 않아 메모리가 해제되지 않는다.
- 레퍼런스 카운트에 대해서만 동기화를 해서 멀티 쓰레드에서의 안정성을 얻는다.




##### 함수포인터란?
- 함수를 변수처럼 사용하기 위해 사용
- C++11 에서는 람다로 사용 (+ std::function)
- 

##### 디자인 패턴 알고 있는 것 설명

#### C#

#### Cocos2d-x
##### Cocos2d-x란?
#####  어떤 애니메이션에 관련되어서 SpriteSheet를 사용하는것이 이득인지 Image를 사용하는것이 이득인지 설명하시오
##### cocos2d-x 메모리 관리 방법
##### cocos2d-x의 사용 디자인 패턴 

### **프로토타입**
#### 원형이 되는 인스턴스를 사용하여 생성할 객체의 종류를 명시하고, 이렇게 만든 견본을 복사해서 새로운 객체를 생성합니다. (GoF의 디자인 패턴 169p)
- 인스턴스를 복제하여 사용하는 구조
- 생성할 객체들의 타입이 프로토타입 인스턴스로부터 결정되도록 하는 디자인 패턴
- 인스턴스는 새 객체를 만들기 위해 자신을 복제(clone)한다.
![Prototype](https://dzone.com/storage/temp/8861139-prototype-1.png)

### **Flyweight**
- 동일한 것을 공유하여 사용하는 구조
- 동일한 것을 공유하여 메모리 사용량을 줄임
- 가능한 많은 데이터를 서로 공유하여 사용하도록 하여 메모리 사용량을 최소화하는 디자인 패턴
- 어떤 객체의 개수가 너무 많아서 좀 더 가볍게 만들고 싶을 때 사용 한다.

![Flyweight](https://dzone.com/storage/temp/8861142-flyweight-1.png)

### **Bridge**
- 기능 계층과 구현 계층이 분리된 구조
- 구현부에서 추상층을 분리하여 각자 독립적으로 변형할 수 있게 하는 패턴
![Bridge](https://dzone.com/storage/temp/8515490-bridge-1.png )
![Bridge](https://dzone.com/storage/temp/8515491-bridge-2.png)

### **Composite**
- "그릇과 내용물을 동일시하기"
- 객체들의 관계를 트리 구조로 구성하여 부분-전체 계층을 표현하는 패턴
- 단일 객체와 복합 객체를 동일하게 다룰 수 있음
![Composite](https://dzone.com/storage/temp/8861156-composite-1.png )
![Composite](https://dzone.com/storage/temp/8861152-composite-2.png)

### **Command**
- 명령을 클래스로 표현하는 구조
- 요청을 객체의 형태로 캡슐화하는 디자인 패턴
- 서로 요청이 다른 사용자의 매개변수, 요청 저장, 로깅, 연산 취소를 지원함
- 요청 자체를 캡슐화 하는 것입니다. 이를 통해 서로 다른 사용자(client)를 매개변수로 만들고, 요청을 대기시키거나 로깅하며, 되돌릴 수 있는 연산을 지원합니다
-  메서드 호출을 실체화 한 것이다.
![Command](https://dzone.com/storage/temp/8861153-command-1.png)

### **Observer**
- 상태 변화를 알려주도록 하는 패턴
- 이벤트 핸들링 시스템 구현에 활용됨
- 어떤 일이 생기면 미리 등록한 객체들에게 통보해주는 패턴
- 옵저버들을 등록해두어 상태변화가 있을 때 통지받는 패턴
- 객체의 상태 변화를 관찰하는 관찰자들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메소드 등을 통해 객체가 각 Observer에게 통지하도록 하는 디자인 패턴
- 발행-구독 모델을 따르는 디자인 패턴
- 객체 사이에 일 대 다의 의존 관계를 정의해두어, 어떤 객체의 상태가 변할 때 그 객체에 의존성을 가진 다른 객체들이 그 변화를 통지 받고 자동으로 업에
![Observer](https://dzone.com/storage/temp/8861167-observer-1.png )

### **Component**
- 한 개체가 여러 분야를 서로 커플링 없이 다룰 수 있게 한다.
![Component](https://dzone.com/storage/temp/8861155-component-1.png )



##### 다른 해상도 / 스크린 / 크기 비율로 처리 방법
##### 백그라운드에서 무거운 작업을 수행하고 UI에 결과를 표시하는 방법?
##### 쉐이더 관련 질문

####  3D 
- 렌더링 파이프 라인
![렌더링 파이프라인](https://img1.daumcdn.net/thumb/R800x0/?scode=mtistory&fname=https%3A%2F%2Ft1.daumcdn.net%2Fcfile%2Ftistory%2F15481C335050AF7D20)

#### Unity 3D


##### 유니티에서 Start 와 Update 같은 함수가 가상함수로 정의되지 않은 이유
- 유니티의 메시지 시스템은 매직함수를 리스트 형태로 관리해, 상황에 맞게 호출하는 방식
- 그래서 클래스가 생성될때 리플렉션을 통해 해당 함수가 존재하는지 여부를 통해 리스트에 등록할지 정한다.
- 매직함수에 아무 내용이 없더라도, 선언되어 호출하는것 자체로도 메모리에 공간을 차지하고 부하가 발생한다.
- 게다가 Native C++ 에서 managed C#의 함수를 호출하는것이다.
- 간단한 게임일 경우는 상관없지만, 수천 수만개의 게임 오브젝트를 쓴다면 고려를 해야함.


#### Unreal
