### 개요
- `Github Actions`를 활용한 `AWS S3 CI` 테스트 레포지토리입니다.

### AWS S3 Settings

- `**AWS` → `S3` → `버킷 만들기` → `버킷 속성` → `정적 웹 사이트 호스팅`
    - **정적 웹 사이트 호스팅** : 활성화
    - **호스팅 유형** : 정적 웹 사이트 호스팅
    - **인덱스 문서** : `index.html`
    
- **버킷 정책 변경**
    - `S3`에 올라간 `React 정적 파일`을 웹에서 액세스 할 수 있게 버킷 정책을 변경해주고 추가
    - `권한` → `퍼블릭 액세스 차단(버킷 설정) : 비활성화` → `버킷 정책 편집` → `Bucket-Name 변경`

```jsx
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::{Bucket-Name}/*"
            ]
        }
    ]
}
```

- **AWS 리전(Region)** : AWS 인프라를 지리적으로 나누어 배포한 것을 의미합니다.
    - 사용자와 리전이 가까울수록 네트워크 지연을 최소화할 수 있습니다
    - 대한민국 리전 : `아시아 태평양(서울) ap-northeast-2`
    

### **IAM(Identity and Access Management)**

- `AWS` → `IAM` → `사용자` → `AmazonS3FullAccess : check`
- AWS 리소스에 대한 액세스를 안전하게 제어할 수 있는 웹 서비스입니다
- IAM을 사용하여 리소스를 사용하도록 인증 및 권한 부여된 대상을 제어

### GitHub Action Settings

![image](https://user-images.githubusercontent.com/106054507/191897690-d15242f1-c17d-4a35-8e66-42204ea61a44.png)

```jsx
name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x, 16.x, 18.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test

    - uses: awact/s3-action@master
      with:
        args: --acl public-read --follow-symlinks --delete
      env:
        SOURCE_DIR: './public'
        AWS_REGION: 'us-east-1'
        AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

- `name` : `work-flow`의 이름
- `on.push.branch : [main]` : `main`에 `source code`를 `push` 를 하면 `jobs` 커맨드 실행.
- `jobs.build : run-on: ubuntu-latest` : `ubuntu`에서 실행을 의미
- `strategy.matrix.node-version` : 버전 정보
- `run` : 어떻게 동작할 것인지를 의미
    - `--if present` : 빌드 스크립트가 있을 때만 실행
- `env.SOURCE_DIR` : 배포할 소스 파일 위치
- `AWS_REGION` : 설정한 AWS 리전 ( 한국은 `ap-northeast-2` )
- `AWS_S3_BUCKET`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` 등은 `Settings → Secrets → Actions → new repository secret` 설정 권장

### 참고문서
- [Inflearn 강의](https://www.inflearn.com/course/%EB%94%B0%EB%9D%BC%ED%95%98%EB%8A%94-%EB%A6%AC%EC%95%A1%ED%8A%B8/)
- [S3 버캣 정책 변경 - 아마존 링크](https://docs.aws.amazon.com/AmazonS3/latest/userguide/WebsiteAccessPermissionsReqd.html)
- [GitHub Actions .yml 파일 아마존 연결 방법](https://github.com/awact/s3-action)
