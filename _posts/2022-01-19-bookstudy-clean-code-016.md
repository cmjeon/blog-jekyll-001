---
layout: single
title: 클린코드 스터디 - 016
categories: 
  - bookstudy - cleancode
tags: 
  - cleancode
---

## 16장 SerialDate 리팩터링

SerialDate 는 org.jfree.date 라는 패키지에 있는 날짜를 표현하는 자바 클래스임

이 장에서는 SerialDate 를 리팩토링 해보겠음

> p.448 부록 B-1 은 리팩토링 전 코드
> p.512 부록 B-7 은 리팩토링 후 코드

### 첫째, 돌려보자

SerialDate의 테스트 케이스인 SerialDateTests 라는 클래스는 모든 메서드를 테스트 하지않음

클래스를 철저히 이해하고 리팩터링하려면 훨씬 높은 테스트 커버리지가 필요로 하기 때문에 독자적인 단위 테스트 케이스를 구현하였음(p.484 B-4)

SerialDate 를 리팩터링하며 가급적 많은 테스트 케이스를 통과하게 변경할 예정임

p.484 l.23 ~ p.485 l.63 테스트 케이스는 통과되어야 함 [G2]

stringToWeekDayCode 메소드의 테스트케이스 작성 [T3]

p.487 l.153 ~ l.154는 통과하지 않음 [G2]

stringToMonthCode 는 아래처럼 변경되어야 함

```java
if((result < 1) || (result > 12)) {
  result = -1;
  for(int i = 0; i < monthNames.length; i++) {
    if(s.equalsIgnoreCase(shortMonthNames[i])) {
      result = i + 1;
      break;
    }
    if(s.equalsIgnoreCase(monthNames[i])) {
      result = i + 1;
      break;
    }
  }
}
```

p.463 l.685 는 변경되어야 함

```java
if(baseDOW >= targetWeekday) {
```

p.491 l.329 의 getTestNearestDayOfWeek 메서드의 테스트 코드는 계속 실패함, 이는 경계 조건 오류 [T5]

p.464 l.719 의 알고리즘은 변경되어야 함

```java
int delta =targetDOW - base.getDayOfWeek();
int positiveDelta = delta + 7;
int adjust = positiveDelta % 7;
if(adjust > 3)
  adjest -= 7;

return SerialDate.addDays(adjust, base);
```

p.493 l.417, p.494 l.429 은 IllegalArgumentException 으로 테스트 통과시킴

이렇게 모든 테스트 케이스를 통과시키고 난 후 SerialDate 코드를 ‘올바로’ 고쳐보자

### 둘째, 고쳐보자

SerialDate 를 처음부터 고쳐봄

p.448 l.1 의 라이선스 정보, 저작권, 작성자, 변경 이력 중에 변경 이력은 삭제함 [C1]

```java
// ASIS
* 변경 내역 (11-Oct-2021부터)
* ----------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키지인
*               com.jrefinery.date로 옮겼다 (DG);
* 05-Nov-2001 : getDescription() 메서드를 추가했으며
...
* 05-Jan-2005 : addYears() 메서드에 있는 버그를 수정했다 (1096282) (DG);
*
*/

// TOBE
삭제
```

p.449 l.61 의 import 문은 `java.text.*;` , `java.util.*;`로 축약함 [J1]

```java
// ASIS
import java.io.Serializable;
import java.text.DateFormatSymbols;
...
import java.util.GregorianCalendar;

// TOBE
import java.io.Serializable;
import java.util.*;
```

p.449 l.67 의 Javadoc 주석, 한 소스코드에 여러 언어를 사용함 [G1]

자바, 영어, Javadoc, HTML 을 사용하기 때문에 모양새를 갖추기 어려움

주석 전부를 `<pre>` 로 감싸는 것이 좋음

p.450 l.86 의 클래스 이름이 SerialDate 인 이유는 Serial Number 을 사용해 클래스를 구현했지 때문임

Serial Number 라는 용어는 제품 식별 번호로 알맞고, 클래스명으로는 차라리 relative offset 이 더 명확함 [N1]

