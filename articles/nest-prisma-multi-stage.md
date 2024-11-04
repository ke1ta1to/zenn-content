---
title: "【NestJS】Prisma併用時にマルチステージビルドで最適化を考える"
emoji: "⛳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nestjs, prisma, docker]
published: false
---

## はじめに

## 対象読者

## 結論

```dockerfile:Dockerfile
FROM node:20 AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npx prisma generate && npm run build

FROM node:20 AS deps
WORKDIR /app
COPY package*.json .
RUN npm install --omit=dev

FROM node:20
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/node_modules/.prisma ./node_modules/.prisma
CMD ["node", "dist/main.js"]
```

## やること

### nestjs

- バンドル？しない方がいい
  https://stackoverflow.com/questions/59635276/how-to-correctly-build-nestjs-app-for-production-with-node-modules-dependencies
  https://github.com/ZenSoftware/bundled-nest
  ncc

https://www.prisma.io/docs/orm/prisma-client/setup-and-configuration/generating-prisma-client

## まとめ
