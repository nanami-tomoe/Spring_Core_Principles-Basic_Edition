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