게다가 SerialDate 라는 이름은 구현을 암시하는데 실상은 추상 클래스임 [N2]

차라리 그냥 Date 나 Day 가 나은데 비슷한 이름의 클래스가 많으므로 DayDate 로 명명함

```java

// ASIS
public abstract class SerialDate implements Comparable, Serializable, MonthConstants {
	...

// TOBE
public abstract class DayDate implements Comparable, Serializable {
	...
```

p.481 B-3 은 MonthConstants 클래스는 달을 정의하는 static final 상수 모음임 [J2]

MonthConstant는 enum 으로 정의함

```java
// ASIS
public interface MonthConstants {
	public static final int JANUARY = 1;
	public static final int FEBRUARY = 2;
	...
	public static final int DECEMBER = 12;
}

// TOBE
public abstract class DayDate implements Comparable, Serializable {
	public static enum Month {
		JANUARY(1),
		FABRUARY(2),
		MARCH(3),
		...
		NOVEMBER(11),
		DECOMBER(12);

		Month(int index) {
			this.index = index;
		}

		public static Month make(int monthIndex) {
			for (Month m : Month.values()) {
				if(m.index == monthIndex)
					return m;
			}
			throw new IllegalArgumentException("Invalid month index " + monthIndex);
		}
		public final int index;
	}
}
```

MonthConstants 를 enum 으로 변경하면 int 로 달을 받던 메서드들이 Month 로 받게됨

monthCodeToQuarter 같은 오류 검사 코드가 필요치 않음 [G5]

p.450 l.91 의 serialVersionUID 가 선언되어 있지만 나는 자동 제어에 의한 직렬화가 가 더 안전하게 여겨지기에 삭제함 [G4]

```java
// ASIS
private static final long serialVersionUID = -293716040467...;

// TOBE
삭제
```

p.450 l.93 은 불필요한 주석 [C2]

p450 l.97, l.100 일련번호를 언급함 [C1], 두 변수는 DayDate 클래스가 표현가능한 최초, 최후날짜를 깔끔하게 표현함 [N1]

```java
// ASIS
public static final int SERIAL_LOWER_BOUND = 2;
public static final int SERIAL_UPPER_BOUND = 2958465;

// TOBE
public static final int EARLIEST_DATE_ORDINAL = 2; // 1900/01/01
public static final int LATEST_DATE_ORDINAL = 2958465; // 9999/12/31
```

EARLIEST_DATE_ORDINAL 은 DayDate 보다도 MS EXCEL 의 날짜를 표현하는 방식과 관련이 있음

EARLIEST_DATE_ORDINAL, LATEST_DATE_ORDINAL 을 SpreadsheetDate 클래스로 옮김 [G6]

p.450 l.104, l.107 의 MININUM_YEAR_SUPPORTED, MAXIMUN_YEAR_SUPPORTED 는 구체적인 정보로 DayDate 에 포함될 이유가 없음 [G6]

하지만 p.507 B-6 의 RelativeDayOfWeekRule 클래스가 위의 변수를 사용함

즉, 추상 클래스 사용자가 구현 정보를 알아야 하게 되었음

DayDate 추상클래스를 훼손하지 않으며서 구현 정보를 전달할 방법을 찾아야 함

p.511 l.187 ~ l.205 를 살펴보면 DayDate(SerialDate) 인스턴스는 getPreviousDayOfWeek, getNearestDayOfWeek, getFollowingDayOfWeek 메서드 중 하나가 생성함

p.462 l.638 ~ p.464 l.724 의 세 메서드는 createInstance 를 통해 DayDate 인스턴스를 반환함

그런데 p.466 l.808 의 createInstance 는 SpreadsheetDate 인스턴스도 생성함 [G7]

ABSTRACT FACTORY 패턴을 적용하여 DayDateFactory 을 만들고 DayDate 인스턴스를 생성하고, 구현관련 질문에도 답하게 만듬

이를 통해 createInstance 메서드를 makeDate 로 서술적인 이름으로 바꿈 [N1]

