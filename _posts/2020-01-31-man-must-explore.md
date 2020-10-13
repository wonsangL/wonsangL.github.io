---
layout: post
title: "[Spring Boot] Custom Constraint / Validation 파헤치기"
subtitle: "Problems look mighty small from 150 miles up"
date: 2020-01-31 10:45:13 -0400
background: '/img/posts/06.jpg'
---

이번 글에서는 Spring Boot에서

validation이 동작하는 원리를 간단하게 살펴보고

직접 validator를 정의하는 방법을 정리해보겠습니다.

 

참고로, validation에 대한 설명을 따로 하지는 않기 때문에

validation이 무엇인지 궁금하신 분들은

다음 글을 참고해 주시면 될 것 같습니다.

( [Validation 어디까지 해봤니? : TOAST Meetup](https://meetup.toast.com/posts/223) )

 

또한, 예제들은 다음과 같은 환경에서 작성되었습니다.

- Spring Boot 2.3.3
- hibernate validator 6.1.5.Final 
('spring-boot-starter-validation' dependency에 포함되어 있습니다.)
- JUnit 5
우선 validation을 수행하기 위해

간단한 핸들러 메소드와 DTO(User class)를 정의해보겠습니다.

 
```java
//SampleController.java
@PostMapping("/signin")
public boolean signin(@Valid @RequestBody User user){
    return service.signin(user);
}

//SampleService.java
public boolean signin(User user){
    return !user.getName().isEmpty();
}
 

@Data
public class User {
    @NotBlank
    @NoSpecialCharacter
    private String name;

    private int age;
}
```

validator에 초점을 맞추기 위해

최대한 간단하게 코드를 작성해 보았습니다.

 

`User` class를 보시면 `name`이라는 필드 값에

`@NotBlank`, `@NoSpecialCharacter`라는 어노테이션이 붙어있는 것을 보실 수 있습니다.

 

여기서 `@NotBlank`는 미리 정의되어 있는 어노테이션,

`@NoSpecialCharacter`는 이번 글에서 우리가 직접 정의할 어노테이션입니다.

 

그럼 바로 `@NoSpecialCharacter`가 어떻게 정의되어 있는지 보겠습니다.

 
```java
@Constraint(validatedBy = NoSpecialCharacterValidator.class)
@Target(FIELD)
@Retention(RUNTIME)
public @interface NoSpecialCharacter {
    String message() default "can't contain special character";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```
 

이번 글에서는 validation에 초점을 맞추기 위해

`@Target`, `@Retention` 어노테이션은 따로 설명하지 않겠습니다.

 

그럼 가장 먼저 눈에 띄는 `@Constraint` 어노테이션을 살펴보겠습니다.

 

`@Constraint`의 `validatedBy` 값으로 우리가 아래에서 정의할

validator를 전달해주면 Spring Boot는 우리가 API를 호출할 때

전달한 값을 가져오면서 validation을 수행합니다.

 
```java
//ConstraintHelper::getDefaultValidatorDescriptors
Class<? extends ConstraintValidator<A, ?>>[] validatedBy = (Class<? extends ConstraintValidator<A, ?>>[]) annotationType
	.getAnnotation( Constraint.class )
	.validatedBy();

//ConstraintTree::validateSingleConstraint
V validatedValue = (V) valueContext.getCurrentValidatedValue();
isValid = validator.isValid( validatedValue, constraintValidatorContext );
```
 

그럼 여기서 전달되는 validator는 어떻게 정의 되어 있는지 살펴보겠습니다.

 
```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.util.regex.Pattern;

public class NoSpecialCharacterValidator implements ConstraintValidator<NoSpecialCharacter, String> {
    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return false;
        }

        Pattern pattern = Pattern.compile("[^a-zA-Z0-9\\s]");
        return !pattern.matcher(value).find();
    }
}
```
 

생각보다 간단하게, `ConstraintValidator`라는 인터페이스를 구현하고

`isValid`라는 메소드 하나를 정의하고 있습니다.

 

이 `isValid`라는 메소드에 우리가 정의하고자하는 validation logic이 정의됩니다.

예제의 메소드에서는 value가 null이거나

value에 알파벳과 숫자가 아닌 문자가 포함되어 있으면 false를 반환합니다.

 

그리고 위의 결과가 false일 경우, `MethodArgumentNotValidException`이 발생합니다.

 

다시 `@NoSpecialCharacter`로 돌아와서 나머지 내용들을 살펴보겠습니다.

 
```java
@Constraint(validatedBy = NoSpecialCharacterValidator.class)
@Target(FIELD)
@Retention(RUNTIME)
public @interface NoSpecialCharacter {
    String message() default "can't contain special character";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```
 

@NoSpecialCharacter 내부에는 message, groups, payload가 정의되어 있습니다.

해당 내용들은 하나라도 빠져서는 안되며 ConstraintHelper가 이를 확인합니다.

 
```java
//ConstraintHelper::isConstraintAnnotation
return externalConstraints.computeIfAbsent( annotationType, a -> {
	assertMessageParameterExists( a );
	assertGroupsParameterExists( a );
	assertPayloadParameterExists( a );
	assertValidationAppliesToParameterSetUpCorrectly( a );
	assertNoParameterStartsWithValid( a );

	return Boolean.TRUE;
} );
```

코드를 보시면 아시겠지만 이 외에도 @NoSpecialCharacter 내부에는

"valid"로 시작되는 메소드가 있어도 예외가 발생합니다.

 

Hibernate의 공식 문서에 따르면 각각의 값이 의미하는 바는 다음과 같습니다.

message: validation이 실패할 경우 반환되는 default 메세지

group: 특정 validation을 group을 지정하는 값( Validation Grouping )

payload: 사용자가 추가 정보를 위해 전달할 수 있는 값으로 주로 심각도를 나타낼 때 사용됩니다.
각 값들에 대한 내용은 기회가 된다면 다른 글에서 조금 더 자세하게 다뤄보겠습니다.

 

마지막으로 우리가 정의한 validator가 잘 동작하는지 테스트 해보겠습니다.

 
```java
class NoSpecialCharacterValidatorTest {
    private static ValidatorFactory validatorFactory;
    private static Validator validator;

    @BeforeAll
    static void init() {
        validatorFactory = Validation.buildDefaultValidatorFactory();
        validator = validatorFactory.getValidator();
    }

    @Test
    void isValid_validUser_Test() {
        User user = new User();
        user.setName("name");

        Set<ConstraintViolation<User>> violations = validator.validate(user);
        assertTrue(violations.isEmpty());
    }

    @Test
    void isValid_invalidUser_Test() {
        User user = new User();
        user.setName("name!");

        Set<ConstraintViolation<User>> violations = validator.validate(user);

        Iterator<ConstraintViolation<User>> iterator = violations.iterator();
        ConstraintViolation<User> violation = iterator.next();

        assertEquals("name", violation.getPropertyPath().toString());
    }
}
```

그럼 여기까지 Custom Constraint 정의하는 방법이었습니다.

 

감사합니다.

Reference
- https://meetup.toast.com/posts/223
- https://docs.jboss.org/hibernate/validator/4.1/reference/en-US/html/validator-customconstraints.html#validator-customconstraints-constraintannotation

