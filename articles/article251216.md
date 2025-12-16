---
title: 実務で使えるチェックボタン同期デモ
emoji: "✅"
type: tech
topics: [javascript, html, css, flocss]
published: true
published_at: 2025-12-16 23:00
---

## 実務でチェックボタンの同期が必要になる場面

実務で一覧画面やフォームを作っていると、「同じデータに対して複数箇所にチェックボタンがある」というケースがあります。  
例えば：

- 詳細レイアウトとカードレイアウトの両方に同じ商品を表示している
- どちらかでチェックしたらもう片方のチェックも連動させたい
- 全選択・個別選択の同期もほしい

こういう要件は、意外とよくあるのですが、普通に HTML だけ書いたのでは対応できません。  
そこで、今回は**FLOCSS でデザインしつつ JS でチェック同期を実装**したサンプルを作りました。

---

## サンプルの動作

StackBlitz で実際に動くデモはこちらです。  
記事内でそのまま操作できます。

@[stackblitz](https://stackblitz.com/edit/stackblitz-starters-hdqzf2am?embed=1&file=index.html&view=preview)

---

## JS の各部分の解説

### 1. タブ切り替え部分

```js
// タブ切り替え
document.querySelectorAll(".c-tabs__btn").forEach((btn) => {
  btn.addEventListener("click", () => {
    const tab = btn.dataset.tab;

    // toggleで一括切り替え
    document.querySelectorAll(".c-tab-content").forEach((el) => {
      el.classList.toggle("is-active", el.id === tab);
    });

    // タブボタンのアクティブ切り替え
    document
      .querySelectorAll(".c-tabs__btn")
      .forEach((b) => b.classList.remove("is-active"));
    btn.classList.add("is-active");
  });
});
```

- classList.toggle('is-active', 条件) を使うことで、CSS の display: block/none を直接書かずに表示切替
- タブボタンのアクティブ状態も同期

### 2. チェック同期の関数

```js
function syncCheckbox(checkbox) {
  const id = checkbox.dataset.id;
  const checked = checkbox.checked;

  // 同じIDのチェックを全て同期
  document
    .querySelectorAll(`.js-check[data-id="${id}"]`)
    .forEach((el) => (el.checked = checked));

  // 全選択の状態更新
  const allItems = document.querySelectorAll(".js-check[data-id]");
  const allChecked = [...allItems].every((el) => el.checked);
  document
    .querySelectorAll('.js-check[data-role="select-all"]')
    .forEach((el) => (el.checked = allChecked));
}
```

- 同じ data-id のチェックボックスをまとめて同期
- 全てチェックされているかを確認して、「全て選択」チェックボックスの状態も更新

### 3. チェック同期の関数

```js
document.querySelectorAll(".js-check[data-id]").forEach((cb) => {
  cb.addEventListener("change", () => syncCheckbox(cb));
});
```

- input 自身がクリックされて change したときに、同期処理だけ呼ぶ

### 4. 行クリックでのチェック反転

```js
document.querySelectorAll(".c-detail-row, .c-card").forEach((row) => {
  row.addEventListener("click", (e) => {
    const checkbox = row.querySelector('input[type="checkbox"]');
    if (!checkbox) return;

    if (!e.target.closest("label")) {
      checkbox.checked = !checkbox.checked;
    }

    syncCheckbox(checkbox);
  });
});
```

- 行全体をクリックしてもチェックが切り替わるようにする処理
- ラベル内のテキストクリックは無視して二重反転を防ぐ
- どちらの場合も syncCheckbox で同期

### 5. 全選択チェックボックス

```js
document.querySelectorAll('.js-check[data-role="select-all"]').forEach((cb) => {
  cb.addEventListener("change", (e) => {
    const checked = cb.checked;
    document.querySelectorAll(".js-check[data-id]").forEach((el) => {
      el.checked = checked;
      syncCheckbox(el);
    });
  });
});
```

- 「全て選択」をチェックすると、全ての個別チェックも連動
- 個別チェックが全て ON になれば、自動で全選択も ON

### まとめ

- 「複数レイアウトに同じデータのチェックがある」場合、JS で同期処理を書く必要がある
- 行クリック、ラベル内クリック、全選択など、操作パターンに応じて処理を整理すると使いやすい UI が作れる
- 複数の機能を使う場合はクラス+ data 属性で役目をわけるとコードがキレイになる
- クラスの追加・削除系は toggle 処理だとスマート
