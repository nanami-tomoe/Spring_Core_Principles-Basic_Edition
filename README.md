# 객체 지향 원리 적용
## 새로운 할인 정책 적용과 문제점
```java
package hello.core.member;

public class MemberServiceImpl implements MemberService {
		
		// MemoryMemberRepository는 구현체를 의존! 인터페이스를 의존하도록 설계해야함.
    private final MemberRepository memberRepository = new MemoryMemberRepository();
    @Override
    public void join(Member member) {
        memberRepository.save(member);
    }

    @Override
    public Member findMember(Long memberId) {
        return memberRepository.findById(memberId);
    }
}
```

- 구현체가 아니라 인터페이스(역할)을 의존하게 수정해야한다. → **DIP 위반**
- `MemoryMemberRepository`를 다른 구현체로 변경하는 순간 `MemberServiceImpl`의 소스 코드도 함께 변경해야 한다. -> **OCP 위반**
## 관심사의 분리
- `AppConfig`를 통해서 관심사를 분리하자.
- `MemberServiceImpl`과 `OrderServiceImpl`은 기능을 실행하는 책임만 지면 된다. (회원 저장소 역할, 할인 정책 역할 수행)
## `AppConfig` 리팩터링
- 따라서 `Appconfig`를 통해서 회원 저장소와 할인 정책 방법을 결정하자.
	- 중복을 제거하고, 역할에 따른 구현이 보이도록 리팩터링 하자.
```java
package hello.core;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.member.MemberRepository;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import hello.core.member.MemoryMemberRepository;
import hello.core.order.OrderService;
import hello.core.order.OrderServiceImpl;

public class AppConfig {

    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }

    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public DiscountPolicy discountPolicy() {
        return new FixDiscountPolicy();
    }
}

```
- `AppConfig`를 보면 역할과 구현이 한눈에 들어온다.
- 구현체를 변경할 때 한 부분만 변경하면 된다.
- 애플리케이션이 크게 **사용 영역**과, 객체를 **구성(configuration)하는 영역**으로 분리
## IoC, DI, 그리고 컨테이너
### 제어의 역전 IoC(Inversion of Control)
- 클라이언트 구현 객체가 프로그램의 제어 흐름 스스로 조종 x
- OrderServiceImpl은 필요한 인터페이스들을 호출한다. -> 어떤 구현 객체들이 실행될지는 모름
- OrderServiceImpl은 AppConfig가 생성 -> 즉, OrderServiceImpl가 생성될지 말지도 AppConfig가 결정
- 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 **제어의 역전(IoC)** 이라 한다.
- **프레임워크**: 내가 작성한 코드를 제어하고, 대신 실행 (JUnit)
- **라이브러리**: 내가 작성한 코드가 직접 제어의 흐름을 담당
### 의존관계 주입 DI(Dependency Injection)
- **정적인 클래스 의존관계**: import 코드만 보고 판단
- **동적인 객체 인스턴스 의존 관계**: 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계
### Ioc 컨테이너, DI 컨테이너
- `AppConfig` 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것을 Ioc 컨테이너 또는 **DI 컨테이너**라 한다.
- 최근 의존관계 주입에 초점을 맞추어, 주로 DI 컨테이너라 한다
- 또는 어샘블러, 오브젝트 팩토리 등으로 불리기도 한다. (토비의 스프링에서는 팩토리라 부름)
### 스프링으로 전환하기
- `ApplicationContext` = 스프링 컨테이너
- `AppConfig`가 객체 생성하고 DI를 함 -> 이제부터는 **스프링 컨테이너**를 통해서 사용한다.
- 스프링 컨테이너는 `@Configuration`이 붙은 `AppConfig`를 **설정(구성) 정보**로 사용한다.
 - `@Bean`이라 적힌 메서드를 모두 호출해서 반환된 객체를 컨테이너에 등록
   - 이렇게 등록된 객체 = **스프링 빈**
   - 스프링 빈은 `@Bean`이 붙은 메서드의 명을 이름으로 사용 (`memberService`)
- 이전에는 개발자가 필요한 객체를 직접 조회 -> 이제부터는 스프링 컨테이너를 통해 스프링 빈 찾기
  - `applicationContext.getBean()`
---
