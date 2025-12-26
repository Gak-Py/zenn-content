---
title: "SCSS変数があるのになぜCSS Variables？ 実務で使ってわかった決定的な使い分け"
emoji: "🎨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["css", "scss", "frontend", "designsystem"]
published: true
published_at: 2025-12-26 20:00
---

## 1. SCSS 変数 vs CSS Variables：決定的な違い

| 特徴                 | SCSS 変数 (`$var`)   | CSS Variables (`--var`)      |
| :------------------- | :------------------- | :--------------------------- |
| **評価タイミング**   | コンパイル時（静的） | ブラウザ実行時（動的）       |
| **JS からの操作**    | 不可                 | 可能                         |
| **継承（スコープ）** | 定義順に従う         | DOM 構造に従って継承される   |
| **主な用途**         | 開発効率化・計算     | **テーマ切り替え・状態変化** |

特に CSS Variables が使い勝手いいと思ったのは、テーマの切り替えができる点です。

SCSS だと変数は設定後、変更はできない。  
CSS だと変更ができるので、CSS Variables なら、`body`に`.dark-mode`クラスをつけるだけで、中身の変数を一括で上書きできます。SCSS 変数だけでは、全スタイルの色違いをもう一度出力し直す必要があり、運用コストが跳ね上がります。

---

CSS Variables の基本構文

定義する際は、慣習として `:root`（ドキュメントの最上位）に記述し、どこからでも参照できるようにします。

```css
/* 通常（ライト）モード */
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --accent-color: #007aff;
}

/* ダークモード用のクラスがhtmlまたはbodyについた時 */
/* 変数の中身だけを「上書き」する */
.dark-mode {
  --bg-color: #1a1a1a;
  --text-color: #f5f5f5;
  --accent-color: #0a84ff;
}

/* 実際のスタイルでは変数を使う */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
  transition: background-color 0.3s, color 0.3s; /* 滑らかに変える */
}

.button {
  background-color: var(--accent-color);
  color: #fff; /* アクセント上の文字は固定でもOK */
}
```

変数の名前を「具体的な色名（例：--blue）」にするのではなく、「役割（例：--bg-color）」にすることで、設計の柔軟性が上がります。

## 4. JavaScript で変数の値が変更できる

これはかなり便利。

```javascript
// JavaScriptからボタン一つでテーマを切り替える例
const toggleTheme = () => {
  const root = document.documentElement;
  const current = getComputedStyle(root)
    .getPropertyValue("--primary-color")
    .trim();

  if (current === "#007aff") {
    root.style.setProperty("--primary-color", "#ff3b30"); // 赤に変更
  } else {
    root.style.setProperty("--primary-color", "#007aff"); // 青に戻す
  }
};
```

## Q. 気づいたらグレーが 10 種類…デザイン破綻をどう防ぐ？

よく実務で出てくるのが、「デザイナーが作った Figma に似たような色味が溢れている」問題。
役割の名前でそのまま変数にすると破綻してしまっていたので、自分の場合は`--grayFD`, `--grayEE`など先端 2 文字のカラーコードで増やしていたけど、そこはデザイナーさんと調整していくしかなさそう。

Figma 上でデザイナーさんにコンポーネント集に加え、カラーパレットも追加してもらってデザイナーさん自身にもどれだけ微妙な色使いわけてるのかを認識してもらうのもよさそうです。

でも制作しているうちにどうしても増えてしまうところはあるので、
そういう場合は tailwindcss とかで設定しているような感じで色をまとめてもらった後に不変の値としてまず設定し、それから Variables を設定すると少しはいい感じになるかも。

### 階層 1：Primitive Tokens（不変の値）

最初に tailwindcss 風に定義。  
あまりにも多い場合はデザイナーさんと要相談。  
本当にこの色でわけたいのか、すでに使ってる同じような色でもいいのか。

```scss
:root {
  --primitive-gray-100: #f5f5f5;
  --primitive-gray-200: #eeeeee;
  --primitive-blue-500: #007aff;
}
```

### 階層 2：Semantic Tokens（役割・意味）

定義したあとに css 変数を当てはめます。

```scss
:root {
  --color-bg-base: var(--primitive-gray-100);
  --color-text-main: var(--primitive-gray-200);
  --color-accent: var(--primitive-blue-500);
}
```

## まとめ：CSS Variables は便利。でも、新たな悩みも…

今回、CSS Variables を学んでみて、SCSS 変数にはなかった「ブラウザ上で動く」という圧倒的な便利さを知ることができました。

- **ダークモード対応が驚くほど簡単になる**
- **JavaScript と CSS の役割分担が綺麗になる**

これだけでも導入する価値は十分にあると感じています。
次回からは CSS Variables は使いつつ、mixin とか CSS Variables ではやりにくいところは引き続き SCSS で設定していこうと思います。
