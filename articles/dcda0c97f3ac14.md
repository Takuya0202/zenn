---
title: "Supabase ✖️ Prisma ✖️ Next.jsでチーム開発してみた"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## prismaとsupabaseを連携する
以下を実行
```
npm install prisma --save-dev
npm install @prisma/client
npx prisma init
```

```.env
DATABASE_URL=
DIRECT_URL=
```


```schema.prisma
generator client {
  provider = "prisma-client-js"
  // output   = "../app/generated/prisma" いらないらしい
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// テーブルを以下に書く
model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  userIcon  String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```