---
title: "flexではなく、gridでつくるレイアウト設計"
emoji: "🏁"
type: "tech"
topics: ["css", "frontend", "html", "design"]
published: true
---

実務では flex でのレイアウトをよく使うのですが、ちょいちょい AI と相談していると **CSS Grid** を使う場面も出てきたのでこの際記事にして理解を深めようと思います。

---

## 1. Flexbox vs CSS Grid：使い分けの基準

一言でいうと、**「線か、面か」**です。

- **Flexbox（1 次元）：** 要素を横（または縦）に並べるのが得意。ナビゲーションメニューや、ボタンの並びなど。
- **CSS Grid（2 次元）：** 縦と横の「グリッド（格子）」を定義し、そこに要素をはめ込んでいくのが得意。ページ全体の構成や、複雑なカードレイアウトなど。

ディレクター視点で嬉しいのは、Grid を使うと **「HTML の記述順を気にせず、CSS だけで配置をガラッと変えられる」** 点です。

---

## 2. Grid の基本：たった 2 ステップで「面」を作る

まずは、親要素に「どんな格子にするか」を指示するだけです。

### 2-1. グリッドの形を決める

```css
:root {
  --sys-space-padding-main: 8px;
}

.container {
  display: grid;
  /* 1：1：1 の比率で3つの列を作る */
  grid-template-columns: repeat(3, 1fr);
  gap: var(--sys-space-padding-main);
  /* 要素の高さを設定 */
  grid-auto-rows: minmax(100px, 1fr);
}
```

### 2-2. 要素を配置する

Flexbox のように nth-child で余白を引いたりする必要はありません。Grid が自動的に等間隔に配置してくれます。

## 3. レイアウトに名前がつけられるので汎用的に使える

grid はレイアウトに 「名前」 をつけて管理できることです。  
CSS で `grid-template-areas` を使って、各要素をどのように配置するか「設計図」を描きます。  
それぞれの要素には `grid-area` プロパティで名前を割り当てます。

```css
.container {
  display: grid;
  /* レイアウトの設計図を名前で定義！ */
  grid-template-areas:
    "header  header  header"
    "nav     main   sidebar"
    "footer  footer footer";

  /* 各列の幅（左は200px固定、右は残りの幅） */
  grid-template-columns: 200px 1fr;
  /* 各行の高さ指定、１行ずつの設定、autoはコンテンツにあわせて、1frは可変 */
  grid-template-rows: auto 1fr auto;

  gap: var(--sys-space-padding-main);
  padding: var(--sys-space-padding-main);
}

/* 各要素にグリッドエリア名を割り当て、スタイルを適用 */
.header {
  grid-area: header;
  background-color: var(--primary-color);
  color: white;
  padding: 20px;
  text-align: center;
}
.nav {
  grid-area: nav;
  background-color: #e9ecef;
  padding: 20px;
}
.main {
  grid-area: main;
  background-color: #ffffff;
  padding: 20px;
}
.sidebar {
  grid-area: sidebar;
  background-color: #e9ecef;
  padding: 20px;
}
.footer {
  grid-area: footer;
  background-color: var(--secondary-color);
  color: white;
  padding: 20px;
  text-align: center;
}
```

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-lubzetoy?embed=1&file=index.html)

#### レスポンシブ対応もシンプル

画面幅が 768px 以下になると、`grid-template-areas` の定義を書き換えるだけで、各要素が縦一列に積み重なるレイアウトに瞬時に切り替わります。

## まとめ：適材適所でレイアウトを操る

今まで全体のレイアウトから flex を使っていましたが、grid も使うことで

- 小さなパーツの並びや、要素の均等配置には **Flexbox**
- ページ全体の枠組みや、複雑なカード一覧、HTML 順に縛られない配置には **CSS Grid**

この「使い分け」ができるようになると、よりコードがシンプルになりそうです。

例えば flex を使っていた場合は wrapper で囲む必要があったものが

```html
<div class="container">
  <header>Header</header>
  <div class="flex-wrapper">
    <nav>Navigation</nav>
    <main>Main Content</main>
    <aside>Sidebar</aside>
  </div>
  <footer>Footer</footer>
</div>
```

grid を使うことで、wrapper を使う必要がなくなります。

```html
<div class="container">
  <header>Header</header>
  <nav>Navigation</nav>
  <main>Main Content</main>
  <aside>Sidebar</aside>
  <footer>Footer</footer>
</div>
```

```css
.container {
  display: grid;
  /* レイアウトの設計図を名前で描く */
  grid-template-areas:
    "header  header  header"
    "nav     main    sidebar"
    "footer  footer  footer";

  grid-template-columns: 200px 1fr 250px; /* 各列の幅 */
  gap: 20px; /* 全体の余白を一括指定 */
  grid-template-rows: auto auto 1fr auto auto;
}

/* 各要素に名前を付けるだけ */
header {
  grid-area: header;
}
nav {
  grid-area: nav;
}
main {
  grid-area: main;
}
aside {
  grid-area: sidebar;
}
footer {
  grid-area: footer;
}
```
