### 회원 도메인 개발

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

- 구현체가 아니라 인터페이스(역할)을 의존하게 수정해야한다. → DIP 위반