```java
// ASIS
public SerialDate getDate(final int year) {
	...
	SerialDate reuslt = null;
	final SerialDate base = this.subrule.getDate(year);
	if(base != null) {
		switch(this.relative) {
			case(SerialDate.PRECEDING):
				result = SerialDate.getPreviousDayOfWeek(this.dayOfWeek, base);
				break;
			case(SerialDate.NEAREST):
				result = SerialDate.getNearestDayOfWeek(this.dayOfWeek, base);
				break;
			case(SerialDate.FOLLOWING):
				result = SerialDate.getFollowingDayOfWeek(this.dayOfWeek, base);
				break;
			default:
				break;
		}
	}
	return result;
}
...
public static SerialDate getPreviousDayOfWeek(final int targetWeekday, final SerialDate base) {
	...
	final int adjust;
	final int baseDOW = base.getDayOfWeek();
	if(baseDOW > targetWeekday) {
		...
	}
	return SerialDate.addDays(adjust, base);
}
...
public static SeiralDate addDays(final int days, final SerialDate base) {
	final int serialDayNumber = base.toSerial() + days;
	return SerialDate.createInstance(serialDayNumber);
}

// TOBE
public abstract class DayDate implements Comparabl, Serializable {
	...
	public DayDate getPreviousDayOfWeek(Day targetDayOfWeek) {
		int offsetToTarget = targetDayOfWeek.toInt() = getDayOfWeek().toInt();
		if(offsetToTarget >= 0)
			offsetToTarget -= 7;
		return plusDays(offsetToTarget);
	}
	...
	public DayDate plusDays(int days) {
		return DayDateFactory.makeDate(getOrdinalDay() + days);
	}

public abstract class DayDateFactory {
	private static DayDateFactory factory = new SpreadsheetDateFactory();
	public static void setInstance(DayDateFactory factory) {
		DayDateFactory.factory = factory;
	}
	...
	public static DayDate makeDate(int ordinal) {
		return factory._makeDate(oridnal);
	}
}

public class SpreadsheetDateFactory extends DayDateFactory { 
  public DayDate _makeDate(int ordinal) {
    return new SpreadsheetDate(ordinal); 
  }
  ...
}

public class SpreadsheetDate extends DayDate {
	...
	public SpreadsheetDate(int serial) {
		...
		calcDayMonthYear();
	}
}
```

추상메서드로 위임하는 정적메서드는 SINGLETON, DECORATOR, ABSTRACT FACTORY 패턴 조합을 사용함

MINIMUM_YEAR_SUPPORTED, MAXIMUM_YEAR_SUPPORTED 는 SpreadsheetDate 클래스로 옮겨임 [G6]

p.450 l.109 에 나오는 요일 상수는 enum 이어야 마땅함 [J3]

p.451 l.140 LAST_DAY_OF_MONTH 부터 배열이 나오는데 불필요한 주석이 있어 삭제함 [G3]

AGGREGATE_DAYS_TO_END_OF_MONTH, LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_MONTH 는 JCommon 어디서도 사용하지 않음 [G9]

p.505 l.434, p.506 l.473 의 AGGREGATE_DAYS_OF_END_OF_PRECEDING_MONTH 는 SpreadsheetDate 에서만 사용하지만 특정 구현에 의존하지 않음 [G6]

따라서 변수가 사용되는 위치에 가깝게 옮겼음 [G10]

일관성을 유지하려면 AGGREGATE_DAYS_TO_END_OF_PRECEDING_MONTH, LEAP_YEAR_AGGREGATE_DAYS_TO_END_OF_PRECEDING  배열을 만들고 정적메서드로 노출해야 함 [G11]

하지만 아무도 이런 메서드를 사용하지 않으므로 일단 SpreadsheetDate 로 옮김

p.451 l.162 ~ p.452 l.205 의 상수는 enum 으로 변환

```java
// ASIS
public static final int FIRST_WEEK_IN_MONTH = 1;
...
public static final int LAST_WEEK_IN_MONTH = 0;

// TOBE
public enum WeekInMonth {
	FIRST(1), SECOND(2), THIRD(3), FOURTH(4), LAST(0);
	public final int index;

	WeekInMonth(int index) {
		this.index = index;
	}
}
```

