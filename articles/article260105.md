---
title: "Promiseから学ぶasync/await。並列処理まで含めた非同期処理の『正解』"
emoji: "⏳"
type: "tech"
topics: ["javascript", "es6", "frontend", "programming"]
published: true,
published_at: 2026-01-05 18:05
---

モダンなフロントエンド開発において、API との通信（非同期処理）は避けて通れない業務。

今回は async/await 以前の Promise から理解してまとめていきたいと思います。

---

## 1. async/await に行く前に。全ての基礎「Promise」の書き方

async 以前は`Promise` オブジェクトを使い、`.then()` で処理をつなげていました。  
async/await があるから今は覚えなくてよさそう、とも思いますが Promise の構造を知っておいた方がいいと思うので記載しました。

```javascript
// Promiseの基本
const myFirstPromise = () => {
  return new Promise((resolve, reject) => {
    const success = true;
    if (success) {
      resolve("データを取得しました！");
    } else {
      reject("失敗しました...");
    }
  });
};

// 実際に使うときはPromiseチェーン
// 東京（緯度:35.67, 経度:139.65）の現在の天気を例として取得
const url =
  "https://api.open-meteo.com/v1/forecast?latitude=35.67&longitude=139.65&current_weather=true";

console.log("通信開始...");

fetch(url) // fetch()関数でデータurlを取得
  .then((response) => {
    // 受け取ったデータがおかしい・なければエラーを出す
    if (!response.ok) {
      throw new Error("ネットワークエラーが発生しました");
    }
    // OKであればデータをjsonにして出力して次のステップへ
    return response.json(); // ここでまたPromiseが返る（解析待ち）
  })
  .then((data) => {
    // jsonデータができたらこのステップに移動する
    console.log("取得成功！", data.current_weather);
    console.log(`現在の気温は ${data.current_weather.temperature}度 です。`);
  })
  // jsonデータがあってもdata.current_weatherなどのデータがなかったらエラーを出す
  .catch((error) => {
    console.error("エラーが発生しました:", error);
  })
  // 一番最後に出力する
  .finally(() => {
    console.log("通信処理を終了します。");
  });
```

わかりにくくはないけどチェーンが目に付く感じです。

## 2. async/await で書き換える場合

async を使うとチェーンもなくなりすっきりしたコードになります。

```javascript
const getWeather = async () => {
  const url =
    "https://api.open-meteo.com/v1/forecast?latitude=35.67&longitude=139.65&current_weather=true";

  try {
    console.log("通信開始...");
    const response = await fetch(url);

    if (!response.ok) throw new Error("取得に失敗しました");

    const data = await response.json();
    console.log(`現在の気温は ${data.current_weather.temperature}度 です。`);
  } catch (error) {
    console.error("データを受信できませんでした。", error);
  } finally {
    console.log("処理終了");
  }
};

getWeather();
```

## 3. パフォーマンスを最大化する「Promise.all」

「ユーザー情報」と「投稿一覧」を両方取得する場合など、それぞれのデータ取得に順序建てが必要ない場合は Promise.all を使うことで並列処理が可能です。

```javascript
// 悪い例（直列処理）：1つ目が終わるまで2つ目が始まらない
const user = await fetchUser(); // 1秒かかる
const posts = await fetchPosts(); // 1秒かかる
// 合計 2秒

// 良い例（並列処理）：同時にリクエストを送る
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
// 同時に処理するので1秒で済む
```

## 4. React ではこんな感じの記載に

非同期処理は、常に「3 つの状態」とセットで設計する必要があります。

- Loading（読み込み中）：ぐるぐるを表示する
- Success（成功）：データを表示する
- Error（失敗）：エラーメッセージを出す

```javascript
const [loading, setLoading] = useState(false);

const handleAction = async () => {
  console.log("データを取得します。しばらくお待ちください。");
  setLoading(true); // 開始
  try {
    await someApiCall();
  } catch (error) {
    console.error("データを受信できませんでした。", error);
  } finally {
    setLoading(false);
    console.log("処理を終了いたします。");
  }
};
```

## まとめ：安全に処理し、エラー・終わりまで記載してエンジニアにもユーザーにもわかりやすく

非同期処理を丁寧に書くことは、そのままユーザーへの優しさに繋がります。

通信エラーで画面が固まらないようにする

Promise.all で待ち時間を最小にする

適切なエラーメッセージでユーザーを迷わせない

「動けばいい」だけでなく 「わかりやすい」 コードを書いていきたいと思います。
