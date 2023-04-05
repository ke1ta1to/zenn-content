---
title: "Zenn API Types"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["zenn", "typescript"]
published: true
---

# Types ファイル

[https://zenn.dev/api/articles](https://zenn.dev/api/articles)から取得できる JSON を元に作成したものです。
ミスなどがあればコメントください

```typescript
export type Article = {
  id: number;
  post_type: "Article";
  title: string;
  slug: string;
  published: boolean;
  comments_count: number;
  liked_count: number;
  body_letters_count: number;
  article_type: "tech" | "idea";
  emoji: string;
  is_suspending_private: boolean;
  published_at: string;
  body_updated_at: string;
  source_repo_updated_at: string;
  path: string;
  user: User;
  publication: Publication | null;
};

export type User = {
  id: number;
  username: string;
  name: string;
  avatar_small_url: string;
};

export type Publication = {
  id: number;
  name: string;
  avatar_small_url: string;
  display_name: string;
  beta_stats: boolean;
  avatar_registered: boolean;
};
```