p.452 l.177 ~ l.187 의 상수는 범위 끝 날짜를 범위에 포함할지 여부를 지정함

INCLUDE_NONE, INCLUDE_FIRST 등의 상수명은 의도를 분명하게 표현하기 위해 enum 을 만들고 CLOSED, CLOSED_LEFT 등으로 정의함 [N3]

```java
// ASIS
public static final int INCLUDE_NODE = 0;
...
public static final int INCLUDE_BOTH = 3;

// TOBE
public enum WeekdayRange {
	LAST, NEAREST, NEXT
}
```

p.452 l.308 description 변수는 제거 [G9]

p.452 l.213 기본 생성자 제거 [G12]

p.453 l.216 ~ l.238 isValidWeekdayCode 메서드 삭제

p.453 l.242 ~ p.454 l.270 stringToWeekdayCode 의 주석은 삭제함 [C3][G12][C2]

인수와 변수 선언에서 final 키워드를 모두 없앴음 [G12]

p454 l.259 ~ l.263 의 for 안에 if 가 두번 나오는 것 수정 [G5]

```java
// ASIS
public static int stringToWeekdayCode(String s) {
	...
	for(int i = 0; i < weekDayNames.length; i++) {
		if(s.equals(shortWeekdayNames[i])) {
			result = i;
			break;
		}
		if(s.equls(weekDayNames[i])) {
			result = i;
			break;
		}
	return result;
}

// TOBE
?
```

stringToWeekdayCode 메서드는 Day의 구문분석 메서드임

Day 는 DayDate 에 의존하지 않으므로 Day 를 분리하여 독자적인 파일로 만듬 [G13]

p.454 l.272 ~ l.286 의 weekdayCodeToString 메서드도 Day 로 옮김

p.454 l.288 ~ p.455 ~ l.316 의 getMonths 2개는 합치고 이름을 서술적으로 변경함 [G9][N1]

```java
// ASIS
public static String[] getMonths() {
	return getMonths(false);
}
public static String[] getMonth(final boolean shortened) {
	if(shortened) {
		return DATE_FORMAT_SYMBOLS.getShortMonths();
	} else {
		return DATE_FORMAT_SYMBOLS.getMonths();
	}
}

// TOBE
public static String[] getMonthNames() {
 return dateFormatSymbols.getMonths();
}
```

p.455 l.326 ~ l.346 의 isValidMonth 메서드 삭제 [G9]

p.456 l.356 ~ l.375 의 monthCodeToQuarter 메서드는 기능 욕심 [G14]

Month 를 DayDate 에서 분리 [G11][G13]

p.456 l.377 ~ p.457 l.426 의 monthCodeToString, stringToMonthCode 메서드 2개는 이름을 변경, 단순화, Month enum 으로 이동 [G15][N1][N3][C3][G14]

```java
// ASIS
public static String monthCodeToString(final int month, final boolean shorened) {
	...
	final String[] months;
	if(shortened) {
		months = DATE_FORMAT_SYMBOLS.getShortMonths();
	} else {
		months = DATE_FORMAT_SYMBOLS.getMonths();
	}
	return months[month - 1];
}

public static int stringToMonthCode(String s) {
	final String[] shortMonthNames = DATE_FORMAT_SYMBOLS.getShortMonths();
	final String[] monthNames = DATE_FORMAT_SYMBOLS.getMonths();
	
	int result = -1;
	s = s.trim();
	try {
		result = Integer.parseInt(s);
	} catch(NumberFormatException e) {
	}
	if((result < 1)}}(result > 12)) {
		for(int i = 0; i < monthNames.length; i++) {
			if(s.equals(shortMonthNames[i])) {
				result = i + 1;
				break;
			}
			if(s.equals(monthNames[i])) {
				result = i + 1;
				break;
			}
		}
	}
}

// TOBE
public String toString() {
	return dateFormatSymbols.getMonths()[index - 1;
}
public static Month parse(String s) {
	s = s.trim();
	for(Month m : Month.values())
		if(m.matches(s))
			return m;
	try {
		return fromInt(Integer.parseInt(s));
	} catch (Number?FormatException e) {
	}
	throw new IllegalArgumentException("Invalid month " + s);
}
```

