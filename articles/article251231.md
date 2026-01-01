---
title: "Reactでは必須。マスターすべきモダンJS必須5選"
emoji: "⚛️"
type: "tech"
topics: ["javascript", "es6", "react", "frontend"]
published: true
---

JavaScript の基本を学んだ後、React などのフレームワークに挑戦すると「何この書き方？」と戸惑うことがよくあります。
実は、React のコードの多くは、最新の JavaScript（ES6+）の便利な構文で成り立っています。

頻出しつつ初見ではわかりにくい、**「安全に、美しくデータを扱う」ための 5 つの必須テクニック**をまとめました。

---

## 1. データの安全を守る「?.」と「??」

API から届くデータが欠けていたり、読み込み中だったりしてもサイトを落とさないための「防御力」です。
検証画面のコンソールエラー対策にも。

### オプショナルチェイニング（?.）

深い階層のデータにアクセスする際、途中で `null` や `undefined` があってもエラーを投げずに `undefined` を返してくれます。

```javascript
// user.profile がなくてもエラーにならない
const bio = user?.profile?.bio;

// 使わないとこういう形になり、冗長になる
let bio;
if (user && user.profile) {
  bio = user.profile.bio;
}
```

### Null 合体演算子（??）

null または undefined の時だけデフォルト値を適用します。|| と違い、数値の 0 や空文字を正しく維持できます。

```javascript
const score = userScore ?? 100; // scoreが0点なら、ちゃんと0を表示でき、typescriptなら型推論でnumberになる

//　使わない場合はこのようになるが、これだとundefined以外はそのまま返ってくる（"ひゃく"でも文字列で返ってくる）
let score;
if (userScore === null || userScore === undefined) {
  score = 100;
} else {
  score = userScore;
}
```

## 2. 条件付き表示の「&&」

「ログインしている時だけメニューを出す」といった、表示の切り替えに必須です。

```javascript
// isLoggedIn が true の時だけ右側が表示される
{
  isLoggedIn && <LogoutButton />;
}

// 使わないとif文になる
let button = null;
if (isLoggedIn) {
  button = <LogoutButton />;
}
```

## 3. データの複製と合成「...」（スプレッド構文）

React の「State（状態）」を更新する際、元のデータを直接書き換えずに（イミュータブルに）新しいデータを作るために必須です。

```javascript
const user = { id: 1, name: "Tanaka" };
// 元のデータを保持しつつ、一部だけ更新した新しいオブジェクトを作る
const updatedUser = { ...user, name: "Sato" };

// データの追加とかもできる
const productList = [
  { id: 1, name: "NIKON", price: "60000" },
  { id: 2, name: "LEICA", price: "120000" },
];

const updatedList = [
  ...productList,
  { id: 3, name: "CANON", price: "50000" },
  { id: 4, name: "SONY", price: "70000" },
];
```

## 4. 配列を自在に操る「map」と「filter」

配列操作は「宣言的」に行います。

**map（一括変換）**  
配列の全要素を加工して、新しい配列を作ります。React のリスト表示の基本です。

```javascript
const productList = [
  { id: 1, name: "NIKON", price: "60000" },
  { id: 2, name: "LEICA", price: "120000" },
  { id: 3, name: "CANON", price: "50000" },
  { id: 4, name: "SONY", price: "70000" },
];

// 全部に税込価格でリストを新規で作る
const taxIncludeProducts = productList.map((p) => ({
  ...p, // { id: 1, name: "NIKON", price: "60000" },から１つずつコピー
  price: p.price * 1.08,
  // priceだけ上書き
}));
```

**filter（絞り込み）**  
条件に合う要素だけを抽出します。「削除機能」の実装などによく使います。

```javascript
const productList = [
  { id: 1, name: "NIKON", price: 60000, isAvailable: true },
  { id: 2, name: "LEICA", price: 120000, isAvailable: false },
  { id: 3, name: "CANON", price: 50000, isAvailable: true },
  { id: 4, name: "SONY", price: 70000, isAvailable: true },
];

// 販売中のものだけを絞り込む
const availableProducts = productList.filter((product) => product.isAvailable);
console.log(availableProducts);

// 結果
[
  { id: 1, name: "NIKON", price: 60000, isAvailable: true },
  { id: 3, name: "CANON", price: 50000, isAvailable: true },
  { id: 4, name: "SONY", price: 70000, isAvailable: true },
];
```

## 5. 究極の集計武器「reduce」

配列をたった一つの値（数値、文字列、あるいは新しいオブジェクト）に集約。

```javascript
const result = productList.reduce(
  (acc, product) => {
    // 初期値変数配列をacc= []としてproductListの１つ１つをproductへ
    if (product.isAvailable) {
      // 1. 合計（税込価格）
      acc.totalTaxPrice += product.price * 1.08;

      // 2. 名前リスト
      acc.names.push(product.name);

      // 3. id でのオブジェクト化
      acc.byId[product.id] = product;
    }
    return acc;
    // accの配列をresultの名前として出力
  },
  { totalTaxPrice: 0, names: [], byId: {} } // 設定した変数accの初期データ、ここからそれぞれデータが入る
);

console.log(result);

// 結果
result =
  {
    totalTaxPrice: 194400,
    names: ["NIKON", "CANON", "SONY"],
    byId: {
      1: { id: 1, name: "NIKON", price: 60000, isAvailable: true },
      3: { id: 3, name: "CANON", price: 50000, isAvailable: true },
      4: { id: 4, name: "SONY", price: 70000, isAvailable: true },
    },
  },
;
```

## まとめ

これらのコードを理解して書いていこうと思います。  
特に React ではデータを扱うことが必須なのでなれていきたいです。
