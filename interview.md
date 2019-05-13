
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

```cpp

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
- 오버라이딩(Overriding) : 상위 클래스가 가지고 있는 메소드를 하위 클래스가 재정의 해서 사용한다.
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





##### 유니티에서 Start 와 Update 같은 함수가 가상함수로 정의되지 않은 이유
- 유니티의 메시지 시스템은 매직함수를 리스트 형태로 관리해, 상황에 맞게 호출하는 방식
- 그래서 클래스가 생성될때 리플렉션을 통해 해당 함수가 존재하는지 여부를 통해 리스트에 등록할지 정한다.
- 매직함수에 아무 내용이 없더라도, 선언되어 호출하는것 자체로도 메모리에 공간을 차지하고 부하가 발생한다.
- 게다가 Native C++ 에서 managed C#의 함수를 호출하는것이다.
- 간단한 게임일 경우는 상관없지만, 수천 수만개의 게임 오브젝트를 쓴다면 고려를 해야함.

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
 - https://dzone.com/articles/design-patterns-in-cocos2d-x 

##### 다른 해상도 / 스크린 / 크기 비율로 처리 방법
##### 백그라운드에서 무거운 작업을 수행하고 UI에 결과를 표시하는 방법?
##### 쉐이더 관련 질문

####  3D 
- 렌더링 파이프 라인

#### Unity 3D


#### Unreal
