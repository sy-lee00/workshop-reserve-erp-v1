# 1단계: 프론트엔드(React) 빌드 환경
FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend

# 💡 현재 위치(e-commerce) 기준에서 소문자 frontend 폴더를 정확히 지목합니다.
COPY ./frontend/package*.json ./
RUN npm ci

COPY ./frontend/ ./
RUN npm run build

# 2단계: 백엔드(Java Gradle) 빌드 환경
FROM gradle:8-jdk21 AS backend-build
WORKDIR /app/backend

# e-commerce 폴더 내부의 모든 프로젝트 파일 복사
COPY . .

# 1단계에서 빌드된 React 결과물(build)을 스프링 부트 정적 자원 폴더(static)로 복사
COPY --from=frontend-build /app/frontend/build ./backend/src/main/resources/static/

# Gradle 빌드 수행을 위해 backend 폴더 안으로 진입 후 빌드
WORKDIR /app/backend/backend
RUN ./gradlew build -x test --no-daemon

# 3단계: 최종 실행 환경 (안정적인 런타임 이미지)
FROM eclipse-temurin:21-jre-jammy
EXPOSE 8080

# 빌드된 jar 파일을 최종 실행 이미지로 복사
COPY --from=backend-build /app/backend/backend/build/libs/*-SNAPSHOT.jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]