# 미션 1. 회원 도메인 만들기

> **필요한 진도**: 실전 자바 기본편 섹션 2 ~ 섹션 8 `56. static 메서드1`
> **다음 미션**: 미션 2. 상품 도메인

## 목표

쇼핑몰의 첫 기능인 **회원 가입과 조회**를 만듭니다.
지금까지 배운 클래스, 생성자, 패키지, 접근 제어자, static을 "실제 백엔드 코드에서는 어디에 쓰는지" 직접 써 보는 미션이에요.

## 이번 미션에서 쓰는 강의 개념

| 개념 | 강의 | 어디에 쓰나 |
|---|---|---|
| 클래스와 객체 | 섹션 2 | `Member`: 회원 한 명의 데이터를 묶는 클래스 |
| 참조형, `null` | 섹션 3 | 회원을 못 찾으면 `null` 반환, 내부 배열 대신 복사본 반환 |
| 객체 지향 | 섹션 4 | 저장은 저장소가, 가입 규칙은 서비스가 맡도록 책임 나누기 |
| 생성자, `this`, `this()` | 섹션 5 | `Member` 생성자, `MemberRepository` 생성자 오버로딩 |
| 패키지, `import` | 섹션 6 | `domain` / `repository` / `service` / `util` 패키지 분리 |
| 접근 제어자, 캡슐화 | 섹션 7 | 필드는 `private`, 밖에서 쓸 메서드만 `public` |
| static 변수 | 섹션 8 | 회원 번호를 1, 2, 3… 으로 매기는 `sequence` |
| static 메서드 | 섹션 8 | 빈 문자열 검사 유틸리티 `StringUtil.isBlank()` |

## 백엔드에서 이 구조의 의미

실무 백엔드는 보통 역할별로 계층을 나눕니다. 이번 미션에서 그 뼈대를 처음 만들어요.

```
MemberCheck (확인용 main)    ← 지금은 직접 객체를 만들고 호출. 나중에 JUnit 테스트가 됨
   │
   ▼
MemberService   (서비스)     ← 비즈니스 규칙: "이메일은 중복될 수 없다"      → 스프링의 @Service
   │
   ▼
MemberRepository (저장소)    ← 데이터 저장·조회만 담당. 지금은 배열            → 스프링의 @Repository + DB
   │
   ▼
Member          (도메인)     ← 회원 데이터                                   → JPA의 @Entity
```

포인트는 **서비스는 "어떻게 저장되는지" 모르고, 저장소는 "가입 규칙"을 모른다**는 것입니다.
그래서 나중에 배열을 DB로 바꿔도 서비스 코드는 거의 그대로 둘 수 있어요.

## 만들 파일

```
src/shop/
├── ShopMain.java                    (이미 있음, 이번 미션에서는 건드리지 않음)
├── domain/Member.java               ← 새로 만들기
├── repository/MemberRepository.java ← 새로 만들기
├── service/MemberService.java       ← 새로 만들기
├── util/StringUtil.java             ← 새로 만들기
└── check/MemberCheck.java           ← 마지막에 아래 코드를 붙여넣기
```

## 요구사항

### 1. `shop.domain.Member`

- 필드: 회원 번호 `id` (`long`), 이름 `name`, 이메일 `email`. **모두 `private`**
- 생성자 `Member(String name, String email)`: 이름과 이메일만 받는다. `id`는 저장소가 저장할 때 정해 준다.
- 메서드: `getId()`, `getName()`, `getEmail()`, `setId(long id)`

### 2. `shop.repository.MemberRepository`

회원을 **배열**에 저장하고 꺼내는 클래스입니다.

- 필드
  - 회원 번호를 만드는 `sequence`: **static 변수**, 0부터 시작
  - 회원을 담을 `Member[]` 배열, 지금까지 저장한 회원 수 `size`
- 생성자 (오버로딩)
  - `MemberRepository(int capacity)`: 배열을 `capacity` 크기로 만든다.
  - `MemberRepository()`: 기본 크기 10. **`this(...)`를 써서** 위 생성자를 호출한다.
- 메서드

