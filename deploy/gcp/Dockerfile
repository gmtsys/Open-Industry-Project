# deploy/gcp/Dockerfile — GOMOTEC 포털용 시뮬레이터 이미지(Cloud Run portal-sim-<env>).
#
# 빌드 컨텍스트는 저장소 루트다:
#   docker build -f deploy/gcp/Dockerfile -t portal-sim .
# Cloud Build 는 deploy/gcp/cloudbuild.yaml. 서비스 · IAP(회사 계정만)는 포털 저장소의 infra/sim.tf.
#
# ① build — 원본과 같은 `npm run build`(모델 검사 → tsc → vite → 에셋 최적화). Node 24 는 원본 package.json engines.
# ② runtime — nginx 정적 서빙. 응답 머리(COOP · COEP 등)는 deploy/gcp/nginx.conf 가 원본 public/_headers 를 옮긴 것.

FROM node:24-bookworm-slim AS build
WORKDIR /src
# 의존부터 — 소스만 바뀐 빌드는 이 층을 다시 쓴다
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build

FROM nginx:1.27-alpine AS runtime
COPY deploy/gcp/nginx.conf /etc/nginx/nginx.conf
RUN rm -rf /usr/share/nginx/html/*
COPY --from=build /src/dist /usr/share/nginx/html
# 원본 호스트(Cloudflare Pages) 설정 파일은 nginx 에서 쓰지 않는다 — 그대로 두면 주소로 읽힌다.
# `nginx -t` 는 root 로 pid · 임시 경로를 실제로 만든다 — 그 «뒤» 에 /var/cache/nginx 를 nginx 계정에 넘겨야 실행 계정이 쓴다.
RUN set -eu; \
    rm -f /usr/share/nginx/html/_headers /usr/share/nginx/html/_redirects; \
    nginx -t; \
    chown -R nginx:nginx /var/cache/nginx; \
    if find /var/cache/nginx ! -user nginx | grep -q .; then \
      echo "/var/cache/nginx 에 nginx 소유가 아닌 항목이 있다 — 실행 계정(uid 101)이 못 쓴다" >&2; \
      exit 1; \
    fi

USER nginx
EXPOSE 8080
ENTRYPOINT ["nginx", "-g", "daemon off;"]
