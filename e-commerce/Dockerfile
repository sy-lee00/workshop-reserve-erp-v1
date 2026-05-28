# 1단계: 프론트엔드(React) 빌드 환경
FROM node:20-alpine AS frontend-build
WORKDIR /app/frontend

COPY ./frontend/package*.json ./
RUN npm ci

COPY ./frontend/ ./
RUN npm run build

# 2단계: 백엔드(Java Gradle) 빌드 환경
FROM gradle:8-jdk21 AS backend-build
WORKDIR /app/backend

# e-commerce 안의 모든 것 복사
COPY . .

# 빌드된 React 결과물을 스프링 부트 정적 자원 폴더로 복사
COPY --from=frontend-build /app/frontend/build ./backend/dia/src/main/resources/static/

# 💡 조치 1: 리눅스 환경에서 gradlew 실행이 가능하도록 권한 부여 추가
WORKDIR /app/backend/backend/dia
RUN chmod +x ./gradlew

# Gradle 빌드 수행 (테스트 제외)
RUN ./gradlew build -x test --no-daemon

# 3단계: 최종 실행 환경
FROM eclipse-temurin:21-jre-jammy
EXPOSE 8080

# 💡 조치 2: plain.jar를 제외하고 오직 진짜 실행 가능한 뚱뚱한(Fat) jar 파일만 타겟팅하여 복사
COPY --from=backend-build /app/backend/backend/dia/build/libs/*[!plain].jar app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]