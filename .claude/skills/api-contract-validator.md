# API Contract Validator

Backend API Response와 App Model 간 일관성을 자동으로 검증하는 skill입니다.

## 트리거 조건

다음 상황에서 이 skill을 자동으로 적용합니다:

1. **Backend Response 수정 시**
   - `Hao-Day-Backend/api/src/main/java/**/dto/*Response.java` 파일 수정
   - 새 필드 추가, 필드 삭제, 타입 변경

2. **App Model 수정 시**
   - `Hao-Day-App/lib/features/**/data/models/*_model.dart` 파일 수정
   - `Hao-Day-App/lib/features/**/domain/entities/*_entity.dart` 파일 수정

3. **크로스 레포 작업 요청 시**
   - "API 필드 추가하고 앱에서도 반영해줘" 같은 요청
   - `/cross-repo` 커맨드 사용 시

## 매핑 규칙

### 파일 매핑
| Backend | App |
|---------|-----|
| `UserResponse.java` | `user_model.dart` 또는 `user_entity.dart` |
| `{Domain}Response.java` | `{domain}_model.dart` |

### 타입 매핑
| Java | Dart | 비고 |
|------|------|------|
| `Long`, `long` | `int` | |
| `Integer`, `int` | `int` | |
| `String` | `String` | |
| `Boolean`, `boolean` | `bool` | |
| `Double`, `double` | `double` | |
| `LocalDateTime` | `DateTime` | |
| `List<T>` | `List<T>` | |
| `Enum` | `String` 또는 Dart Enum | 앱에서 String으로 받는 경우 많음 |

### 네이밍 매핑
| Java (camelCase) | Dart (camelCase) | JSON (snake_case) |
|------------------|------------------|-------------------|
| `userId` | `userId` | `user_id` |
| `createdAt` | `createdAt` | `created_at` |
| `profileImageUrl` | `profileImageUrl` | `profile_image_url` |

## 검증 수행 방법

### 1. Backend Response 분석
```java
// UserResponse.java 예시
public class UserResponse {
    private Long id;           // → int
    private String email;      // → String
    private String name;       // → String
    private UserStatus status; // → String (enum)
    private LocalDateTime createdAt; // → DateTime
}
```

### 2. App Model 분석
```dart
// user_model.dart 예시 (freezed)
@freezed
class UserModel with _$UserModel {
  const factory UserModel({
    required int id,
    required String email,
    required String name,
    required String status,
    required DateTime createdAt,
  }) = _UserModel;
}
```

### 3. 불일치 리포트 출력
```
📋 API Contract Validation: UserResponse ↔ UserModel

✅ id: Long → int (OK)
✅ email: String → String (OK)
✅ name: String → String (OK)
✅ status: UserStatus → String (OK - Enum to String)
✅ createdAt: LocalDateTime → DateTime (OK)

Result: All fields matched!
```

### 4. 불일치 발견 시
```
📋 API Contract Validation: UserResponse ↔ UserModel

✅ id: Long → int (OK)
✅ email: String → String (OK)
❌ profileImageUrl: String → (MISSING in App)
⚠️ status: UserStatus → int (Type mismatch - expected String)

Action Required:
1. App에 profileImageUrl 필드 추가 필요
2. status 타입 확인 필요 (String 권장)
```

## 작업 흐름

### Backend에서 새 필드 추가 시
1. Response 파일 수정 감지
2. 대응하는 App Model 파일 확인
3. 누락된 필드 리포트
4. App Model 수정 제안

### App에서 새 필드 추가 시
1. Model 파일 수정 감지
2. 대응하는 Backend Response 확인
3. Backend에 없는 필드면 경고
4. Backend 추가 필요 여부 확인

## 주의사항

- `@JsonKey(name: 'xxx')` 어노테이션 확인 (Dart)
- `@JsonProperty("xxx")` 어노테이션 확인 (Java)
- nullable 필드 처리: Java `@Nullable` ↔ Dart `?` 또는 `@Default`
- App에서 일부 필드만 사용하는 것은 허용 (Backend가 더 많은 필드를 가질 수 있음)
- Backend에 없는 필드를 App에서 요구하면 에러
