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
## 3. 코드 개선 퀴즈 - Validation

# Lv2. N+1 문제

# Lv3. 테스트코드 연습
## 1. 테스트 코드 연습 - 1
## 2. 테스트 코드 연습 - 2
### case1
### case2
### case3
