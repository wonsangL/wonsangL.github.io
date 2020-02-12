---
layout: post
title: Spring::@PathVariable과 @RequestParam
---
<span style="color:gray">
[생각정리]에 있는 글들은 학습을 하면서

생긴 궁금증들에 대해 저 나름대로 고민하면서

끄적인 지극히 주관적인 글들입니다.

때문에 잘못된 정보가 있을 수 있으며

이에 대한 피드백은 언제든지 환영입니다!!
</span>
 
<span style="color:black">
int 타입의 ID를 가진 User의 정보를 가져오는 메소드가 있다고 가정해보겠습니다.

<br>
<br>

```java
@GetMapping("/user")
public String getUser(@RequestParam int id){
    return service.getUserById(id);
}
```

<br>그러던 중 갑자기 의문이 들었습니다.

 

<br>ID값은 `@PathVariable`로 전달하는게 적절할까요?

아니면 `@RequestParam`으로 전달하는게 적절할까요?

 

<br>언제 `@PathVariable`를 써야하고, 언제 `@RequestParam`을 써야할까요?

 

<br>이는 우리가 전달하려는 데이터의 성질에 따라 다르다고 생각합니다.

 

<br>전달하고 싶은 데이터가 계층, path의 성질을 가지고 있다면

해당 데이터는 `@PathVariable`로 전달되어야 한다고 생각합니다.

 

<br>반면, `@RequestParam`으로 전달하는 데이터는

검색에 적절한 데이터들이 전달되어야 한다고 생각합니다.

 

<br>위의 예제에서 ID는 

 
```bash
.../user/2/age
 ```

와 같이 계층 구조에 조금 더 친화적인 데이터라고 생각되기 때문에

`@PathVariable`을 통해 전달하는 것이 적절하다고 생각합니다.

 

<br>반면, age와 같은 데이터는 

 
```bash
.../user?age=20
```

와 같이 검색에 조금 더 적절하다고 생각됩니다.
</span>