| 메서드 | 동작 |
|---|---|
| `Member save(Member member)` | 배열이 가득 찼으면 `null` 반환. 아니면 `sequence`를 1 올려 회원에 `id`를 넣고, 배열에 저장한 뒤 그 회원을 반환 |
| `Member findById(long id)` | 번호가 같은 회원을 반환. 없으면 `null` |
| `Member findByEmail(String email)` | 이메일이 같은 회원을 반환. 없으면 `null` |
| `Member[] findAll()` | 저장된 회원만 담은 **새 배열**을 반환 (길이 = 회원 수) |
| `int count()` | 저장된 회원 수 |

### 3. `shop.util.StringUtil`

- **static 메서드** `boolean isBlank(String value)`: `value`가 `null`이거나 빈 문자열 `""`이면 `true`

### 4. `shop.service.MemberService`

- 필드: `MemberRepository memberRepository` (`private`)
- 생성자 `MemberService(MemberRepository memberRepository)`: 저장소를 **직접 `new` 하지 말고 밖에서 받는다.**
  (왜 이렇게 하는지는 다형성을 배울 때 확실히 와닿을 거예요. 스프링의 "생성자 주입"이 바로 이 모양입니다.)
- 메서드

| 메서드 | 동작 |
|---|---|
| `Member join(String name, String email)` | 아래 규칙을 순서대로 검사. 실패하면 이유를 출력하고 `null` 반환, 성공하면 저장된 회원 반환 |
| `Member findMember(long memberId)` | 회원 한 명 조회. 없으면 `null` |
| `Member[] findMembers()` | 전체 회원 조회 |

`join` 검사 규칙과 실패 메시지:

1. 이름이나 이메일이 비어 있으면 (`StringUtil.isBlank` 사용) → `[가입 실패] 이름과 이메일은 필수입니다.`
2. 같은 이메일로 가입한 회원이 있으면 → `[가입 실패] 이미 가입된 이메일입니다: java@shop.com`
3. 저장소가 가득 차서 저장에 실패하면 → `[가입 실패] 저장 공간이 부족합니다.`

> 지금은 서비스가 직접 메시지를 출력하지만, 중급 1편에서 **예외**를 배우면 서비스는 예외를 던지고
> 출력은 화면 계층이 맡도록 바꿀 거예요. 실패를 `null`로 알리는 방식의 불편함을 미리 느껴 두세요.

<details>
<summary>힌트: 패키지가 다른 클래스를 쓰려면?</summary>

`MemberService`는 `shop.service` 패키지에 있고 `Member`는 `shop.domain`에 있어요.
`import shop.domain.Member;`가 필요하고, 다른 패키지에서 쓰는 생성자와 메서드는 `public`이어야 합니다.
</details>

<details>
<summary>힌트: <code>findAll()</code>에서 왜 새 배열을 만들까?</summary>

`store`를 그대로 반환하면 두 가지 문제가 생겨요.
1. 길이가 10이라서 뒤쪽에 `null`이 섞여 나온다.
2. 반환받은 쪽이 배열을 고치면 **저장소 안의 배열도 같이 바뀐다.** 배열은 참조형이니까요. (섹션 3)

`new Member[size]`로 새 배열을 만들고 반복문으로 옮겨 담으세요.
</details>

## 완성 확인

`src/shop/check/MemberCheck.java`를 만들고 아래 코드를 그대로 붙여넣은 뒤 실행하세요.

