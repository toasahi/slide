---
name: layout-fix
description: Marpスライドのレイアウト崩れを検出し自動修正する
---

# layout-fix

Marpスライドのレイアウト崩れを視覚的に検出し、自動修正するスキル。

## 前提条件

- ブラウザ自動化ツール（Playwright/Puppeteer）が利用可能であること
- `npx @marp-team/marp-cli` が実行可能であること

## 処理手順

1. Marp CLIでHTMLをエクスポート
2. ブラウザでスライドを開く
3. 各スライドのスクリーンショットを撮影
4. 画像を確認してレイアウト崩れを検出
5. 修正を適用
6. 再度スクリーンショットで確認

## 検出パターンと修正方法

### 1. オーバーフロー（下部が切れる）

**原因**: gap/margin/paddingが大きすぎる

```html
<!-- 修正前 -->
<div class="grid grid-cols-1 gap-4 mt-6">
  <div class="p-5">...</div>
</div>

<!-- 修正後 -->
<div class="grid grid-cols-1 gap-2 mt-2">
  <div class="p-3">...</div>
</div>
```

**修正指針**:
- `gap-4` → `gap-2`
- `mt-6` → `mt-2`
- `p-5` → `p-3`

### 2. 縦5項目以上が収まらない

**原因**: 1カラムでは物理的に収まらない

```html
<!-- 修正前: 縦1列で5項目 -->
<div class="grid grid-cols-1 gap-3">
  <div>項目1</div>
  <div>項目2</div>
  <div>項目3</div>
  <div>項目4</div>
  <div>項目5</div>
</div>

<!-- 修正後: 2列グリッドに変更 -->
<div class="grid grid-cols-2 gap-3">
  <div>項目1</div>
  <div>項目2</div>
  <div>項目3</div>
  <div>項目4</div>
  <div class="col-span-2">項目5（全幅）</div>
</div>
```

### 3. テキストがはみ出す

**原因**: フォントサイズが大きすぎる

**修正指針**:
- `text-em-3xl` → `text-em-2xl`
- `text-em-2xl` → `text-em-xl`
- `text-em-xl` → `text-em-lg`

### 4. 要素が重なる

**原因**: 絶対位置指定の競合

**修正指針**:
- `absolute` を使用している場合は `relative` + Flexbox/Grid に変更
- `z-index` の調整

## 使い方

```
/layout-fix をつかってslides/my-presentation.mdのスライド15-25を
確認して、レイアウト崩れがあれば修正して
```

## スクリーンショット撮影コマンド例

```bash
# HTMLにエクスポート
npx @marp-team/marp-cli slides/presentation.md -o output.html

# Playwrightでスクリーンショット撮影
npx playwright screenshot output.html screenshots/slide-%d.png
```

## チェックポイント

- [ ] 全スライドが画面内に収まっているか
- [ ] テキストが切れていないか
- [ ] 要素が重なっていないか
- [ ] 余白が均等か
- [ ] フォントサイズが適切か
