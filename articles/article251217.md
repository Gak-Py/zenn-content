---
title: "JavaScript と TypeScript の違いと実務での最低限の書き方"
emoji: "💻"
type: "tech"
topics: ["JavaScript", "TypeScript", "フロントエンド", "開発"]
published: true
published_at: 2025-12-22 20:00
---

## React・Next.js を使うために Typescript を学ぶ

React を使うときにセットで覚えないといけない Typescript。
JS に型を指定してより厳格なデータ受送信をするための上位互換なイメージですが、基礎的なところ・最低限ここはやっといた方がよさそうな感じの部分をまとめてみました。

---

## 1. 変数宣言の比較

### JavaScript

```javascript
let productName = "Laptop";
const price = 1500;

productName = "Desktop"; // OK
```

### TypeScript

```typescript
// 型注釈なしでも型推論で string / number が自動判定される
let productName = "Laptop";
const price = 1500;

productName = "Desktop"; // OK
// productName = 123; // ✖ 型エラー
```

**ポイント**

- TS では型推論が働くため、基本的に型注釈なしでも安全。
- 「関数の引数・戻り値」「外部データの型」だけ明示する、でもいいかも

## 2. 関数の比較

### JavaScript

```javascript
function calculateDiscount(price, discount) {
  return price - discount;
}

console.log(calculateDiscount(1500, 200)); // 1300
console.log(calculateDiscount("1500", "200")); // "1500200"（意図しない動作になりうる）
```

### TypeScript

```typescript
// 必要に応じて型注釈を付ける
function calculateDiscount(price: number, discount: number): number {
  return price - discount;
}

// 型推論を活かした短縮書き方
const calcDiscount = (price = 0, discount = 0) => price - discount; // 戻り値は number と推論される
```

**ポイント**

- 初期値や処理内容で型が明確なら型注釈を省略可能。
- 外部から使われる関数は型注釈を付けると安全。

## 3. オブジェクト・配列の型### JavaScript

```javascript
const product = { name: "Laptop", price: 1500 };
const prices = [1500, 1200, 800];
```

### TypeScript（型推論活用）

```typescript
const product = { name: "Laptop", price: 1500 };
// {name: string, price: number} と推論
// const prices = [1500, 1200, 800]; // number[] と推論
```

**ポイント**

- 単純なオブジェクトや配列は型推論で十分。
- 複雑なオブジェクトや外部 API の返却値は interface/type で型を明示。

## 4. 実務で最低限押さえる TS の書き方（型推論前提）

```typescript
// 関数
const calcTotal = (price: number, quantity: number) => price * quantity; // 戻り値は number と推論

// オブジェクト（単純なら推論でOK）
const product = { name: "Laptop", price: 1500 };

// 配列
const products = [
  { name: "Laptop", price: 1500 },
  { name: "Mouse", price: 50 },
]; // {name:string, price:number}[] と推論
```

- 型注釈は必要な箇所のみ追加（関数の引数・戻り値、外部 API データなど）
- any は極力使わない
- tsconfig.json は strict モード必須

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true
  }
}
```

## 5. JS → TS 移行フロー（型推論活用）

- JS で書く
- 拡張子を .ts に変更
- 型注釈は 必要な箇所のみ追加
- 型推論で十分な部分は注釈不要
- strict モードで型チェック
