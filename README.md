# SPRING ADVANCED
# Lv1. 코드 개선
## 1. 코드 개선 퀴즈 - Early Return

```
        String encodedPassword = passwordEncoder.encode(signupRequest.getPassword());

        UserRole userRole = UserRole.of(signupRequest.getUserRole());

        if (userRepository.existsByEmail(signupRequest.getEmail())) {
            throw new InvalidRequestException("이미 존재하는 이메일입니다.");
        }
```

에서 조건에 맞지 않는 경우 불필요하게 encode 메서드를 실행하지 않도록

```
        if (userRepository.existsByEmail(signupRequest.getEmail())) {
            throw new InvalidRequestException("이미 존재하는 이메일입니다.");
        }

        String encodedPassword = passwordEncoder.encode(signupRequest.getPassword());

        UserRole userRole = UserRole.of(signupRequest.getUserRole());
```

와 같이 순서를 바꾸어 리팩토링

## 2. 리팩토링 퀴즈 - 불필요한 if-else 피하기

```
        if (!HttpStatus.OK.equals(responseEntity.getStatusCode())) {
            throw new ServerException("날씨 데이터를 가져오는데 실패했습니다. 상태 코드: " + responseEntity.getStatusCode());
        } else {
            if (weatherArray == null || weatherArray.length == 0) {
                throw new ServerException("날씨 데이터가 없습니다.");
            }
```

에서 가독성을 떨어뜨리고, 유지보수를 어렵게 만드는 else 구문을 없애

```
        if (!HttpStatus.OK.equals(responseEntity.getStatusCode())) {
            throw new ServerException("날씨 데이터를 가져오는데 실패했습니다. 상태 코드: " + responseEntity.getStatusCode());
        }
        if (weatherArray == null || weatherArray.length == 0) {
            throw new ServerException("날씨 데이터가 없습니다.");
```

와 같이 리팩토링

## 3. 코드 개선 퀴즈 - Validation

```
        if (userChangePasswordRequest.getNewPassword().length() < 8 ||
                !userChangePasswordRequest.getNewPassword().matches(".*\\d.*") ||
                !userChangePasswordRequest.getNewPassword().matches(".*[A-Z].*")) {
            throw new InvalidRequestException("새 비밀번호는 8자 이상이어야 하고, 숫자와 대문자를 포함해야 합니다.");
        }
```

UserService 클래스에서 위의 코드를 지우고, 해당 API의 요청 DTO에서 처리할 수 있도록 UserChangePasswordRequest 클래스에

```
    @Pattern(regexp = "(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z]).{8,}$", message = "새 비밀번호는 8자 이상이어야 하고, 숫자와 대문자를 포함해야 합니다.")
```

추가 및 UserController 클래스에서

```
    public void changePassword(@Auth AuthUser authUser, @Valid @RequestBody UserChangePasswordRequest userChangePasswordRequest) {
```

위와 같이 @RequestBody 앞에 @Valid 추가

# Lv2. N+1 문제

```
    @Query("SELECT t FROM Todo t LEFT JOIN FETCH t.user u ORDER BY t.modifiedAt DESC")
```

와 같이 fetch join을 통해 N+1 문제를 해결하고 있는 코드를

```
    @EntityGraph (attributePaths = {"user"})
```

로 변경하면서 EntityGraph를 사용하여 N+1 문제를 해결

# Lv3. 테스트코드 연습
## 1. 테스트 코드 연습 - 1

```
        boolean matches = passwordEncoder.matches(encodedPassword, rawPassword);
```

에서 matches 메서드를 확인해보니

```
    public boolean matches(String rawPassword, String encodedPassword)
```

로 구현되어 있어서

```
        boolean matches = passwordEncoder.matches(rawPassword, encodedPassword);
```

의도와 일치하도록 다음과 같이 rawPassword와 encodedPassword의 위치를 변경

또한 @ExtendWith(SpringExtension.class)로 어노테이션이 설정되어 있었는데
13:07:47.615 [main] INFO org.springframework.test.context.support.AnnotationConfigContextLoaderUtils -- Could not detect default configuration classes for test class [org.example.expert.config.PasswordEncoderTest]: PasswordEncoderTest does not declare any static, non-private, non-final, nested classes annotated with @Configuration.
와 같은 메세지가 떠서 @ExtendWith(MockitoExtension.class)로 변경

## 2. 테스트 코드 연습 - 2
### case1

```
    @Test
    public void manager_목록_조회_시_Todo가_없다면_NPE_에러를_던진다() {
        // given
        long todoId = 1L;
        given(todoRepository.findById(todoId)).willReturn(Optional.empty());

        // when & then
        InvalidRequestException exception = assertThrows(InvalidRequestException.class, () -> managerService.getManagers(todoId));
        assertEquals("Manager not found", exception.getMessage());
    }
```

에서 NPE 에러가 아닌 IRE 에러가 발생하는데 메서드 명이 NPE로 되어있어서

```
    @Test
    public void manager_목록_조회_시_Todo가_없다면_InvalidRequestException_에러를_던진다() {
        // given
        long todoId = 1L;
        given(todoRepository.findById(todoId)).willReturn(Optional.empty());

        // when & then
        InvalidRequestException exception = assertThrows(InvalidRequestException.class, () -> managerService.getManagers(todoId));
        assertEquals("Todo not found", exception.getMessage());
    }
```

메서드 명을 InvalidRequestException로 명확하게 수정했고, 예외 메세지도 "Todo not found"로 수정

### case2

```
        ServerException exception = assertThrows(ServerException.class, () -> {
            commentService.saveComment(authUser, todoId, request);
        });
```

CommetServiceTest 클래스에서 실제로 InvalidRequestException 에러가 발생하는데 ServerException을 기대값으로 Testcode에서 잘 못 설정하고 있어서

```
        InvalidRequestException exception = assertThrows(InvalidRequestException.class, () -> {
            commentService.saveComment(authUser, todoId, request);
        });
```

와 같이 적절하게 수정


### case3

```
        if (!ObjectUtils.nullSafeEquals(user.getId(), todo.getUser().getId())) {
            throw new InvalidRequestException("담당자를 등록하려고 하는 유저가 일정을 만든 유저가 유효하지 않습니다.");
        }
```

로 if 문이 구성되어 있는데, todo.getUser().getId()에서 User가 null이므로 if 문으로 넘어오기 전에 getUser()에서 NPE가 발생

```
        if (todo.getUser() == null || !ObjectUtils.nullSafeEquals(user.getId(), todo.getUser().getId())) {
            throw new InvalidRequestException("담당자를 등록하려고 하는 유저가 일정을 만든 유저가 유효하지 않습니다.");
```

따라서 뒤의 조건문을 검사하기 이 전에 todo.getUser() == null을 or 조건으로 추가하여, User가 null인 경우, 뒤의 조건문을 검사하지 않고도 if 문을 통과해서 정상적으로 예외 처리가 되도록 구현
