### 객체 지향 원리 적용

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
- `AppConfig`를 통해서 관심사를 분리하자.
- `MemberServiceImpl`과 `OrderServiceImpl`은 기능을 실행하는 책임만 지면 된다. (회원 저장소 역할, 할인 정책 역할 수행)
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
- `AppConfig`를 보면 역할과 구현이 한눈에 들어온다.
- 구현체를 변경할 때 한 부분만 변경하면 된다.