p.457 l.428 ~ p.459 l.472 의 stringToMonthCode 의 이름을 변경, 단순화, Month enum 으로 이동 [N1][N3][C3][G14]

p.459 l.495 ~ l.517 의 isLeapYear 메서드는 서술적인 표현으로 변경 [G16]

p.460 l.519 ~ l.536 의 leapYearCount메서드는 이동 [G6]

p.460 l.538 ~ l.560 의 lastDayOfMonth 메서드는 Month enum 으로 이동, 서술적인 표현으로 변경 [G17][G16]

> p.360 이제부터 흥미로워진다...?

p.461 l.576 의 addDay 메서드는 이름 변경, 인스턴스 메서드로 변경 [G18][N1]

p.461 l.578 ~ p.462 l.602 의 addMonths 메서드는 인스턴스 메서드로 변경, 가독성 개선, 이름 변경[G18][G19][N1]
    
```java
// ASIS
public static SerialDate addMonths(final int months, final SerialDate base) {
  final int yy = (12 * base.getYYYY() + base.getMonth() + months - 1) / 12;
  final int mm = (12 * base.getYYYY() + base.getMonth() + months - 1) % 12 + 1;
  final int dd = Math.min(base.getDayOfMonth(), SerialDate.lastDayOfMonth(mm, yy));
  retunr SerialDate.createInstance(dd, mm, yy);
}

// TOBE
public DayDate addMonths(int months) {
  int thisMonthAsOrdianl = 12 * getYear() + getMonth().index - 1;
  int resultMonthAsOrdinal = thisMonthAsOrdinal + months;
  int resultYear = resultMonthAsOrdinal / 12;
  Month result Month = Month.make(resultMonthAsOrdinal % 12 + 1);
  
  int lastDayOfResultMonth = lastDayOfMonth(resultMonth, resultYear);
  int resultDay = Math.min(GetDayOfMonth(), lastDayOfResultMonth);
  return DayDateFactory.makeDate(resultDay, resultMonth, resultYear);
}
```
    

p.462 l.604 ~ l.626 의 addYears 메서드도 마찬가지임

date.addDays(5) 라는 표현은 date 객체에 5일이 더해진다고 보일까? 새 DayDate 인스턴스를 반환하는 것으로 보일까봐 이름을 변경 [G20][N4]
    
```java
// ASIS
DayDate date = DayFactory.makeDate(5, Month.DECEMBER, 1952);
date.addDays(5); // 날짜를 일주일 뒤로 미룬다.

// TOBE
DayDate date = oldDate.plusDays(5);
```
    

p.462 l.628 ~ l.660 의 getPreviousDayOfWeek 메서드는 단순화, 임시 변수 설명을 수정, 인스턴스 메서드로 변경[G21][G19][G18]

p.471 l.997 ~ l.1008 의 중복 메서드 삭제[G15]

p.463 l.662 ~ p.464 l.693 의 getFollowingDayOfWeek 메서드도 동일하게 변경

p.464 l.695 ~ l.726 의 getNearestDayOfWeek 메서드는 패턴을 맞추고 임시 변수 설명을 이용해 알고리즘을 명확하게 변경[G11][G19]

p.464 l.728 ~ p.465 l.740 의 getEndOfCurrentMonth 메서드는 인스턴스 메서드로 변경
    
```java
// ASIS
public SerialDate getEndOfCurrentMonth(final SerialDate base) {
  final int last = SerialDate.lastDayMonth(base.getMonth(), base.getYYYY());
  return SerialDate.createInstance(last, base.getMonth(), base.getYYYY());
}

// TOBE
public DayDate getEndOfMonth() {
  Month month = getMonth();
  int year = getYear();
  int lastDay = lastDayOfMonth(month, year);
  return DayDateFactory.makeDate(lastDay, month, year);
}
```

