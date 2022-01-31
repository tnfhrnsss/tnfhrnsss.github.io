---
layout: post
title: 전락 열거 타입 패턴
date: 2021-09-09 13:07:45
last_modified_at : 2021-09-09 13:07:45
parent: Patterns
has_children: false
nav_order: 2
nav_exclude: true
---

# 전락 열거 타입 패턴

- 열거형 클래스에 새로운 값을 열거타입에 추가하려면 그 값을 처리하는 case문을 쌍으로 넣어줘야하는 문제가 있다.
- 새로운 상수를 추가할 때마다 '전략'을 선택하도록 하면 switch문이나 상수별 메서드 구현이 필요없게 된다.

```java
enum PayrollDay {
	MONDAY(WEEKDAY), SUNDAY(WEEKEND);

	private final PayType payType;

	PayrollDay(PayType payType) { this.payType = payType; }

	int pay(int minutesWorked, int payRate) {
		return payType.pay(minutesWorked, payRate);
	}

	enum PayType {
		WEEKDAY {
			int overtimePay(int minsWorked, int payRate) {
				return minsWorked <= MIN_PER_SHIFT ? 0 : (minsWorked - MINS_PER_SHIFT) * payRate / 2;
			}
		},
		WEEKEND {

		};

		abstract int overtimePay(int mins, int payRate);
		private static final int MINS_PER_SHIFT = 8 * 60;

		int pay(int minsWorked, int payRate) {
			int basePay = minsWorked * payRate;
			return basePay + overtimePay(minsWorked, payRate);
		}
	}
}
```