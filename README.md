# Docker Image Optimization and Size Reduction: Tips & Tricks
> 클라우드 AWS 구축시 참고자료

## 👨‍💻Team

|<img src="https://avatars.githubusercontent.com/u/139302518?v=4" width="100" height="100"/>|<img src="https://avatars.githubusercontent.com/u/78792358?v=4" width="100" height="100"/>|
|:-:|:-:|
|곽병찬<br/>[@gato-46](https://github.com/gato-46)|박현우<br/>[@smartcow99](https://github.com/smartcow99)|

## 개요
- Oracle 11g는 복잡한 데이터베이스 시스템을 Docker 환경에서 실행할 때 특히 큰 용량을 차지합니다. 
- 이러한 이유로, Docker 이미지를 최적화하고 크기를 줄이는 방법을 익히는 것이 중요합니다.

## 목표🎁
- Oracle 11g를 사용하는 Docker 이미지를 최적화하여 디스크 공간 절약과 배포 속도를 향상시키는 것이 목표입니다.
- 이러한 최적화 기법을 통해 효율적인 리소스 사용과 향상된 성능, 그리고 보안 강화까지 가능하게 할 것입니다.

---

## 최적화 전략

- Oracle 11g Docker 이미지를 최적화하기 위한 여러 방법을 제시합니다.

### 1. Use Minimal Base Images
Oracle 11g는 이미지 크기가 크기 때문에 가벼운 베이스 이미지를 사용하는 것이 매우 중요합니다. 

예를 들어, oraclelinux:7-slim과 같은 경량 이미지를 사용하는 것이 도움이 됩니다. 

Alpine Linux는 사용하기 어렵기 때문에 Oracle Linux의 슬림 버전을 사용하는 것이 일반적입니다.

```bash
FROM oraclelinux:7-slim
```

### 2. Multi-Stage Builds
Oracle 11g는 설치 과정이 복잡하고 많은 리소스를 요구하므로 다단계 빌드가 유용합니다. 

빌드 단계에서 불필요한 패키지와 파일을 포함시키지 않도록 유의하세요.

```bash
# Install Oracle 11g in a separate stage
FROM oraclelinux:7-slim as build
WORKDIR /install


# Oracle 11g 설치 파일 복사 및 설치
COPY oracle_install_files /install/
RUN ./install_oracle.sh

# 최종 이미지로 필요한 부분만 복사
FROM oraclelinux:7-slim
COPY --from=build /oracle /oracle
```

### 3. Clean Up After Installations
Oracle 11g 설치 후 불필요한 파일 및 캐시를 제거하여 이미지 크기를 줄이세요. 

특히 yum 패키지를 설치할 때는 다음과 같은 방식으로 캐시를 청소하는 것이 좋습니다.

```bash
RUN yum install -y oracle-rdbms-server-11gR2-preinstall && \
    yum clean all && \
    rm -rf /var/cache/yum
```

### 4. Use .dockerignore
Oracle 설치 파일 외에도 개발 과정에서 발생하는 불필요한 파일을 Docker 이미지에 포함시키지 않도록 .dockerignore 파일을 활용하세요. 

아래는 예시입니다.

```bash
oracle_install_files/logs
oracle_install_files/temporary_files
```

### 5. Docker Image Pruning
Oracle과 같은 대형 이미지를 사용하다 보면 오래된 이미지나 컨테이너가 쌓여 디스크를 차지할 수 있습니다. 

정기적으로 docker system prune 명령어를 실행하여 사용하지 않는 리소스를 정리하세요.

### 6. Optimize Dockerfile Instructions
Dockerfile에서 Oracle을 설치할 때 불필요한 패키지를 제거하고, 설치 과정 중 캐시를 관리하여 레이어를 최적화하세요. 

Oracle 11g는 용량이 크기 때문에 단계별로 이미지 크기를 최적화하는 것이 중요합니다.

### 7. Security Scanning
Oracle 11g의 보안 취약점을 사전에 방지하기 위해 이미지 보안 스캔 도구를 활용하여 주기적으로 점검하세요. 

특히, Oracle의 보안 패치 릴리즈 주기를 참고하여 이미지를 업데이트하고 유지보수하는 것이 중요합니다.

### 8. Leverage Compression Tools
Oracle 11g 이미지의 크기를 줄이기 위해 Docker Slim과 같은 도구를 사용하여 이미지를 최적화할 수 있습니다.

```bash
docker-slim build --http-probe=false oracle11g_image
이 명령어는 이미지의 불필요한 파일을 제거하고, 이미지 크기를 줄이는 데 도움을 줍니다.
```

### 9. Implement Caching
Oracle 11g 설치는 시간이 오래 걸리므로 빌드 캐시를 효과적으로 사용하여 빌드 시간을 줄이세요. 

자주 변경되는 부분은 Dockerfile의 하단에 위치시키는 것이 캐시 효율을 높이는 데 도움이 됩니다.

### 10. Inspect Image Layers
docker history 명령어를 사용하여 Oracle 11g 이미지의 레이어를 분석하고, 불필요한 레이어를 줄일 수 있습니다. 

이를 통해 이미지 크기를 줄일 기회를 찾을 수 있습니다.

## Conclusion🎉
Oracle 11g와 같은 대형 애플리케이션을 Docker 환경에서 최적화하는 것은 중요한 과제입니다. 

위에 제시된 방법들을 통해 Docker 이미지를 최적화하고, 배포 속도를 개선하며, 비용 절감을 이룰 수 있습니다.


> Dockerfile🛠
```bash
# Install Oracle 11g in a separate stage
FROM oraclelinux:7-slim as build
WORKDIR /install

# Oracle 11g 설치 파일 복사 및 설치
COPY oracle_install_files /install/
RUN ./install_oracle.sh

# 최종 이미지로 필요한 부분만 복사
FROM oraclelinux:7-slim

# Oracle 11g 설치에 필요한 환경 설정
RUN yum install -y oracle-rdbms-server-11gR2-preinstall && \
    yum clean all && \
    rm -rf /var/cache/yum

# Oracle 설치 후 필요한 파일만 복사
COPY --from=build /oracle /oracle

# 불필요한 파일 제거
RUN rm -rf /install/logs /install/temporary_files

# Oracle 11g 이미지 최적화
CMD ["/oracle/start.sh"] # Oracle 시작 스크립트 경로를 정확히 설정

```

### 실행하기 전 확인해야 할 사항👀:
- oracle_install_files 경로에 설치 파일과 스크립트들이 올바르게 위치했는지.
- install_oracle.sh에서 Oracle 11g 설치가 제대로 처리되는지.
- Oracle 설치 후 /oracle에 필요한 파일들이 제대로 존재하는지.
---


> 참고자료 [Medium: Docker Image optimization tips & tricks](https://overcast.blog/docker-image-optimization-tips-tricks-6a17f687162b), [Medium: Reduce the size of the Docker Image](https://faun.pub/reduce-the-size-of-the-docker-image-e6895b653419)
