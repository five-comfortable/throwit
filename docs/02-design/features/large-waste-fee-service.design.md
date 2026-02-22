# 개발 가이드: 대형폐기물 수수료 조회 웹 서비스

> 작성일: 2026-02-20
> 이 문서는 팀원이 순서대로 따라하며 프로젝트를 완성할 수 있는 실행 가이드입니다.
> 참고 프로젝트: `C:\Users\csj20\Desktop\throw_it`

---

## 목차

- [Step 1. 사전 준비](#step-1-사전-준비)
- [Step 2. DB 설계 및 구축](#step-2-db-설계-및-구축)
- [Step 3. Backend 프로젝트 생성](#step-3-backend-프로젝트-생성)
- [Step 4. Backend 글로벌 설정](#step-4-backend-글로벌-설정)
- [Step 5. Backend 도메인 개발](#step-5-backend-도메인-개발)
- [Step 6. Frontend 프로젝트 생성](#step-6-frontend-프로젝트-생성)
- [Step 7. Frontend 기반 코드 작성](#step-7-frontend-기반-코드-작성)
- [Step 8. Frontend 공통 컴포넌트 개발](#step-8-frontend-공통-컴포넌트-개발)
- [Step 9. Frontend 페이지 개발](#step-9-frontend-페이지-개발)
- [Step 10. Frontend-Backend 연동](#step-10-frontend-backend-연동)
- [Step 11. 테스트 및 QA](#step-11-테스트-및-qa)
- [Step 12. 배포](#step-12-배포)

---

## Step 1. 사전 준비

> 담당: 최세진 (PM/Server)
> 전원이 아래 도구를 설치해야 합니다.

### 1-1. 필수 설치 프로그램

| 도구 | 버전 | 다운로드 |
|------|------|----------|
| JDK | 17 | https://adoptium.net/ |
| Node.js | 18+ | https://nodejs.org/ |
| MySQL | 8.0+ | https://dev.mysql.com/downloads/ |
| Git | 최신 | https://git-scm.com/ |
| VS Code | 최신 | https://code.visualstudio.com/ |
| IntelliJ IDEA | Community/Ultimate | https://www.jetbrains.com/idea/ |

### 1-2. VS Code 추천 확장 (Frontend 팀)

```
- ESLint
- Tailwind CSS IntelliSense
- TypeScript Vue Plugin (React용이 아닌 기본 TS 지원)
- Prettier
```

### 1-3. IntelliJ 추천 플러그인 (Backend 팀)

```
- Lombok
- Spring Boot Assistant
```

### 1-4. Git 저장소 설정

```bash
# PM이 저장소를 생성하고 팀원을 초대합니다
git clone <저장소-URL>
cd thowit

# 프로젝트 폴더 구조 생성
mkdir -p backend frontend docs
```

### 1-5. 프로젝트 폴더 구조 (최종 목표)

```
thowit/
├── backend/                    # Spring Boot
│   ├── build.gradle.kts
│   ├── gradlew / gradlew.bat
│   └── src/
│       └── main/
│           ├── java/com/thowit/
│           │   ├── ThowItApplication.java
│           │   ├── domain/
│           │   │   ├── user/          # 인증
│           │   │   ├── fee/           # 수수료 조회
│           │   │   ├── disposal/      # 온라인 배출
│           │   │   ├── recycle/       # 재활용
│           │   │   └── offline/       # 오프라인 시설
│           │   └── global/
│           │       ├── config/        # CORS 등
│           │       └── exception/     # 예외 처리
│           └── resources/
│               ├── application.yml
│               └── sql/               # 스키마 + 데이터
├── frontend/                   # React + Vite + TypeScript
│   ├── package.json
│   ├── vite.config.ts
│   ├── index.html
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── index.css
│       ├── components/         # 공통 컴포넌트
│       │   ├── layout/
│       │   ├── ui/
│       │   ├── waste/
│       │   └── map/
│       ├── features/           # 기능 모듈
│       │   ├── auth/
│       │   ├── disposal/
│       │   ├── fee/
│       │   ├── mypage/
│       │   └── recycle/
│       ├── pages/              # 페이지 (17개)
│       │   ├── HomePage.tsx
│       │   ├── FeeCheckPage.tsx
│       │   ├── auth/
│       │   ├── offline/
│       │   ├── online/
│       │   ├── recycle/
│       │   └── mypage/
│       ├── services/           # API 호출 (7개)
│       ├── stores/             # Zustand 스토어
│       ├── lib/                # 유틸리티
│       │   ├── apiClient.ts
│       │   └── map/
│       ├── types/              # TypeScript 타입
│       └── router/             # 라우팅
└── docs/                       # 문서
```

---

## Step 2. DB 설계 및 구축

> 담당: 최세진, 이재훈

### 2-1. MySQL 데이터베이스 생성

MySQL에 접속하여 아래 명령어를 실행합니다.

```sql
CREATE DATABASE waste_db
  DEFAULT CHARACTER SET utf8mb4
  DEFAULT COLLATE utf8mb4_0900_ai_ci;
```

### 2-2. 테이블 생성 (schema.sql)

파일 위치: `backend/src/main/resources/sql/schema.sql`

```sql
-- ============================================
-- 대형폐기물 수수료 테이블 (공공데이터)
-- ============================================
CREATE TABLE large_waste_fee (
  id BIGINT NOT NULL AUTO_INCREMENT,
  시도명 VARCHAR(255) DEFAULT NULL,
  시군구명 VARCHAR(255) DEFAULT NULL,
  대형폐기물명 VARCHAR(255) DEFAULT NULL,
  대형폐기물구분명 VARCHAR(255) DEFAULT NULL,
  대형폐기물규격 VARCHAR(255) DEFAULT NULL,
  유무료여부 VARCHAR(255) DEFAULT NULL,
  수수료 INT DEFAULT NULL,
  관리기관명 VARCHAR(255) DEFAULT NULL,
  데이터기준일자 DATE DEFAULT NULL,
  제공기관코드 VARCHAR(20) DEFAULT NULL,
  제공기관명 VARCHAR(100) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY idx_시도명 (시도명),
  KEY idx_시군구명 (시군구명),
  KEY idx_대형폐기물명 (대형폐기물명)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ============================================
-- 폐기물 처리시설 테이블 (공공데이터)
-- ============================================
CREATE TABLE waste_facility (
  id BIGINT NOT NULL AUTO_INCREMENT,
  시설명 VARCHAR(255) DEFAULT NULL,
  소재지도로명주소 VARCHAR(255) DEFAULT NULL,
  소재지지번주소 VARCHAR(255) DEFAULT NULL,
  위도 DECIMAL(38,2) DEFAULT NULL,
  경도 DECIMAL(38,2) DEFAULT NULL,
  업종명 VARCHAR(255) DEFAULT NULL,
  전문처리분야명 VARCHAR(255) DEFAULT NULL,
  처리폐기물정보 VARCHAR(255) DEFAULT NULL,
  영업구역 VARCHAR(255) DEFAULT NULL,
  시설장비명 VARCHAR(255) DEFAULT NULL,
  허가일자 DATE DEFAULT NULL,
  전화번호 VARCHAR(255) DEFAULT NULL,
  관리기관명 VARCHAR(255) DEFAULT NULL,
  데이터기준일자 DATE DEFAULT NULL,
  제공기관코드 VARCHAR(255) DEFAULT NULL,
  제공기관명 VARCHAR(255) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY idx_시설명 (시설명),
  KEY idx_업종명 (업종명),
  KEY idx_소재지도로명주소 (소재지도로명주소(100))
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ============================================
-- 사용자 테이블
-- ============================================
CREATE TABLE users (
  id BIGINT NOT NULL AUTO_INCREMENT,
  email VARCHAR(255) NOT NULL UNIQUE,
  password VARCHAR(255) NOT NULL,
  salt VARCHAR(255) NOT NULL,
  nickname VARCHAR(255) NOT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ============================================
-- 배출 신청 테이블
-- ============================================
CREATE TABLE disposal_applications (
  id BIGINT NOT NULL AUTO_INCREMENT,
  application_number VARCHAR(20) NOT NULL UNIQUE,
  user_id BIGINT NOT NULL,
  sido VARCHAR(50) NOT NULL,
  sigungu VARCHAR(50) NOT NULL,
  disposal_address VARCHAR(255) NOT NULL,
  preferred_date DATE NOT NULL,
  total_fee INT NOT NULL DEFAULT 0,
  status ENUM('PENDING_PAYMENT','PAID','COLLECTED','CANCELLED','REFUNDED') NOT NULL DEFAULT 'PENDING_PAYMENT',
  payment_method ENUM('CARD','WIRE_TRANSFER') DEFAULT NULL,
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  KEY idx_user_id (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ============================================
-- 배출 품목 테이블
-- ============================================
CREATE TABLE disposal_items (
  id BIGINT NOT NULL AUTO_INCREMENT,
  application_id BIGINT NOT NULL,
  waste_item_name VARCHAR(255) NOT NULL,
  size_label VARCHAR(100) DEFAULT NULL,
  quantity INT NOT NULL DEFAULT 1,
  fee INT NOT NULL DEFAULT 0,
  photo_url VARCHAR(500) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY idx_application_id (application_id),
  CONSTRAINT fk_disposal_items_application
    FOREIGN KEY (application_id) REFERENCES disposal_applications(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

-- ============================================
-- 재활용 물품 테이블
-- ============================================
CREATE TABLE recycle_items (
  id BIGINT NOT NULL AUTO_INCREMENT,
  user_id BIGINT NOT NULL,
  title VARCHAR(255) NOT NULL,
  description TEXT DEFAULT NULL,
  photos LONGTEXT DEFAULT NULL,
  sido VARCHAR(50) DEFAULT NULL,
  sigungu VARCHAR(50) DEFAULT NULL,
  address VARCHAR(255) DEFAULT NULL,
  lat DECIMAL(10,7) DEFAULT NULL,
  lng DECIMAL(10,7) DEFAULT NULL,
  status ENUM('AVAILABLE','RESERVED','SOLD','WITHDRAWN') NOT NULL DEFAULT 'AVAILABLE',
  created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id),
  KEY idx_sido (sido),
  KEY idx_sigungu (sigungu),
  KEY idx_status (status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

### 2-3. 스키마 적용 및 데이터 삽입

```bash
# 스키마 적용
mysql -u root -p waste_db < backend/src/main/resources/sql/schema.sql

# 공공데이터 삽입 (참고 프로젝트에서 복사)
mysql -u root -p waste_db < backend/src/main/resources/sql/large_waste_fee_data.sql
mysql -u root -p waste_db < backend/src/main/resources/sql/waste_facility_data.sql
```

> **참고**: `large_waste_fee_data.sql`과 `waste_facility_data.sql`은 참고 프로젝트(`throw_it/backend/src/main/resources/sql/`)에서 복사합니다.

### 2-4. ERD (Entity Relationship Diagram)

```
┌──────────────────┐
│   users          │
│──────────────────│
│ id (PK)          │
│ email (UNIQUE)   │
│ password         │
│ salt             │
│ nickname         │
│ created_at       │
└──────┬───────────┘
       │ 1:N
       │
┌──────▼───────────────────┐       ┌─────────────────────────┐
│ disposal_applications    │       │ recycle_items            │
│──────────────────────────│       │─────────────────────────│
│ id (PK)                  │       │ id (PK)                 │
│ application_number (UQ)  │       │ user_id (FK→users)      │
│ user_id (FK→users)       │       │ title                   │
│ sido, sigungu            │       │ description             │
│ disposal_address         │       │ photos (JSON)           │
│ preferred_date           │       │ sido, sigungu, address  │
│ total_fee                │       │ lat, lng                │
│ status (ENUM)            │       │ status (ENUM)           │
│ payment_method (ENUM)    │       │ created_at              │
│ created_at, updated_at   │       └─────────────────────────┘
└──────┬───────────────────┘
       │ 1:N
       │
┌──────▼───────────────────┐
│ disposal_items           │
│──────────────────────────│
│ id (PK)                  │
│ application_id (FK)      │
│ waste_item_name          │
│ size_label               │
│ quantity                 │
│ fee                      │
│ photo_url                │
└──────────────────────────┘

┌──────────────────────────┐       ┌─────────────────────────┐
│ large_waste_fee          │       │ waste_facility           │
│──────────────────────────│       │─────────────────────────│
│ id (PK)                  │       │ id (PK)                 │
│ 시도명, 시군구명          │       │ 시설명                   │
│ 대형폐기물명              │       │ 소재지도로명주소          │
│ 대형폐기물구분명          │       │ 위도, 경도               │
│ 대형폐기물규격            │       │ 업종명                   │
│ 수수료                   │       │ 전화번호                 │
│ ...                      │       │ ...                     │
└──────────────────────────┘       └─────────────────────────┘
```

---

## Step 3. Backend 프로젝트 생성

> 담당: 최세진

### 3-1. Spring Initializr에서 프로젝트 생성

https://start.spring.io/ 에서 다음 설정으로 생성합니다:

| 항목 | 값 |
|------|-----|
| Project | Gradle - Kotlin |
| Language | Java |
| Spring Boot | 3.4.x (최신 안정) |
| Group | com.thowit |
| Artifact | backend |
| Java | 17 |

**Dependencies 선택:**
- Spring Web
- Spring Data JPA
- MySQL Driver
- Lombok
- Validation

### 3-2. build.gradle.kts (최종 형태)

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.4.5"
    id("io.spring.dependency-management") version "1.1.7"
}

group = "com.thowit"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

configurations {
    compileOnly {
        extendsFrom(configurations.annotationProcessor.get())
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    runtimeOnly("com.mysql:mysql-connector-j")
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### 3-3. application.yml

파일 위치: `backend/src/main/resources/application.yml`

```yaml
spring:
  profiles:
    active: local

  datasource:
    url: jdbc:mysql://localhost:3306/waste_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Seoul&characterEncoding=UTF-8
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:}
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQLDialect

  sql:
    init:
      mode: never

server:
  port: 8080
```

### 3-4. application-local.yml (Git에 올리지 않음!)

파일 위치: `backend/src/main/resources/application-local.yml`

```yaml
spring:
  datasource:
    username: root
    password: 여기에_본인_비밀번호
```

> **.gitignore에 반드시 추가**: `application-local.yml`

### 3-5. Backend 실행 확인

```bash
cd backend
./gradlew bootRun
```

`http://localhost:8080`에 접속하여 서버가 뜨는지 확인합니다. (404 에러가 나면 정상)

---

## Step 4. Backend 글로벌 설정

> 담당: 최세진

### 4-1. CORS 설정

파일 위치: `backend/src/main/java/com/thowit/global/config/CorsConfig.java`

```java
package com.thowit.global.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

import java.util.List;

@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowedOrigins(List.of(
            "http://localhost:5173",
            "http://localhost:5174",
            "http://localhost:3000"
        ));
        config.setAllowedMethods(List.of("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"));
        config.setAllowedHeaders(List.of("*"));
        config.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", config);
        return new CorsFilter(source);
    }
}
```

### 4-2. 글로벌 예외 처리

**BusinessException.java**

파일 위치: `backend/src/main/java/com/thowit/global/exception/BusinessException.java`

```java
package com.thowit.global.exception;

import lombok.Getter;
import org.springframework.http.HttpStatus;

@Getter
public class BusinessException extends RuntimeException {

    private final HttpStatus status;
    private final String code;

    public BusinessException(HttpStatus status, String code, String message) {
        super(message);
        this.status = status;
        this.code = code;
    }

    public static BusinessException notFound(String code, String message) {
        return new BusinessException(HttpStatus.NOT_FOUND, code, message);
    }

    public static BusinessException badRequest(String code, String message) {
        return new BusinessException(HttpStatus.BAD_REQUEST, code, message);
    }

    public static BusinessException conflict(String code, String message) {
        return new BusinessException(HttpStatus.CONFLICT, code, message);
    }
}
```

**ErrorResponse.java**

파일 위치: `backend/src/main/java/com/thowit/global/exception/ErrorResponse.java`

```java
package com.thowit.global.exception;

import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class ErrorResponse {
    private String code;
    private String message;
}
```

**GlobalExceptionHandler.java**

파일 위치: `backend/src/main/java/com/thowit/global/exception/GlobalExceptionHandler.java`

```java
package com.thowit.global.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.util.stream.Collectors;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(BusinessException e) {
        return ResponseEntity
                .status(e.getStatus())
                .body(ErrorResponse.builder()
                        .code(e.getCode())
                        .message(e.getMessage())
                        .build());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(FieldError::getDefaultMessage)
                .collect(Collectors.joining(", "));

        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(ErrorResponse.builder()
                        .code("VALIDATION_ERROR")
                        .message(message)
                        .build());
    }
}
```

---

## Step 5. Backend 도메인 개발

> 담당: 최세진(User, Disposal), 이재훈(Fee, Recycle, Offline)

각 도메인은 아래 패턴으로 개발합니다. **반드시 이 순서로 파일을 생성하세요.**

```
domain/{도메인명}/
├── {Entity}.java           ← 1. 엔티티 먼저
├── {Repository}.java       ← 2. 레포지토리
├── dto/
│   ├── {Request}.java      ← 3. 요청 DTO
│   └── {Response}.java     ← 4. 응답 DTO
├── {Service}.java          ← 5. 서비스 (비즈니스 로직)
└── {Controller}.java       ← 6. 컨트롤러 (API 엔드포인트)
```

---

### 5-1. User 도메인 (담당: 최세진)

#### Entity: User.java

파일 위치: `backend/src/main/java/com/thowit/domain/user/User.java`

```java
package com.thowit.domain.user;

import jakarta.persistence.*;
import lombok.*;

import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.time.LocalDateTime;
import java.util.Base64;

@Entity
@Table(name = "users")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String salt;

    @Column(nullable = false)
    private String nickname;

    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
    }

    // Salt 생성 (16바이트 랜덤)
    public static String generateSalt() {
        byte[] saltBytes = new byte[16];
        new SecureRandom().nextBytes(saltBytes);
        return Base64.getEncoder().encodeToString(saltBytes);
    }

    // SHA-256 + Salt 해싱
    public static String hashPassword(String rawPassword, String salt) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            md.update(salt.getBytes(StandardCharsets.UTF_8));
            byte[] hashed = md.digest(rawPassword.getBytes(StandardCharsets.UTF_8));
            return Base64.getEncoder().encodeToString(hashed);
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("SHA-256 not available", e);
        }
    }

    // 비밀번호 검증
    public boolean checkPassword(String rawPassword) {
        return this.password.equals(hashPassword(rawPassword, this.salt));
    }
}
```

#### Repository: UserRepository.java

```java
package com.thowit.domain.user;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
}
```

#### DTO: SignupRequest.java

파일 위치: `backend/src/main/java/com/thowit/domain/user/dto/SignupRequest.java`

```java
package com.thowit.domain.user.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Size;
import lombok.Getter;

@Getter
public class SignupRequest {

    @Email(message = "유효한 이메일을 입력해주세요")
    @NotBlank(message = "이메일은 필수입니다")
    private String email;

    @NotBlank(message = "비밀번호는 필수입니다")
    @Size(min = 6, message = "비밀번호는 6자 이상이어야 합니다")
    private String password;

    @NotBlank(message = "닉네임은 필수입니다")
    private String nickname;
}
```

#### DTO: LoginRequest.java

```java
package com.thowit.domain.user.dto;

import jakarta.validation.constraints.Email;
import jakarta.validation.constraints.NotBlank;
import lombok.Getter;

@Getter
public class LoginRequest {

    @Email(message = "유효한 이메일을 입력해주세요")
    @NotBlank(message = "이메일은 필수입니다")
    private String email;

    @NotBlank(message = "비밀번호는 필수입니다")
    private String password;
}
```

#### DTO: UserResponse.java

```java
package com.thowit.domain.user.dto;

import com.thowit.domain.user.User;
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class UserResponse {
    private Long id;
    private String email;
    private String nickname;

    public static UserResponse from(User user) {
        return UserResponse.builder()
                .id(user.getId())
                .email(user.getEmail())
                .nickname(user.getNickname())
                .build();
    }
}
```

#### Service: AuthService.java

```java
package com.thowit.domain.user;

import com.thowit.domain.user.dto.LoginRequest;
import com.thowit.domain.user.dto.SignupRequest;
import com.thowit.domain.user.dto.UserResponse;
import com.thowit.global.exception.BusinessException;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class AuthService {

    private final UserRepository userRepository;

    @Transactional
    public UserResponse signup(SignupRequest request) {
        if (userRepository.existsByEmail(request.getEmail())) {
            throw BusinessException.conflict("EMAIL_DUPLICATE", "이미 사용 중인 이메일입니다");
        }

        String salt = User.generateSalt();
        String hashedPassword = User.hashPassword(request.getPassword(), salt);

        User user = User.builder()
                .email(request.getEmail())
                .password(hashedPassword)
                .salt(salt)
                .nickname(request.getNickname())
                .build();

        User saved = userRepository.save(user);
        return UserResponse.from(saved);
    }

    public UserResponse login(LoginRequest request) {
        User user = userRepository.findByEmail(request.getEmail())
                .orElseThrow(() -> BusinessException.badRequest(
                    "LOGIN_FAILED", "이메일 또는 비밀번호가 올바르지 않습니다"));

        if (!user.checkPassword(request.getPassword())) {
            throw BusinessException.badRequest(
                "LOGIN_FAILED", "이메일 또는 비밀번호가 올바르지 않습니다");
        }

        return UserResponse.from(user);
    }

    public UserResponse getUser(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> BusinessException.notFound(
                    "USER_NOT_FOUND", "사용자를 찾을 수 없습니다"));
        return UserResponse.from(user);
    }
}
```

#### Controller: AuthController.java

```java
package com.thowit.domain.user;

import com.thowit.domain.user.dto.LoginRequest;
import com.thowit.domain.user.dto.SignupRequest;
import com.thowit.domain.user.dto.UserResponse;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthService authService;

    @PostMapping("/signup")
    public ResponseEntity<UserResponse> signup(@Valid @RequestBody SignupRequest request) {
        return ResponseEntity.ok(authService.signup(request));
    }

    @PostMapping("/login")
    public ResponseEntity<UserResponse> login(@Valid @RequestBody LoginRequest request) {
        return ResponseEntity.ok(authService.login(request));
    }

    @GetMapping("/me")
    public ResponseEntity<UserResponse> getMe(@RequestHeader("X-User-Id") Long userId) {
        return ResponseEntity.ok(authService.getUser(userId));
    }
}
```

#### API 테스트 (Postman 또는 curl)

```bash
# 회원가입
curl -X POST http://localhost:8080/api/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"123456","nickname":"테스터"}'

# 로그인
curl -X POST http://localhost:8080/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"123456"}'

# 내 정보 (응답의 id를 사용)
curl http://localhost:8080/api/auth/me -H "X-User-Id: 1"
```

---

### 5-2. Fee 도메인 (담당: 이재훈)

> **핵심 기능**입니다. 22,819건의 수수료 데이터를 조회합니다.

#### API 목록

| Method | Endpoint | 기능 |
|--------|----------|------|
| GET | `/api/regions/sido` | 시도 목록 (17개) |
| GET | `/api/regions/sigungu?sido=서울특별시` | 해당 시도의 시군구 목록 |
| GET | `/api/waste/categories?sigungu=강남구` | 폐기물 카테고리(구분명) 목록 |
| GET | `/api/waste/items?sigungu=강남구&category=가구류&keyword=소파` | 품목 검색 |
| GET | `/api/fees?sido=서울특별시&sigungu=강남구&wasteName=소파` | 수수료 조회 |

#### 개발할 파일 목록

```
domain/fee/
├── LargeWasteFee.java          ← Entity
├── LargeWasteFeeRepository.java ← Repository (JPQL 쿼리 포함)
├── dto/
│   ├── FeeInfoDto.java          ← 수수료 정보 응답
│   └── WasteItemResult.java     ← 품목 검색 결과
├── LargeWasteFeeService.java   ← 비즈니스 로직
└── LargeWasteFeeController.java ← 컨트롤러
```

**핵심 비즈니스 로직:**
- `대형폐기물규격`이 null인 경우 수수료 금액별로 소형/중형/대형/특대형 자동 매핑
- 결과를 수수료 오름차순 정렬

---

### 5-3. Disposal 도메인 (담당: 최세진)

#### API 목록

| Method | Endpoint | 기능 |
|--------|----------|------|
| POST | `/api/disposals` | 배출 신청 생성 |
| GET | `/api/disposals/my` | 내 신청 목록 |
| GET | `/api/disposals/{id}` | 신청 상세 |
| PATCH | `/api/disposals/{id}/cancel` | 취소 |
| POST | `/api/disposals/{id}/payment` | 결제 처리 |

#### 개발할 파일 목록

```
domain/disposal/
├── DisposalApplication.java     ← Entity (1:N 관계)
├── DisposalItem.java            ← Entity (FK: application_id)
├── DisposalStatus.java          ← ENUM (PENDING_PAYMENT, PAID, COLLECTED, CANCELLED, REFUNDED)
├── PaymentMethod.java           ← ENUM (CARD, WIRE_TRANSFER)
├── DisposalApplicationRepository.java
├── dto/
│   ├── DisposalCreateRequest.java
│   ├── DisposalItemRequest.java
│   ├── DisposalResponse.java
│   ├── DisposalItemResponse.java
│   └── PaymentRequest.java
├── DisposalService.java
└── DisposalController.java
```

**핵심 비즈니스 로직:**
- 신청번호 자동 생성: `{지역코드 2자}-{YYYYMMDD}-{5자리 시퀀스}` (예: GN-20260220-00001)
- preferred_date는 오늘 이후 날짜만 허용
- 취소는 PENDING_PAYMENT, PAID 상태에서만 가능

---

### 5-4. Recycle 도메인 (담당: 이재훈)

#### API 목록

| Method | Endpoint | 기능 |
|--------|----------|------|
| GET | `/api/recycle/items?sigungu=강남구` | 목록 조회 |
| GET | `/api/recycle/items/my` | 내 물품 |
| POST | `/api/recycle/items` | 등록 |
| PATCH | `/api/recycle/items/{id}/status?status=RESERVED` | 상태 변경 |
| DELETE | `/api/recycle/items/{id}` | 삭제 |

#### 개발할 파일 목록

```
domain/recycle/
├── RecycleItem.java             ← Entity (photos는 JSON 문자열)
├── RecycleStatus.java           ← ENUM (AVAILABLE, RESERVED, SOLD, WITHDRAWN)
├── RecycleItemRepository.java
├── dto/
│   ├── RecycleCreateRequest.java
│   └── RecycleItemResponse.java
├── RecycleService.java
└── RecycleController.java
```

---

### 5-5. Offline 도메인 (담당: 이재훈)

#### API 목록

| Method | Endpoint | 기능 |
|--------|----------|------|
| GET | `/api/offline/sticker-shops?sigungu=강남구` | 스티커 판매소 |
| GET | `/api/offline/centers?sigungu=강남구` | 주민센터 |
| GET | `/api/offline/transport?sigungu=강남구` | 운반 대행 |
| GET | `/api/offline/waste-facilities?sido=서울특별시&sigungu=강남구` | 처리시설 |

#### 개발할 파일 목록

```
domain/offline/
├── WasteFacility.java           ← Entity (DB 테이블 매핑)
├── WasteFacilityRepository.java
├── dto/
│   ├── StickerShopResponse.java
│   ├── CommunityCenterResponse.java
│   ├── TransportCompanyResponse.java
│   └── WasteFacilityResponse.java
├── OfflineService.java
└── OfflineController.java
```

> **참고**: 스티커 판매소, 주민센터, 운반 대행은 현재 하드코딩 샘플 데이터를 사용합니다. 이후 공공데이터 API 또는 DB로 전환할 수 있습니다.

---

### 5-6. Backend 개발 완료 체크리스트

```
[ ] User 도메인 - signup, login, me API 동작 확인
[ ] Fee 도메인 - 시도/시군구/카테고리/품목/수수료 API 동작 확인
[ ] Disposal 도메인 - 신청 생성/조회/취소/결제 API 동작 확인
[ ] Recycle 도메인 - 등록/조회/상태변경/삭제 API 동작 확인
[ ] Offline 도메인 - 4종 시설 조회 API 동작 확인
[ ] 모든 API를 Postman 또는 curl로 테스트 완료
```

---

## Step 6. Frontend 프로젝트 생성

> 담당: 최가을

### 6-1. 프로젝트 생성

```bash
cd thowit
npm create vite@latest frontend -- --template react-ts
cd frontend
```

### 6-2. 의존성 설치

```bash
# 핵심 의존성
npm install react-router-dom zustand @tanstack/react-query react-hook-form

# 개발 의존성 (Tailwind CSS)
npm install -D tailwindcss @tailwindcss/vite
```

### 6-3. vite.config.ts

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'
import path from 'path'

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
```

### 6-4. tsconfig.app.json에 경로 별칭 추가

`compilerOptions`에 아래 내용을 추가합니다:

```json
{
  "compilerOptions": {
    // ... 기존 설정 유지 ...
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### 6-5. index.css (Tailwind CSS + 디자인 시스템)

파일 위치: `frontend/src/index.css`

```css
@import "tailwindcss";

@theme {
  --color-primary: #2563eb;
  --color-primary-dark: #1d4ed8;
  --color-secondary: #64748b;
  --color-success: #22c55e;
  --color-warning: #f59e0b;
  --color-danger: #ef4444;
  --color-bg: #f8fafc;
  --color-surface: #ffffff;
  --color-text: #1e293b;
  --color-text-secondary: #64748b;
  --color-border: #e2e8f0;
}

body {
  margin: 0;
  font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  background-color: var(--color-bg);
  color: var(--color-text);
  -webkit-font-smoothing: antialiased;
}

#root {
  min-height: 100vh;
}

@keyframes slide-up {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}

.animate-slide-up {
  animation: slide-up 0.3s ease-out;
}
```

### 6-6. index.html

```html
<!doctype html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>대형폐기물 배출 도우미</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

### 6-7. .env 파일 생성

파일 위치: `frontend/.env`

```
VITE_API_BASE_URL=http://localhost:8080
VITE_MAP_API_KEY=your_kakao_map_api_key_here
```

### 6-8. Frontend 실행 확인

```bash
npm run dev
```

`http://localhost:5173`에 접속하여 Vite 기본 화면이 뜨는지 확인합니다.

---

## Step 7. Frontend 기반 코드 작성

> 담당: 최가을

### 7-1. API 클라이언트 (apiClient.ts)

파일 위치: `frontend/src/lib/apiClient.ts`

```typescript
const BASE_URL = import.meta.env.VITE_API_BASE_URL ?? 'http://localhost:8080';

export async function apiFetch<T>(path: string, init?: RequestInit): Promise<T> {
  const { headers: customHeaders, ...rest } = init ?? {};
  const res = await fetch(`${BASE_URL}${path}`, {
    ...rest,
    headers: {
      'Content-Type': 'application/json',
      ...(customHeaders as Record<string, string>),
    },
  });

  if (!res.ok) {
    const text = await res.text();
    let message = `HTTP ${res.status}`;
    try {
      const json = JSON.parse(text);
      message = json.message || message;
    } catch {
      message = text || message;
    }
    throw new Error(message);
  }

  return res.json() as Promise<T>;
}
```

### 7-2. TypeScript 타입 정의

**auth.ts** (`frontend/src/types/auth.ts`)

```typescript
export interface User {
  id: number;
  email: string;
  nickname: string;
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface SignupRequest {
  email: string;
  password: string;
  nickname: string;
}
```

**fee.ts** (`frontend/src/types/fee.ts`)

```typescript
export interface FeeInfo {
  wasteName: string;
  sizeLabel: string;
  fee: number;
}
```

**region.ts** (`frontend/src/types/region.ts`)

```typescript
export interface Region {
  sido: string;
  sigungu: string;
}
```

**disposal.ts** (`frontend/src/types/disposal.ts`)

```typescript
export interface DisposalApplication {
  id: number;
  applicationNumber: string;
  sido: string;
  sigungu: string;
  disposalAddress: string;
  preferredDate: string;
  totalFee: number;
  status: DisposalStatus;
  paymentMethod: string | null;
  items: DisposalItem[];
  createdAt: string;
}

export interface DisposalItem {
  id: number;
  wasteItemName: string;
  sizeLabel: string;
  quantity: number;
  fee: number;
  photoUrl: string | null;
}

export type DisposalStatus = 'PENDING_PAYMENT' | 'PAID' | 'COLLECTED' | 'CANCELLED' | 'REFUNDED';

export interface DisposalCreateRequest {
  sido: string;
  sigungu: string;
  disposalAddress: string;
  preferredDate: string;
  items: {
    wasteItemName: string;
    sizeLabel: string;
    quantity: number;
    fee: number;
  }[];
}
```

**recycle.ts** (`frontend/src/types/recycle.ts`)

```typescript
export interface RecycleItem {
  id: number;
  userId: number;
  title: string;
  description: string;
  photos: string[];
  sido: string;
  sigungu: string;
  address: string;
  status: RecycleStatus;
  createdAt: string;
}

export type RecycleStatus = 'AVAILABLE' | 'RESERVED' | 'SOLD' | 'WITHDRAWN';

export interface RecycleCreateRequest {
  title: string;
  description: string;
  photos: string[];
  sido: string;
  sigungu: string;
  address: string;
}
```

**waste.ts** (`frontend/src/types/waste.ts`)

```typescript
export interface WasteItem {
  wasteName: string;
  category: string;
}
```

**offline.ts** (`frontend/src/types/offline.ts`)

```typescript
export interface StickerShop {
  name: string;
  address: string;
  phone: string;
}

export interface CommunityCenter {
  name: string;
  address: string;
  phone: string;
}

export interface TransportCompany {
  name: string;
  phone: string;
  area: string;
}

export interface WasteFacility {
  id: number;
  name: string;
  address: string;
  lat: number;
  lng: number;
  businessType: string;
  phone: string;
}
```

### 7-3. API 서비스 레이어 (7개)

**authService.ts** (`frontend/src/services/authService.ts`)

```typescript
import { apiFetch } from '@/lib/apiClient';
import type { User, LoginRequest, SignupRequest } from '@/types/auth';

export const authService = {
  async signup(data: SignupRequest): Promise<User> {
    return apiFetch<User>('/api/auth/signup', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  },

  async login(data: LoginRequest): Promise<User> {
    return apiFetch<User>('/api/auth/login', {
      method: 'POST',
      body: JSON.stringify(data),
    });
  },

  async getMe(userId: number): Promise<User> {
    return apiFetch<User>('/api/auth/me', {
      headers: { 'X-User-Id': String(userId) },
    });
  },
};
```

> 나머지 6개 서비스(regionService, wasteService, feeService, disposalService, offlineService, recycleService)도 같은 패턴으로 작성합니다. Backend API 목록(Step 5)의 엔드포인트와 1:1 대응됩니다.

### 7-4. 인증 상태 관리 (AuthContext)

파일 위치: `frontend/src/features/auth/AuthContext.tsx`

```typescript
import { createContext, useContext, useState, useEffect, useCallback } from 'react';
import { authService } from '@/services/authService';
import type { User, LoginRequest, SignupRequest } from '@/types/auth';

interface AuthContextType {
  user: User | null;
  loading: boolean;
  login: (data: LoginRequest) => Promise<void>;
  signup: (data: SignupRequest) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

const STORAGE_KEY = 'thowit_user';

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const stored = localStorage.getItem(STORAGE_KEY);
    if (stored) {
      try {
        setUser(JSON.parse(stored));
      } catch {
        localStorage.removeItem(STORAGE_KEY);
      }
    }
    setLoading(false);
  }, []);

  const login = useCallback(async (data: LoginRequest) => {
    const u = await authService.login(data);
    setUser(u);
    localStorage.setItem(STORAGE_KEY, JSON.stringify(u));
  }, []);

  const signup = useCallback(async (data: SignupRequest) => {
    const u = await authService.signup(data);
    setUser(u);
    localStorage.setItem(STORAGE_KEY, JSON.stringify(u));
  }, []);

  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem(STORAGE_KEY);
  }, []);

  return (
    <AuthContext.Provider value={{ user, loading, login, signup, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error('useAuth must be used within AuthProvider');
  return ctx;
}
```

### 7-5. 라우팅 설정

파일 위치: `frontend/src/router/index.tsx`

```typescript
import { createBrowserRouter } from 'react-router-dom';
import App from '@/App';
import HomePage from '@/pages/HomePage';
import FeeCheckPage from '@/pages/FeeCheckPage';
import OfflinePage from '@/pages/offline/OfflinePage';
import StickerShopsPage from '@/pages/offline/StickerShopsPage';
import CentersPage from '@/pages/offline/CentersPage';
import TransportPage from '@/pages/offline/TransportPage';
import OnlinePage from '@/pages/online/OnlinePage';
import ApplyPage from '@/pages/online/ApplyPage';
import ReviewPage from '@/pages/online/ReviewPage';
import PaymentPage from '@/pages/online/PaymentPage';
import CompletePage from '@/pages/online/CompletePage';
import RecyclePage from '@/pages/recycle/RecyclePage';
import RegisterPage from '@/pages/recycle/RegisterPage';
import LoginPage from '@/pages/auth/LoginPage';
import SignupPage from '@/pages/auth/SignupPage';
import MyPage from '@/pages/mypage/MyPage';
import ReceiptPage from '@/pages/mypage/ReceiptPage';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <App />,
    children: [
      { index: true, element: <HomePage /> },
      { path: 'fee-check', element: <FeeCheckPage /> },
      { path: 'offline', element: <OfflinePage /> },
      { path: 'offline/sticker-shops', element: <StickerShopsPage /> },
      { path: 'offline/centers', element: <CentersPage /> },
      { path: 'offline/transport', element: <TransportPage /> },
      { path: 'online', element: <OnlinePage /> },
      { path: 'online/apply', element: <ApplyPage /> },
      { path: 'online/review', element: <ReviewPage /> },
      { path: 'online/payment', element: <PaymentPage /> },
      { path: 'online/complete', element: <CompletePage /> },
      { path: 'recycle', element: <RecyclePage /> },
      { path: 'recycle/register', element: <RegisterPage /> },
      { path: 'login', element: <LoginPage /> },
      { path: 'signup', element: <SignupPage /> },
      { path: 'mypage', element: <MyPage /> },
      { path: 'mypage/receipt/:id', element: <ReceiptPage /> },
    ],
  },
]);
```

### 7-6. main.tsx

```typescript
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import { RouterProvider } from 'react-router-dom'
import { router } from '@/router'
import './index.css'

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>,
)
```

### 7-7. App.tsx (루트 레이아웃)

```typescript
import { useEffect } from 'react'
import { Outlet, useLocation } from 'react-router-dom'
import MobileContainer from '@/components/layout/MobileContainer'
import BottomNav from '@/components/layout/BottomNav'
import { AuthProvider } from '@/features/auth/AuthContext'

function ScrollToTop() {
  const { pathname } = useLocation()

  useEffect(() => {
    window.scrollTo(0, 0)
  }, [pathname])

  return null
}

export default function App() {
  return (
    <AuthProvider>
      <MobileContainer>
        <ScrollToTop />
        <div className="pb-16">
          <Outlet />
        </div>
        <BottomNav />
      </MobileContainer>
    </AuthProvider>
  )
}
```

---

## Step 8. Frontend 공통 컴포넌트 개발

> 담당: 최은아

### 8-1. Layout 컴포넌트

**MobileContainer.tsx** (`frontend/src/components/layout/MobileContainer.tsx`)

```typescript
interface MobileContainerProps {
  children: React.ReactNode;
}

export default function MobileContainer({ children }: MobileContainerProps) {
  return (
    <div className="mx-auto min-h-screen max-w-[428px] bg-gray-50">
      {children}
    </div>
  );
}
```

**Header.tsx** (`frontend/src/components/layout/Header.tsx`)

```typescript
import { useNavigate } from 'react-router-dom';

interface HeaderProps {
  title: string;
  showBack?: boolean;
}

export default function Header({ title, showBack = true }: HeaderProps) {
  const navigate = useNavigate();

  return (
    <header className="sticky top-0 z-10 flex items-center gap-3 bg-white px-4 py-3 shadow-sm">
      {showBack && (
        <button onClick={() => navigate(-1)} className="text-gray-600">
          ← 뒤로
        </button>
      )}
      <h1 className="text-lg font-bold">{title}</h1>
    </header>
  );
}
```

**BottomNav.tsx** (`frontend/src/components/layout/BottomNav.tsx`)
- 5개 메뉴: 홈(/), 수수료(/fee-check), 배출(/online), 재활용(/recycle), 마이(/mypage)
- `useLocation()`으로 현재 경로에 따라 활성 메뉴 표시

**ProgressBar.tsx** (`frontend/src/components/layout/ProgressBar.tsx`)
- 배출 신청 4단계 진행 표시용
- props: `currentStep`, `totalSteps`

### 8-2. UI 컴포넌트

아래 컴포넌트를 `frontend/src/components/ui/` 에 생성합니다:

| 컴포넌트 | 파일명 | 설명 |
|----------|--------|------|
| Button | `Button.tsx` | variant(primary/secondary/outline), disabled 지원 |
| Card | `Card.tsx` | 클릭 가능한 카드 컨테이너 |
| Input | `Input.tsx` | label, error message 지원 |
| Select | `Select.tsx` | 드롭다운 선택 |
| Modal | `Modal.tsx` | 알림/확인 대화상자 |
| Badge | `Badge.tsx` | 상태 뱃지 (색상 variant) |
| SearchBar | `SearchBar.tsx` | 검색창 |
| DatePicker | `DatePicker.tsx` | 날짜 선택 |

**Card.tsx 예시:**

```typescript
interface CardProps {
  children: React.ReactNode;
  className?: string;
  onClick?: () => void;
}

export default function Card({ children, className = '', onClick }: CardProps) {
  return (
    <div
      className={`rounded-xl bg-white p-4 shadow-sm ${className}`}
      onClick={onClick}
    >
      {children}
    </div>
  );
}
```

### 8-3. Waste 컴포넌트 (`frontend/src/components/waste/`)

| 컴포넌트 | 설명 |
|----------|------|
| CategoryTree | 카테고리 필터 (가구류, 가전류 등) |
| WasteSearchBar | 폐기물 품목 검색 |
| WasteItemCard | 검색된 품목 정보 표시 |
| SizeSelector | 크기/수량 선택 |
| FeeResultCard | 수수료 결과 표시 |

### 8-4. Map 컴포넌트 (`frontend/src/components/map/`)

| 컴포넌트 | 설명 |
|----------|------|
| MapView | Kakao Maps 지도 표시 |
| MapPlaceholder | API 키 없을 때 대체 UI |
| LocationCard | 위치 정보 카드 |

**Kakao Maps 어댑터 패턴:**

```
lib/map/
├── MapAdapter.ts         ← 인터페이스 정의
├── KakaoMapAdapter.ts    ← 실제 카카오맵 구현
├── MockMapAdapter.ts     ← 개발용 목 구현
├── createMapAdapter.ts   ← 팩토리 함수 (API 키 유무에 따라 선택)
└── useMap.ts             ← 지도 사용 훅
```

---

## Step 9. Frontend 페이지 개발

> 담당: 최가을(6페이지), 최은아(6페이지), 강해원(5페이지)

### 개발 순서 (권장)

**1차 (핵심 기능 - 최가을)**
1. `HomePage.tsx` - 메인 화면
2. `FeeCheckPage.tsx` - 수수료 조회 (핵심)

**2차 (인증 - 최은아)**
3. `LoginPage.tsx` - 로그인
4. `SignupPage.tsx` - 회원가입

**3차 (배출 신청 플로우 - 최가을 + 강해원)**
5. `OnlinePage.tsx` - 배출 안내 (강해원)
6. `ApplyPage.tsx` - 신청 폼 (최가을)
7. `ReviewPage.tsx` - 검토 (최가을)
8. `PaymentPage.tsx` - 결제 (최가을)
9. `CompletePage.tsx` - 완료 (최가을)

**4차 (오프라인 - 최은아)**
10. `OfflinePage.tsx` - 메뉴
11. `StickerShopsPage.tsx` - 스티커 판매소
12. `CentersPage.tsx` - 주민센터
13. `TransportPage.tsx` - 운반 대행

**5차 (마이페이지 + 재활용 - 강해원)**
14. `MyPage.tsx` - 신청 내역
15. `ReceiptPage.tsx` - 영수증
16. `RecyclePage.tsx` - 재활용 목록
17. `RegisterPage.tsx` - 물품 등록

### HomePage 예시 코드

```typescript
import { useNavigate } from 'react-router-dom'
import Card from '@/components/ui/Card'
import { useAuth } from '@/features/auth/AuthContext'

export default function HomePage() {
  const navigate = useNavigate()
  const { user, logout } = useAuth()

  return (
    <div className="p-4">
      {/* 로그인 상태 표시 */}
      <div className="flex items-center justify-end py-2">
        {user ? (
          <div className="flex items-center gap-2">
            <span className="text-sm text-gray-700 font-medium">{user.nickname}님</span>
            <button onClick={logout} className="text-xs text-gray-400 hover:text-gray-600">
              로그아웃
            </button>
          </div>
        ) : (
          <button onClick={() => navigate('/login')} className="text-sm text-blue-600 font-medium">
            로그인
          </button>
        )}
      </div>

      {/* 타이틀 */}
      <div className="text-center py-4">
        <h1 className="text-2xl font-bold text-gray-900">대형폐기물 배출 도우미</h1>
        <p className="text-sm text-gray-500 mt-1">수수료 조회부터 배출까지 한번에</p>
      </div>

      {/* 메인 카드: 수수료 조회 */}
      <Card
        className="bg-primary text-black mb-4 cursor-pointer active:opacity-90"
        onClick={() => navigate('/fee-check')}
      >
        <div className="py-4 text-center">
          <div className="text-3xl mb-2">💰</div>
          <div className="text-lg font-bold">수수료 조회하기</div>
          <div className="text-sm opacity-90 mt-1">내 폐기물의 수수료를 바로 확인하세요</div>
        </div>
      </Card>

      {/* 4개 서브 카드 */}
      <div className="grid grid-cols-2 gap-3">
        <Card className="cursor-pointer" onClick={() => navigate('/offline')}>
          <div className="text-center py-3">
            <div className="text-2xl mb-1">📋</div>
            <div className="font-semibold text-sm">오프라인 배출</div>
          </div>
        </Card>
        <Card className="cursor-pointer" onClick={() => navigate('/online')}>
          <div className="text-center py-3">
            <div className="text-2xl mb-1">💻</div>
            <div className="font-semibold text-sm">온라인 배출</div>
          </div>
        </Card>
        <Card className="cursor-pointer" onClick={() => navigate('/offline/transport')}>
          <div className="text-center py-3">
            <div className="text-2xl mb-1">🚛</div>
            <div className="font-semibold text-sm">운반 대행</div>
          </div>
        </Card>
        <Card className="cursor-pointer" onClick={() => navigate('/recycle')}>
          <div className="text-center py-3">
            <div className="text-2xl mb-1">♻️</div>
            <div className="font-semibold text-sm">재활용 역경매</div>
          </div>
        </Card>
      </div>
    </div>
  )
}
```

---

## Step 10. Frontend-Backend 연동

> 담당: 전원

### 10-1. 연동 확인 체크리스트

```
[ ] Backend 서버 실행 (localhost:8080)
[ ] Frontend 서버 실행 (localhost:5173)
[ ] 회원가입 → 로그인 → 마이페이지 흐름
[ ] 수수료 조회: 시도 선택 → 시군구 → 카테고리 → 품목 → 수수료 확인
[ ] 배출 신청: 정보 입력 → 검토 → 결제 → 완료 → 마이페이지에서 확인
[ ] 재활용: 등록 → 목록에서 확인 → 상태 변경
[ ] 오프라인: 각 카테고리 시설 목록 + 지도 확인
```

### 10-2. 흔한 에러 대응

| 에러 | 원인 | 해결 |
|------|------|------|
| CORS 에러 | Backend CORS 설정 누락 | CorsConfig.java 확인 |
| 404 Not Found | API 경로 불일치 | Backend Controller의 @RequestMapping 확인 |
| 500 Internal Error | DB 연결 실패 | application-local.yml 비밀번호 확인 |
| JSON Parse Error | 요청 Body 형식 오류 | Content-Type: application/json 확인 |

---

## Step 11. 테스트 및 QA

> 담당: 전원

### 11-1. 기능 테스트 체크리스트

**인증**
```
[ ] 이메일 형식 검증이 동작하는가?
[ ] 비밀번호 6자 이상 검증이 동작하는가?
[ ] 중복 이메일 가입 시 에러 메시지가 뜨는가?
[ ] 로그인 후 닉네임이 표시되는가?
[ ] 로그아웃 후 localStorage가 비워지는가?
```

**수수료 조회**
```
[ ] 17개 시도가 모두 표시되는가?
[ ] 시도 선택 시 해당 시군구만 표시되는가?
[ ] 카테고리 필터가 동작하는가?
[ ] 키워드 검색이 동작하는가?
[ ] 수수료가 올바르게 표시되는가?
```

**배출 신청**
```
[ ] 비로그인 시 로그인 페이지로 이동하는가?
[ ] 신청번호가 자동 생성되는가?
[ ] 과거 날짜 선택이 막히는가?
[ ] 취소가 정상 동작하는가?
[ ] 마이페이지에서 신청 내역이 보이는가?
```

**재활용**
```
[ ] 물품 등록이 되는가?
[ ] 사진 첨부가 되는가? (최대 5장)
[ ] 상태 변경이 되는가?
[ ] 삭제가 되는가?
```

### 11-2. 반응형 테스트

브라우저 개발자 도구(F12)에서 모바일 모드로 전환하여 428px 너비에서 레이아웃이 깨지지 않는지 확인합니다.

---

## Step 12. 배포

> 담당: 최세진

### 12-1. Frontend 빌드

```bash
cd frontend
npm run build    # dist/ 폴더에 빌드 결과 생성
npm run preview  # 빌드 결과 미리보기
```

### 12-2. Backend 빌드

```bash
cd backend
./gradlew build  # build/libs/에 JAR 파일 생성
java -jar build/libs/backend-0.0.1-SNAPSHOT.jar  # 실행 테스트
```

### 12-3. 배포 환경 설정

| 항목 | 설정 |
|------|------|
| Frontend | Vercel, Netlify, 또는 Nginx |
| Backend | AWS EC2, GCP, 또는 NCP |
| Database | AWS RDS MySQL 또는 직접 MySQL 서버 |

### 12-4. 환경 변수 (운영)

```
# Frontend (.env.production)
VITE_API_BASE_URL=https://api.도메인.com
VITE_MAP_API_KEY=실제_카카오_API_키

# Backend (application-prod.yml)
spring.datasource.url=jdbc:mysql://운영DB주소:3306/waste_db
spring.datasource.username=운영계정
spring.datasource.password=운영비밀번호
spring.jpa.hibernate.ddl-auto=validate  # 운영에서는 validate!
spring.jpa.show-sql=false               # 운영에서는 끄기
```

---

## 부록: Git 브랜치 전략

```
main                        ← 배포용 (안정)
├── develop                 ← 개발 통합
│   ├── feature/user-auth   ← 최세진: 인증 기능
│   ├── feature/fee-lookup  ← 이재훈: 수수료 조회
│   ├── feature/disposal    ← 최세진: 배출 신청
│   ├── feature/recycle     ← 이재훈: 재활용
│   ├── feature/offline     ← 이재훈: 오프라인
│   ├── feature/ui-common   ← 최은아: 공통 컴포넌트
│   ├── feature/pages-core  ← 최가을: 핵심 페이지 (홈, 수수료, 배출 플로우)
│   ├── feature/pages-sub   ← 최은아: 인증 + 오프라인 페이지
│   └── feature/pages-myrecycle ← 강해원: 온라인 안내, 마이페이지, 재활용
└── hotfix/xxx              ← 긴급 수정
```

**Git 작업 흐름:**

```bash
# 1. develop에서 feature 브랜치 생성
git checkout develop
git pull
git checkout -b feature/user-auth

# 2. 작업 후 커밋
git add .
git commit -m "feat: 회원가입/로그인 API 구현"

# 3. develop에 병합 (Pull Request 사용 권장)
git push origin feature/user-auth
# GitHub에서 PR 생성 → 코드 리뷰 → Merge
```

---

## 부록: 커밋 메시지 규칙

```
feat: 새 기능 추가
fix: 버그 수정
style: 코드 스타일 변경 (동작 변화 없음)
refactor: 코드 리팩토링
docs: 문서 수정
chore: 빌드, 설정 변경
```

예시:
```
feat: 수수료 조회 API 구현
fix: 시군구 필터링 오류 수정
style: HomePage 레이아웃 간격 조정
docs: API 엔드포인트 문서 추가
```
