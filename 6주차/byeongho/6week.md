# 15. JUnit 들여다 보기

> **JUnit**
> 자바 프로그래밍 언어용 유닛 테스트 프레임워크

JUnit에도 실제 코드들을 살펴보면 그 동안 배운 부분이 적용되지 않고 개선되어야할 부분들이 보이기도 한다.

그 중 해당 책에서는 `ComparisionCompactor`모듈을 통해 실제 리팩토링하는 과정을 보여주고 있다. (목록 15-2의 코드를 확인하면 된다.)

### 접두어 f의 제거

<pre><code>
private int fContextLength;
ptivate int contextLength;
</code></pre>

위와 같이 거슬리면서 범위를 명시하는 중복되는 정보인 접두어 f를 제거한다.

### 조건문의 캡슐화


<pre><code>
if (expected == null || actual == null || areStringsEqual()) {
  return Assert.format(message, expected, actual);
}

if (shouldNotCompact()) {
  return Assert.format(message, expected, actual);
}

private boolean shouldNotCompact() {
  return expected == null || actual == null || areStringsEqual();
}
</code></pre>

위와 같이 조건문을 메서드로 뽑아내 적절한 이름을 붙인다.

### 명확한 변수명
<pre><code>
String expected = compactString(fExpected);
String actual = compactString(fActual);

String compactExpected = compactString(expected);
String compactActual = compactString(actual);
</code></pre>

### 부정문에서의 긍정문으로의 변경

<pre><code>
private boolean shouldNotCompact() {
  return expected == null || actual == null || areStringsEqual();
}

private boolean canBeCompacted() {
  return expected != null && actual != null && areStringsEqual();
}
</code></pre>

부정문보다 긍정문이 더 이해하기 쉽기에 이로 변경한다.

### 적절한 함수 이름

<pre><code>
private boolean compact() {
}

private String formatCompactedComparison(String message) {}
</code></pre>

실제 함수가 하는 역할을 표현할 수 있도록 한다.



### 함수 분리

<pre><code>
private String compactExpected;
private String compactActual;

public String formatCompactedComparison(String message) {
  if (canBeCompacted()) {
    compactExpectedAndActual(); // 압축
    return Assert.format(message, compactExpected, compactActual); // 형식 맞추기
  } else {
    return Assert.format(message, expected, actual);
  }
}
	
private void compactExpectedAndActual() {
  findCommonPrefix();
  findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}
</code></pre>

실제 하는 역할에 맞추어 함수를 분리하여 표현하도록 한다.

### 일관적인 함수 사용방식

<pre><code>
private void compactExpectedAndActual() {
  findCommonPrefix();
  findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

private void compactExpectedAndActual() {
  prefixlndex = findCommonPrefix();
  suffixlndex = findCommonSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}
</code></pre>

`findCommonPrefix`, `findCommonSuffix` 는 반환 값이 없다. 그렇기에 이러한 형식을 맞추도록 한다.

### 숨겨진 시간적인 결합

<pre><code>
private void compactExpectedAndActual() { 
  prefixIndex = findCommonPrefix(); 
  suffixIndex = findCommonSuffix(prefixlndex); 
  compactExpected = compactString(expected); 
  compactActual = compactString(actual);
}
</code></pre>

findCommonSuffix 는 findCommonPrefix가 prefixIndex를 계산한다는 사실에 의존한다는 시간적인 결함이 있다. 즉, 함수 호출 순서가 바뀔 경우 오류를 찾아내기 어렵다. 그렇기에 위와 같은 방식으로 코드를 수정한다.

하지만 위 방법은 prefixIndex가 필요한 이유를 설명하지 못하기에 다음과 같은 방안을 제시한다.

<pre><code>
private void compatExpectedAndActual() {
  findCommonPrefixAndSuffix();
  compactExpected = compactString(expected);
  compactActual = compactString(actual);
}

private void findCommonPrefixAndSuffix() {
  findCommonPrefix();
}
</code></pre>

# 16. SerialDate 리팩토링

> **SerialDate**
> 날짜를 표현하는 자바 클래스

## 실제 SerialDateTests 상황

해당하는 클래스는 단위 테스트 케이스를 포함하나 실제 테스트 커버리지는 대략 50%정도밖에 되지 않는다.

이에 책의 저자는 독자적으로 단위 테스트 케이스를 구현하였다. (실제 코드의 논리적 오류로 인해 결코 실행되지 않는 코드도 존재한다.)

### 변경이력의 제거

현재 소스 코드 제어 도구를 사용하기에 이를 삭제해도 상관 없다.

### import 문의 간결화

import 문의 경우 `java.text.*`, `java.util.*`과 같이 줄여도 된다.

### 서술적인 이름의 사용

서술적인 이름으로 용어가 잘 표현되어야 한다.

- SerialDate라는 이름은 구현을 암시하지만 실상은 추상클래스로 실제 추상화 수준이 올바르지 않다. 그렇기에 DayDate로 클래스 이름을 변경한다.

### Enum의 사용

<pre><code>
public static enum Month {
    JANUARY(1)
    FEBRUARYY(2)
    ...
    DECEMBER(12)
}
</code></pre>

### 변수 위치의 변경

변수의 경우 해당 변수를 사용하거나 관련한 책임이 존재하는 클래스로 옮겨야 한다.

### 기반 클래스와 파생 클래스

기반 클래스는 파생 클래스를 몰라야한다. 그렇기에 `ABSTRACT FACTORY 패턴`을 적용하여 객체를 생성하여 구현관련 질문에 대답할 수 있도록 한다.

<pre><code>
public abstract class DayDateFactory {
  private static DayDateFactory factory = new SpreadsheetDateFactory();
  public static void setInstance(DayDateFactory factory) {
    DayDateFactory.factory = factory;
  }
  
  protected abstract DayDate _makeDate(int ordinal);
  protected abstract DayDate _makeDate(int day, int month, int year);
    
  public static DayDate makeDate(int ordinal) {
    return factory._makeDate(ordinal);
  }
  
  public static DayDate makeDate(int day, int month, int year) {
    return factory._makeDate(day, month, year);
  }

}
</code></pre>

### 불필요한 final의 키워드 제거

실질적인 가치를 지니지 않는 final을 제거한다. 이는 코드만 복잡하게 만들기 떄문이다.

### 중첩된 if문의 제거

중첩된 if문은 ||, &&, 메서드를 이용해 하나로 만든다.



## 결론

실제 사용중인 코드들을 리팩토링해보고 커버리즈를 증가시켜보았다. 이러한 과정에서 보이스카우트 규칙을 따르면서 계속해서 코드를 개선해 나가야 한다.