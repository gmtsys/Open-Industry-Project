# GOMOTEC 배포 (Cloud Run)

이 가지(`gomotec/main`)는 [Open Industry Project](https://github.com/Open-Industry-Project/Open-Industry-Project)(MIT)를
GOMOTEC 그룹웨어(포털)에 붙인 판이다. 포털 왼쪽 메뉴 «시뮬레이터 › 라인 시뮬레이터»가 새 탭으로 이 서비스를 연다.

| 무엇 | 어디 |
|---|---|
| 이미지 | `deploy/gcp/Dockerfile` — 원본 `npm run build` + nginx |
| 응답 머리 | `deploy/gcp/nginx.conf` — 원본 `public/_headers` · `_redirects` 를 옮김 + 바깥 연결 제한 |
| 빌드 | `deploy/gcp/cloudbuild.yaml` |
| 서비스 · IAP | 포털 저장소 `infra/sim.tf` (`portal-sim-<env>`, 회사 계정만) |

## 바깥 연결

`connect-src` 로 브라우저가 연결할 수 있는 곳을 이 서버와 원본 부품 라이브러리(받아 오기만)로 묶었다.
그래서 원본 회사의 같이 편집 서버와 GitHub 게시는 막힌다(«Share» 를 누르면 방이 열리지 않는다).
같이 편집이 필요하면 우리 쪽에 Hocuspocus 서버(`server/`)를 띄우고 `VITE_COLLAB_URL` 과 `connect-src` 에 그 주소를 더한다.

## 배포

```sh
# 커밋한 판을 빌드 폴더로 풀고(작업 중인 파일 · node_modules 가 섞이지 않게)
git archive <커밋> | tar -x -C <빌드 폴더>
gcloud builds submit <빌드 폴더> --config <빌드 폴더>/deploy/gcp/cloudbuild.yaml \
  --substitutions="_AR=asia-northeast3-docker.pkg.dev/gmt-cloud/gw-repo,_SHA=<커밋>" --project=gmt-cloud
# 처음 한 번은 포털 infra/sim.tf(README §5 «라인 시뮬레이터»), 그 뒤로는
gcloud run services update portal-sim-dev --image=asia-northeast3-docker.pkg.dev/gmt-cloud/gw-repo/portal-sim:<커밋> \
  --region=asia-northeast3 --project=gmt-cloud
```

## 원본 따라가기

```sh
git remote add upstream https://github.com/Open-Industry-Project/Open-Industry-Project.git   # 한 번
git fetch upstream && git merge upstream/main
```

원본은 초기 버전이라 저장 형식이 바뀔 수 있다. 원본 `public/_headers` 가 바뀌었으면 `nginx.conf` 도 맞춘다.