```java
package shop.check;

import shop.domain.Member;
import shop.repository.MemberRepository;
import shop.service.MemberService;

public class MemberCheck {

    public static void main(String[] args) {
        MemberRepository memberRepository = new MemberRepository();
        MemberService memberService = new MemberService(memberRepository);

        System.out.println("== 1. 회원 가입 ==");
        Member memberA = memberService.join("김자바", "java@shop.com");
        Member memberB = memberService.join("이스프링", "spring@shop.com");
        System.out.println("memberA id=" + memberA.getId() + ", name=" + memberA.getName());
        System.out.println("memberB id=" + memberB.getId() + ", name=" + memberB.getName());

        System.out.println("== 2. 중복 이메일 가입 ==");
        Member duplicate = memberService.join("박중복", "java@shop.com");
        System.out.println("duplicate = " + duplicate);

        System.out.println("== 3. 잘못된 입력 ==");
        System.out.println("이름 빈칸 = " + memberService.join("", "blank@shop.com"));
        System.out.println("이메일 null = " + memberService.join("최널", null));

        System.out.println("== 4. 회원 조회 ==");
        Member findMember = memberService.findMember(memberA.getId());
        System.out.println("findMember name=" + findMember.getName());
        System.out.println("memberA와 같은 객체인가? " + (findMember == memberA));
        System.out.println("없는 회원 = " + memberService.findMember(999));

        System.out.println("== 5. 전체 조회 ==");
        Member[] members = memberService.findMembers();
        System.out.println("회원 수 = " + members.length);
        for (Member member : members) {
            System.out.println("id=" + member.getId() + ", name=" + member.getName() + ", email=" + member.getEmail());
        }

        System.out.println("== 6. 내부 배열 보호 ==");
        members[0] = null;
        System.out.println("밖에서 배열을 바꾼 뒤 첫 회원 = " + memberService.findMembers()[0].getName());

        System.out.println("== 7. 저장 공간이 가득 찼을 때 ==");
        MemberService smallService = new MemberService(new MemberRepository(2));
        Member first = smallService.join("회원1", "m1@shop.com");
        smallService.join("회원2", "m2@shop.com");
        System.out.println("세 번째 가입 = " + smallService.join("회원3", "m3@shop.com"));
        System.out.println("다른 저장소의 첫 회원 id = " + first.getId());
    }
}
```

터미널에서는 저장소 루트에서 이렇게 실행해요.

```bash
java src/shop/check/MemberCheck.java
```

**예상 출력**

```
== 1. 회원 가입 ==
memberA id=1, name=김자바
memberB id=2, name=이스프링
== 2. 중복 이메일 가입 ==
[가입 실패] 이미 가입된 이메일입니다: java@shop.com
duplicate = null
== 3. 잘못된 입력 ==
[가입 실패] 이름과 이메일은 필수입니다.
이름 빈칸 = null
[가입 실패] 이름과 이메일은 필수입니다.
이메일 null = null
== 4. 회원 조회 ==
findMember name=김자바
memberA와 같은 객체인가? true
없는 회원 = null
== 5. 전체 조회 ==
회원 수 = 2
id=1, name=김자바, email=java@shop.com
id=2, name=이스프링, email=spring@shop.com
== 6. 내부 배열 보호 ==
밖에서 배열을 바꾼 뒤 첫 회원 = 김자바
== 7. 저장 공간이 가득 찼을 때 ==
[가입 실패] 저장 공간이 부족합니다.
세 번째 가입 = null
다른 저장소의 첫 회원 id = 3
```

출력이 다르게 나오면, 어느 번호에서 달라졌는지가 곧 힌트예요. 예를 들어 6번에서 `NullPointerException`이 난다면 아래 질문 2를 보세요.

## 생각해 볼 질문

리뷰할 때 같이 이야기할 거예요. 답을 짧게라도 생각해 오세요.

1. **4번 결과가 `true`인 이유는?** 저장소에 넣은 회원과 꺼낸 회원은 같은 객체일까요, 복사본일까요? (섹션 3 참조형)
2. **`findAll()`이 내부 배열 `store`를 그대로 반환했다면** 6번에서 무슨 일이 생길까요?
3. **7번에서 다른 저장소인데 왜 번호가 3부터 시작할까요?** `sequence`가 static이 아니라면 어떻게 될까요? 실제 쇼핑몰이라면 어느 쪽이 맞을까요?
4. `StringUtil.isBlank`에서 `value.isEmpty() || value == null` 순서로 쓰면 3번 테스트에서 어떻게 될까요?
5. `setId()`가 `public`이라 아무나 회원 번호를 바꿀 수 있어요. 괜찮을까요? 막으려면 어떤 방법이 있을까요?
6. `MemberService`가 생성자에서 저장소를 직접 `new MemberRepository()` 하지 않고 밖에서 받으면 무엇이 좋을까요? 7번 테스트를 떠올려 보세요.

## 도전 과제 (선택)

- `MemberService`에 `Member findMemberByEmail(String email)` 추가
- `Member`에 이름을 바꾸는 `changeName(String newName)` 추가. 빈 이름으로는 바꿀 수 없게 막기 (캡슐화)

## 다 했으면

Claude에게 **"미션 1 리뷰해줘"** 라고 요청하세요. 리뷰를 반영한 뒤 커밋하고 미션 2로 넘어갑니다.
