# Docker Image Optimization and Size Reduction: Tips & Tricks

## 👨‍💻Team

|<img src="https://avatars.githubusercontent.com/u/139302518?v=4" width="100" height="100"/>|<img src="https://avatars.githubusercontent.com/u/78792358?v=4" width="100" height="100"/>|
|:-:|:-:|
|곽병찬<br/>[@gato-46](https://github.com/gato-46)|박현우<br/>[@smartcow99](https://github.com/smartcow99)|

## 개요

Docker를 사용하는 가장 **큰 이유**는 가상머신에 비해서 **작은 크기를 가지는 것**이라 생각합니다.

이에, Docker 이미지를 **최적화하고 크기를 줄이는 것**이 굉장히 중요하다는 생각이 들어 학습을 시작하게 되었습니다.

## 목표🎁

이 문서의 목표는 *Docker 이미지를 최적화하고 크기를 줄이는 다양한 기법*을 배우는 것입니다.  
이러한 최적화 기법을 통해 리소스를 효율적으로 활용하고 배포 속도를 향상시키며 보안을 강화할 수 있습니다.  
이를 통해 더 나은 개발 환경을 만들고 서비스 제공 시 비용을 절감하는 데 기여할 수 있는것을 목표로 합니다.

## 

Docker 이미지를 최적화하고 크기를 줄이기 위해 여러 기법과 모범 사례를 구현할 수 있습니다. 아래는 이미지 최적화 및 크기 축소를 위한 주요 전략들입니다.

### 1. Use Minimal Base Images

Docker 이미지를 작성할 때 최소한의 베이스 이미지를 선택하는 것이 매우 중요합니다. 일반적으로 사용되는 베이스 이미지인 Ubuntu는 약 128MB이지만, **Alpine Linux**를 사용하면 약 5MB로 크게 줄일 수 있습니다.

더 나아가 **Distroless** 이미지를 사용하는 것도 고려해볼 수 있습니다. Distroless 이미지는 애플리케이션과 그 런타임 종속성만 포함되며, 불필요한 패키지를 포함하지 않아 보안성을 향상시킬 수 있습니다. 이는 알려진 취약점(CVEs)의 수를 줄이는 데 기여합니다.

| TAG | SIZE |
| --- | --- |
| debian | 124.0MB |
| alpine | 5.0MB |
| gcr.io/distroless/static-debian11 | 2.51MB |

예시:

```bash
gcr.io/distroless/static-debian11    2.51MB

```

### 2. Minimize Layers

Dockerfile에서 레이어 수를 줄이는 것은 이미지 크기를 줄이는 또 다른 중요한 방법입니다. Docker는 각 명령어마다 새로운 레이어를 생성하므로, 여러 `RUN` 명령어를 하나의 명령어로 결합하여 레이어 수를 줄일 수 있습니다. 예를 들어, 다음과 같이 작성할 수 있습니다.

**Before:**

```bash
FROM golang:1.18 as build
RUN go mod download
RUN go mod verify
```

**After:**

```bash
FROM golang:1.18
RUN go mod download && go mod verify
```

이렇게 하면 레이어 수가 줄어들어 이미지 크기도 감소합니다.

### 3. Multi-Stage Builds

다단계 빌드를 활용하면 빌드 환경과 최종 이미지를 분리할 수 있습니다. 첫 번째 단계에서 애플리케이션을 빌드하고, 두 번째 단계에서 필요한 아티팩트만 복사하여 최종 이미지를 생성합니다. 이렇게 하면 최종 이미지에는 불필요한 파일이 포함되지 않습니다.

```
# Start by building the application.
FROM golang:1.20-alpine AS build

WORKDIR /build
COPY go.mod go.sum ./
RUN go mod download && go mod verify
COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o run .

# Now copy it into our distroless image.
FROM gcr.io/distroless/static-debian11
WORKDIR /app
COPY --from=build /build/run .
CMD ["/app/run"]
```

### 4. Use .dockerignore

`.dockerignore` 파일을 생성하여 Docker 이미지 빌드 시 불필요한 파일과 디렉터리를 제외합니다. 이 파일은 `git` 저장소의 파일, 개발 도구 및 기타 임시 파일을 포함할 수 있으며, 이를 통해 이미지 크기를 최소화할 수 있습니다. 예를 들어, 다음과 같은 내용을 포함할 수 있습니다.

```bash
node_modules
Dockerfile*
.git
.github
.gitignore
dist/**
README.md
```

### 5. Leverage Compression Tools

Docker Slim과 같은 도구를 사용하여 Docker 이미지를 압축할 수 있습니다. 이러한 도구는 이미지를 분석하고 필요 없는 파일과 종속성을 제거하여 이미지 크기를 효과적으로 줄입니다. 다음과 같은 명령어로 Docker Slim을 사용할 수 있습니다.

```bash
docker-slim build --sensor-ipc-mode proxy --sensor-ipc-endpoint 172.17.0.1 --http-probe=false nginx
```

### 6. Security Scanning

Docker 이미지는 주기적으로 보안 스캔을 통해 취약점을 점검해야 합니다. 여러 도구를 사용하여 이미지 내 보안 취약점을 식별하고, 정기적으로 업데이트와 패치를 적용하여 보안을 강화할 수 있습니다. 보안성을 높이는 것은 장기적인 안정성을 확보하는 데 매우 중요합니다.

### 7. Clean Up After Installations

패키지 설치 후에는 임시 파일과 캐시를 제거하여 이미지 크기를 줄이는 것이 좋습니다. 예를 들어, `apt-get`을 사용할 때는 다음과 같이 캐시를 청소할 수 있습니다.

```
RUN apt-get update && \
    apt-get install -y git && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 8. Single Responsibility Principle

각 Docker 이미지는 단일 책임 원칙을 따라야 합니다. 여러 서비스를 하나의 이미지에 포함시키기보다는 각 서비스마다 별도의 이미지를 생성하고, 이를 Docker Compose나 Kubernetes로 조합하여 사용하는 것이 좋습니다.

### 9. Optimize Dockerfile Instructions

패키지 설치 시 특정 버전을 사용하고, 종속성을 최소화하며, 불필요한 패키지를 설치한 후에는 제거하여 이미지 크기를 줄이는 것이 중요합니다.

### 10. Inspect Image Layers

`docker history` 및 `docker inspect`와 같은 도구를 사용하여 이미지 레이어를 분석하고 최적화 기회를 식별합니다. 불필요한 파일과 명령을 Dockerfile에서 제거하여 레이어 크기를 줄일 수 있습니다.

### 11. Docker Image Pruning

사용하지 않는 Docker 이미지, 컨테이너, 볼륨 및 네트워크를 정기적으로 정리하여 디스크 공간을 회수하고 성능을 개선합니다. `docker system prune` 명령어를 사용하면 됩니다.

### 12. Implement Caching

Docker의 빌드 캐시를 활용하여 Dockerfile을 구성합니다. 자주 변경되는 명령어는 Dockerfile의 끝 부분에 배치하여 캐시 무효화를 최소화합니다.

### 13. Compress Artifacts

애플리케이션이 생성하는 빌드 아티팩트를 압축하여 Docker 이미지에 복사하기 전에 크기를 줄입니다. 이는 이미지 크기를 줄이고 빌드 속도를 개선하는 데 도움이 됩니다.

### 14. Use Smaller Alternatives

가능한 경우 도구와 라이브러리의 더 작은 대안을 사용하여 이미지 크기를 줄입니다. 예를 들어, BusyBox나 Microcontainers를 고려할 수 있습니다.

### 15. Use Docker Squash

Docker Squash를 사용하여 이미지의 크기를 줄일 수 있지만, 빌드 시간을 증가시키고 캐시 가능성을 줄일 수 있으므로 주의해야 합니다.

## Conclusion🎉

이러한 전략을 통해 Docker 이미지를 최적화하고 크기를 줄이면 배포 속도가 개선되고 리소스 사용이 감소하며 보안이 강화됩니다.  
Docker 이미지는 단순한 도구가 아닌, 효율적인 개발 및 배포를 위한 중요한 요소로서 좋은 방법이 있다면 지속해서 학습할 에정입니다.
