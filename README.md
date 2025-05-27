
## 프론트엔드 배포 파이프라인

<p>
 <img src="https://velog.velcdn.com/images/chloeee/post/2f6769ba-5b39-4949-bb3b-753aad7b3564/image.png" alt="image" width="700px"/>
</p>


 GitHub Actions를 사용해 Next.js 애플리케이션을 Amazon S3에 자동으로 배포하고, CloudFront 캐시를 무효화하는 배포 파이프라인을 설명하고자 합니다.

GitHub Actions에 워크플로우를 작성해 다음과 같이 배포가 진행되도록 했습니다.
1. 레포지토리 소스를 현재 워크플로우 환경으로 가져옵니다.
2. `npm ci` 명령어를 통해 프로젝트 의존성을 설치합니다.
   - CI/CD 환경에서 안정성을 확보하기 위해 npm install 대신 `npm ci` 사용
3. `npm run build` 명령어로 Next.js 프로젝트를 빌드합니다.
   -  next export 결과물이 out/ 폴더에 생성됩니다.
4. GitHub Secrets에 등록된 자격 증명을 바탕으로 AWS 인증을 구성합니다.
   - GitHub Secrets에 저장된 AWS IAM 키로 인증합니다.
   - 이후 AWS CLI 명령어를 사용 가능하도록 설정합니다.
5. 빌드된 정적 파일을 aws s3 sync 명령어로 S3 버킷에 업로드(동기화)합니다.
   - out/ 디렉토리의 정적 파일을 S3 버킷에 동기화합니다.
   - `--delete 옵션`을 통해 S3에서 제거된 파일도 삭제되어 항상 최신 상태 유지됩니다.
6. aws cloudfront create-invalidation 명령어를 통해 `CloudFront 캐시를 무효화`하여 최신 파일이 사용자에게 제공되도록 합니다.
   - CloudFront는 성능 향상을 위해 정적 파일을 캐시합니다.
   - 배포 후에도 사용자 브라우저에서 최신 파일을 보도록 하려면 캐시 무효화가 필요합니다.
   - /* 경로를 무효화하여 전체 캐시를 갱신합니다.

### 📦 주요 링크
S3 버킷 웹사이트 엔드포인트:  http://hanghae-infra-task.s3-website-ap-southeast-2.amazonaws.com/

CloudFront 배포 도메인 이름: https://dcrpqw4gqr0ey.cloudfront.net/

### 📚 주요 개념
개념	설명
GitHub Actions와 CI/CD 도구	코드가 main 브랜치에 푸시될 때 자동으로 빌드·배포 과정을 실행하는 CI/CD 플랫폼
S3와 스토리지	AWS의 정적 파일 저장 서비스로, 빌드 결과물을 호스팅하는 데 사용됨
CloudFront와 CDN	S3에 저장된 정적 콘텐츠를 전 세계에 빠르게 배포해주는 CDN 서비스
캐시 무효화 (Cache Invalidation)	CloudFront가 이전에 캐시해둔 파일을 제거하고, 최신 버전의 파일로 교체되도록 하는 작업
Repository Secrets와 환경변수	AWS 인증 정보(AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, 등)를 안전하게 GitHub에 저장하여 워크플로우에서 사용

