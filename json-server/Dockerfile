# json-server db.json 파일 분리

FROM node:16-alpine

# 도커 컨테이너에 설치 될 경로 // 명령어가 실행 되는 경로
WORKDIR /db

# 이미지를 빌드 단계에서 실행 될 커맨드
RUN yarn install

COPY db.json ./
COPY package*.json ./

# 포트 노출 4000번 포트를 사용 하겠다는 의미
# EXPOSE 4000

# 컨테이너가 시작 될때 실행되는 커맨드
CMD ["yarn","server"]