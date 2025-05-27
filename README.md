
## 프론트엔드 배포 파이프라인

<p>
 <img src="https://velog.velcdn.com/images/chloeee/post/2f6769ba-5b39-4949-bb3b-753aad7b3564/image.png" alt="image" width="700px"/>
</p>


 GitHub Actions를 사용해 Next.js 애플리케이션을 Amazon S3에 자동으로 배포하고, CloudFront 캐시를 무효화하는 배포 파이프라인을 설명하고자 합니다.

GitHub Actions에 워크플로우`.github/workflows/deployment.yml`를 작성해 다음과 같이 배포가 진행되도록 했습니다.
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

### 📘 주요 개념
- GitHub Actions과 CI/CD 도구:<br/>
GitHub Actions는 코드 변경 시 자동으로 빌드, 테스트, 배포 과정을 수행하는 CI/CD(지속적 통합 및 배포) 도구입니다. 이를 통해 배포 과정을 자동화하고 안정적으로 관리할 수 있습니다.<br/>
`예시`: main 브랜치에 코드가 push되면, GitHub Actions가 자동으로 npm run build → S3 업로드 → CloudFront 캐시 무효화까지  배포 전 과정을 자동으로 수행합니다.

- S3와 스토리지:<br/>
Amazon S3는 정적 웹사이트 파일(HTML, CSS, JS 등)을 저장하고 호스팅할 수 있는 객체 스토리지 서비스입니다. 빠르고 안정적으로 정적 자산을 제공하는 데 적합합니다.<br/>
`예시`: out/ 디렉토리에 생성된 Next.js 정적 파일을 `hanghae-infra-task`라는 S3 버킷에 업로드하여 배포합니다.

- CloudFront와 CDN:<br/>
Amazon CloudFront는 S3에 저장된 정적 파일을 전 세계 엣지 서버에 캐싱하여 빠르게 전달하는 CDN(콘텐츠 전송 네트워크)입니다. 사용자와 가까운 서버에서 콘텐츠를 제공해 지연 시간을 줄입니다.<br/>
`예시`: 한국 사용자가 사이트에 접속하면, 한국에 위치한 엣지 서버에서 콘텐츠를 바로 제공받아 빠른 로딩이 가능합니다.


- 캐시 무효화(Cache Invalidation):<br/>
CloudFront는 성능 향상을 위해 파일을 캐시하지만, 변경된 파일이 즉시 반영되지 않을 수 있습니다. 캐시 무효화는 기존 캐시를 제거하고 최신 파일로 갱신되도록 하는 작업입니다.<br/>
`예시`: 새로 빌드한 파일이 배포된 후, `aws cloudfront create-invalidation --paths "/*"` 명령어로 기존 캐시를 무효화하여 사용자가 변경 사항을 즉시 확인할 수 있도록 합니다.

- Repository secret과 환경변수:<br/>
GitHub Actions에서 민감한 정보를 안전하게 사용하기 위해 Repository Secrets를 활용합니다. AWS 자격 증명, 배포 대상 버킷명 등은 외부에 노출되지 않도록 환경변수로 관리합니다.<br/>
`예시`: 워크플로우에서 ${{ secrets.AWS_ACCESS_KEY_ID }} 형태로 IAM 키를 불러와 CLI 인증을 수행합니다.