p.465 l.742 ~ l.761 의 weekInMonthToString 메서드는 WeekInMonth enum 으로 옮긴 후 toString 으로 이름 변경, 인스턴스 메서드로 변경, 테스트를 통과 시킴

이후 메서드를 삭제하면 B-4 p.493 l.411 ~ l.415 의 테스트가 실패함. 

테스트 케이스를 FIRST 등 enum 값을 사용하도록 변경하여 테스트를 통과시킴

하지만 위의 메서드는 테스트 케이스에서만 호출되므로 메서드, 테스트 케이스를 삭제함

> 드디어 추상 클래스 DayDate 의 추상메서드를 살펴볼 차례다.

p.467 l.829 ~ l.836 의 toSerial 메서드는 getOrdinalDay 로 변경

p.467 l.844 ~ l.846 의 toDate 메서드는 DayDate 메서드로 끌어올림?

getDayOfWeek 메서드가 SpreadsheetDate 인스턴스에 의존하지 않으므로, DayDate 로 옮겨야 할까?[G6]

B-5 p.500 l.247 을 보면 알고리즘은 0번째 날짜의 요일에 의존하기 때문에 getDayOfWeek 메서드를 DayDate 로 옮기지 못함

구현이 논리적으로 의존한다면 물리적으로도 의존해야함[G22]

DayDate 에 getDayOfWeekForOrdinalZero 추상 메서드를 구현하고, SpreadsheetDate 에 Day.SATURDAY 를 반환하도록 구현함

그리고 getDayOfWeek 메서드를 DayDate 로 이동하여 getOrdinalDay, getDayOfWeekForOrdinalZero 메서드를 호출하도록 변경

p.468 l.895 ~ l.899 의 중복 주석 제거

p.469 l.902 ~ l.913 의 compare 메서드는 DayDate 로 이동, 이름 변경[G6][N1]

p.469 l.982 ~ p.471 l.995 의 isInRange 메서드도 DayDate 로 이동, if 문은 DateInterval enum 으로 옮겨서 삭제

마지막으로 지금까지 한 작업을 정리함

1. 주석을 고치고 개선[C2]
2. enum 을 독자적인 소스 파일로 옮김[G12]
3. 정적 변수(dateFormatSymbols) 와 정적 메서드(getMonthNames, isLeapYear, lastDayOfMonth) 를 DateUtil 이라는 클래스로 옮김[G6]
4. 일부 추상 메서드를 DayDate 클래스로 이동 [G24]
5. Month.make 를 Month.fromInt 로 변경[N1], 모든 enum 에 toInt() 접근자를 생성, index 필드를 private 로 정의
6. plusYears 와 plusMonths 의 중복은 correctLastDayOfMonth 라는 새 메서드로 제거[G5]
7. 팔방미인으로 사용하던 숫자 1을 삭제하고, Month.JANUARY.toInt()  혹은 Day.SUNDAY.toInt() 로 변경[G25]

### 결론

보이스카우트 규칙을 잘 지켰음

좀 더 깨끗한 코드를 체크인하였음

테스트 커버리지가 증가하였음

버그가 수정되고, 코드 크기가 줄고, 코드가 명확해졌음

다음 사람은 우리보다 코드를 좀 더 쉽게 이해하게 되었고, 따라서 좀 더 쉽게 개선하리라

---

SerialDate 를 리팩터링한 본 장을 읽고난 후 느낀 점

1. 데이터를 구조화
    1. DAY, MONTH 등 enum 클래스를 잘 활용하여 유사한 데이터들끼리 잘 뭉쳐주었음
2. 패턴 활용
    1. 다른 책에서도 비슷한 케이스를 보았는데 리팩터링에서는 특히 (추상)팩토리 패턴은 한번쯤은 꼭 나옴
    2. 이는 리팩터링이 미래의 변경이 용이하게 해주는 구조를 채택하는 경우가 많으며, 이를 추상 팩토리 패턴으로 가능하기 때문임
3. 이름바꾸기
    1. 변수, 메서드 이름을 이해하기 쉽게 바꾸는 것이 리팩터링에 상당부분을 차지함
4. 중복제거
    1. 중복은 버그의 근원임

## 참고
- 