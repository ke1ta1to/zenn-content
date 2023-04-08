---
title: "PropsWithChildrenのススメ"
emoji: "🔖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react"]
published: true
---

# 知ってる？

PropsWithChildren の存在を知ってますか？
children を受け取るとき毎回 Props を書いていたら大変ですよね。

```tsx
type Props = {
  children: ReactNode;
};

function Component1({ children }: Props) {
  // ...
}
```

# PropsWithChildren を使うと

こんな感じにできます。

```tsx
import { PropsWithChildren } from "react";

function Component1({ children }: PropsWithChildren) {
  // ...
}
```

# 他の Props を受け取るときは

次のようにしてください

```tsx
import { PropsWithChildren } from "react";

type Props = {
  label: string;
};

function Component1({ children, label }: PropsWithChildren<Props>) {
  // ...
}
```

まあ中身がこんな感じなので当然ですね

```tsx
type PropsWithChildren<P = unknown> = P & { children?: ReactNode | undefined };
```

# 最後に

`React.FC`と`React.VFC`の良し悪し、`Arrow Functions`を使うかどうかなど、様々な議論が今も続いています。調べてみてはいかがでしょうか。
自分なりのベストプラクティスを見つけることをお勧めします。
私はこの記事の書き方を採用しています。
