# 1.3 자바 함수

프로그래밍 언어에서 `함수(function)`라는 용어는 메서드 특히 `정적 메서드(static method)`와 같은 의미로 사용된다.
<br> 하지만 자바의 함수는 이에 더해 `수학적인 함수`처럼 사용되며 부작용을 일으키지 않는 함수를 의미한다.<br>
자바 8에서는 멀티코어에서 병렬 프로그래밍을 활용할 수 있는 스트림과 연계될 수 있도록 함수를 새로운 값의 형식으로 추가했다.<br>


`함수를 값처럼 취급한다`라는 특징은 어떤 장점을 제공하는가?
<br><br>자바 프로그램에서 조작할 수 있는 값
- int, double 등의 기본값
- 객체
<br>

>**프로그래밍 언어의 핵심은 값을 바꾸는 것이다.** 
>프로그래밍에서는 이 값을 `일급 값` 또는 `일급 시민`이라고 부른다.
><br><br>자바에서 구조체(메서드, 클래스)는 값의 구조를 표현하는 데 도움은 될 수 있지만, 그 자체로 값이 될 수는 없다. 이를 `이급 시민`이라고 부른다.
> <br>이렇게 이급 시민의 메서드를 런타임에 전달 할 수 있다면, 즉 일급 시민으로 만들면 프로그래밍에 유용하게 사용할 수 있을 것이다.
> <br>그렇기에 자바 8 설계자들은 이급 시민을 일급 시민으로 바꿀 수 있는 기능을 추가했다.

---
## 메서드와 람다를 일급 시민으로

### 메서드 참조

자바 8이전에는 메서드가 값이 될 수 없었기 때문에 메서드를 객체로 `인스턴스화`하여 전달해야 했다.<br>
하지만 자바 8의 메서드 참조(::)를 이용하면 메서드 자체가 함수의 파라미터로 전달될 수 있다.<br>

ex) 디렉터리에서 모든 숨겨진 파일을 필터링

```java
//자바 8 이전
File[]hiddenFiles=new File(".").listFiles(new FileFilter(){
    public boolean accept(File file){
        return file.isHidden(); // 숨겨진 파일 필터링
    }
})

//자바 8이후 메서드 참조 적용
File[]hiddenFiles=new File(".").listFiles(File::isHidden);
```

- `isHidden`이라는 함수는 준비되어 있으므로 `자바 8의 메서드 참조::`(이 메서드를 값으로 사용하라는 의미)를 이용해 listFiles에 값을 직접 전달할 수 있게 되었다.<br>
기존에 `객체 참조`(new로 객체 참조를 생성함)를 이용해서 객체를 전달했던 것처럼 자바 8에서는 File::isHidden을 이용해서 `메서드 참조`를 만들어 전달할 수 있게 되었다.
### 람다 : 익명 함수

자바 8에서는 (기명-named) `메서드`를 일급값으로 취급할 뿐 아니라 `람다`(또는 익명 함수-anonymous functions)를 포함하여 함수도 값으로 취급할 수 있다.<br>
클래스에 직접 메서드를 정의할 수도 있지만, 이용할 수 있는 편리한 클래스나 메서드가 없을 때 새로운 람다 문법을 이용하면 `간결하게 코드를 구현`할 수 있다.<br>
람다 문법 형식으로 구현된 프로그램을 **함수형 프로그래밍**, 즉 `함수를 일급값으로 넘겨주는 프로그램을 구현한다`라고 한다.

## 코드 넘겨주기 : 예제

Apple 클래스와 `getColor` 메서드가 있고, Apples 리스트를 포함하는 변수 inventory가 있다고 가정하고, 모든 녹색 사과를 선택해서 리스트를 반환하는 프로그램을 구현해보자<br>
> 이처럼 특정 항목을 선택해서 반환하는 동작을 `필터`(filter)라고 한다.

자바 8 이전 

```java
public static List<Apple> filterGreenApple(List<Apple> inventory){
    List<Apple> result = new ArrayList<>();
    
        for(Apple apple:inventory){
            if(GREEN.equals(apple.getColor())){
                result.add(apple);
            }
        }
        return result;
}
```
만약 누군가 색상이 아닌 사과 무게로 필터링하고 싶다면? <br>
위 코드를 복사&붙여넣기해서 `조건만 다른 같은 구조`의 코드를 구성할 것이다.<br>
-> 복붙한 코드(동일 구조) 중 버그가 발생할 경우 **모든 코드를 고쳐야 한다**는 큰 단점이 존재한다. 
```java
public static List<Apple> filterHeavyApples(List<Apple> inventory){
    List<Apple> result = new ArrayList<>();
        for(Apple apple:inventory){
            if(apple.getWeight() > 150){ 
                result.add(apple);
            }
        }
        return result;
}
```

하지만 자바 8에서는 코드를 인수로 넘겨줄 수 있으므로 filter 관련 메서드를 중복으로 구현할 필요가 없다.

```java
public static boolean isGreenApple(Apple apple){
    return GREEN.equals(apple.getColor());
}

public static boolean isHeavyApple(Apple apple){
        return apple.getWeight() > 150;
}

public interface Predicate<T> {
    boolean test(T t);
}

static List<Apple> filterApples(List<Apple> inventory, Predicate<Apple> p) { //구현 메서드가 p라는 이름의 Predicate Parameter 로 전달
    List<Apple> result = new ArrayList<>();
    for (Apple apple : inventory) {
        if (p.test(apple)) { //사과는 p가 제시하는 조건에 맞는가 ? 
            result.add(apple);
        }
    }
    return result;
}

// 호출 측 코드 - 간결하다.
filterApples(inventory, Apple::isGreenApple);

filterApples(inventory, Apple::isHeavyApple);
```

>`프레디케이트(Predicate)`란 무엇인가?
> - 수학에서 인수로 값을 받아 true, false를 반환하는 함수를 의미하는 용어
> - 여러 조건을 검증하기 위해 코드를 복사 & 붙여넣기 하는 방식(버그 발생 시 모든 코드 수정)이 아닌 검증 메서드만 추가로 작성하면 된다.
> - 자바 8에서 `Function<Apple, Boolean>` 같이 구현 가능하지만, `Predicate<Apple>`이 더 표준 방식이며, `boolean-> Boolean` 변환 과정도 필요 없다.


## 메서드 전달에서 람다로
위의 코드에서 메서드를 값으로 전달하는 것은 분명 유용한 기능이다 <br>
하지만, 만약 `isHeavyApple()`, `isGreenApple()`가 한두 번만 사용하는 메서드라면? : 매번 정의하는 것은 귀찮은 일이다.<br>
자바 8에서는 이러한 문제를 간단하게 `람다(익명함수)`로 해결하며 코드를 구현할 수 있다.

`filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor);`<br>
`filterApples(inventory, (Apple a) -> a.getWeight() > 150);`<br>
위 두 가지를 합쳐서 아래처럼 코드를 구현할 수도 있다.<br>
`filterApples(inventory, (Apple a) -> a.getWeight() > 150 || GREEN.equals(a.getColor()));`<br>

즉, 한 번만 사용할 메서드는 따로 정의를 구현할 필요가 없다. -> 우리가 넘겨주려는 코드를 애써 찾을 필요가 없을 정도로 짧고 간결하다.<br>
하지만 람다가 복잡한 동작을 수행하는 상황이라면, 익명 람다보다는 코드가 수행하는 일을 잘 설명하는 이름을 가진 메서드를 정의하고 메서드 참조를 활용하는 것이 바람직하다.<br>
`코드의 명확성이 우선시 되어야 한다.`