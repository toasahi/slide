---
marp: true
---

<script src="theme/tailwindcss-3.0.16.js"></script>
<script>tailwind.config = { corePlugins: { preflight: false } }</script>

# A. タイトル・セクション系

---

<!-- パターン1: タイトルスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 1. タイトルスライド

<p class="text-em-xl mt-4">サブタイトルがここに入ります</p>

<p class="text-em-lg mt-8">株式会社サンプル 代表取締役 山田太郎</p>

---

<!-- パターン2: セクション開始スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 2. セクション開始スライド

<p class="text-em-xl mt-4">新しい章の始まりを明確づける</p>

<p class="text-em-lg mt-6">章タイトルはここ</p>

---

<!-- パターン3: セクション終了・まとめ -->

# 3. セクション終了・まとめ

<p class="text-em-lg text-gray-600 mt-2">章の要点を整理して次へつなげる</p>

<div class="bg-gray-50 rounded-xl shadow-lg p-6 mt-4">
  <ul class="text-em-lg space-y-3 text-gray-700">
    <li>1.主なポイントはここに入ります</li>
    <li>2.大切なことがわかりやすく</li>
    <li>3.まとめのポイントはここにあります</li>
  </ul>
</div>

<p class="text-em-base text-gray-500 mt-4">この章で解説したことは？</p>

---

<!-- パターン4: 目次スライド -->

# 4. 目次スライド

<p class="text-em-base text-gray-600 mt-2">講義全体の構成を示す</p>

<div class="grid grid-cols-4 gap-4 mt-6">
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">第1章</p>
    <p class="text-em-base">概要とゴール</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">第2章</p>
    <p class="text-em-base">環境を知る</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">第3章</p>
    <p class="text-em-base">実践する</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">第4章</p>
    <p class="text-em-base">まとめ</p>
  </div>
</div>

---

<!-- パターン5: クロージングスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 5. クロージングスライド

<p class="text-em-lg mt-4">感謝と連絡先を伝える</p>

<p class="text-em-xl mt-8">ご清聴ありがとうございました</p>

<p class="text-em-base mt-4">example@example.com</p>

---

# B. カラムレイアウト系

---

<!-- パターン6: 2カラム比較（Before/After） -->

# 6. 2カラム比較（Before/After）

<p class="text-em-base text-gray-600">変化を左右で対比させるパターン</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-50 rounded-xl shadow-lg p-6">
    <h2 class="text-em-xl font-bold mb-4 text-gray-700">Before</h2>
    <ul class="text-em-lg space-y-2 text-gray-600">
      <li>・従来の方法の説明</li>
      <li>・課題点を記載</li>
      <li>・改善前の状態</li>
    </ul>
  </div>
  <div class="bg-gray-100 rounded-xl shadow-lg p-6">
    <h2 class="text-em-xl font-bold mb-4 text-gray-700">After</h2>
    <ul class="text-em-lg space-y-2 text-gray-600">
      <li>・新しい方法の説明</li>
      <li>・改善点を記載</li>
      <li>・改善後の状態</li>
    </ul>
  </div>
</div>

---

<!-- パターン7: 2カラム（テキスト+画像） -->

# 7. 2カラム（テキスト+画像）

<p class="text-em-base text-gray-600">テキストと画像を横並びに配置</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div>
    <p class="text-em-lg font-bold text-gray-700">テキストエリア</p>
    <p class="text-em-base text-gray-600 mt-2">
      説明文がここに入ります。画像と組み合わせて
      情報を効果的に伝えることができます。
    </p>
  </div>
  <div class="bg-gray-200 rounded-xl flex items-center justify-center h-40">
    <p class="text-gray-500">画像エリア</p>
  </div>
</div>

---

<!-- パターン7b: 2カラム（画像+テキスト） -->

# 7b. 2カラム（画像+テキスト）

<p class="text-em-base text-gray-600">画像とテキストを横並びに配置（逆パターン）</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-200 rounded-xl flex items-center justify-center h-40">
    <p class="text-gray-500">画像エリア</p>
  </div>
  <div>
    <p class="text-em-lg font-bold text-gray-700">テキストエリア</p>
    <p class="text-em-base text-gray-600 mt-2">
      説明文がここに入ります。画像と組み合わせて
      情報を効果的に伝えることができます。
    </p>
  </div>
</div>

---

<!-- パターン8: 3カラム（画像+テキスト） -->

# 8. 3カラム（画像+テキスト）

<p class="text-em-base text-gray-600">画像カードを横に3つ並べる</p>

<div class="grid grid-cols-3 gap-4 mt-6">
  <div class="bg-gray-50 rounded-xl shadow p-4 text-center">
    <div class="text-em-2xl mb-2">📊</div>
    <p class="text-em-lg font-bold text-gray-700">データ分析</p>
    <p class="text-em-base text-gray-600 mt-1">説明テキスト</p>
  </div>
  <div class="bg-gray-50 rounded-xl shadow p-4 text-center">
    <div class="text-em-2xl mb-2">🔄</div>
    <p class="text-em-lg font-bold text-gray-700">チームワーク</p>
    <p class="text-em-base text-gray-600 mt-1">説明テキスト</p>
  </div>
  <div class="bg-gray-50 rounded-xl shadow p-4 text-center">
    <div class="text-em-2xl mb-2">💡</div>
    <p class="text-em-lg font-bold text-gray-700">イノベーション</p>
    <p class="text-em-base text-gray-600 mt-1">説明テキスト</p>
  </div>
</div>

---

<!-- パターン9: 3カラム（アクセントカラー付き） -->

# 9. 3カラム（アクセントカラー付き）

<p class="text-em-base text-gray-600">色で3つの項目を区別する</p>

<div class="grid grid-cols-3 gap-4 mt-6">
  <div class="rounded-lg shadow p-4 text-center"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <p class="text-em-xl font-bold text-white">機能A</p>
    <p class="text-em-base text-white mt-2">説明</p>
  </div>
  <div class="rounded-lg shadow p-4 text-center"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <p class="text-em-xl font-bold text-white">機能B</p>
    <p class="text-em-base text-white mt-2">説明</p>
  </div>
  <div class="rounded-lg shadow p-4 text-center"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <p class="text-em-xl font-bold text-white">機能C</p>
    <p class="text-em-base text-white mt-2">説明</p>
  </div>
</div>

---

<!-- パターン10: 4カラムレイアウト -->

# 10. 4カラムレイアウト

<p class="text-em-base text-gray-600">4つのフェーズや選択肢を並べる</p>

<div class="grid grid-cols-4 gap-3 mt-6">
  <div class="bg-gray-50 rounded-lg shadow p-3 text-center">
    <p class="text-em-lg font-bold text-gray-700">Phase 1</p>
    <p class="text-em-base text-gray-600 mt-1">準備</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3 text-center">
    <p class="text-em-lg font-bold text-gray-700">Phase 2</p>
    <p class="text-em-base text-gray-600 mt-1">計画</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3 text-center">
    <p class="text-em-lg font-bold text-gray-700">Phase 3</p>
    <p class="text-em-base text-gray-600 mt-1">実行</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3 text-center">
    <p class="text-em-lg font-bold text-gray-700">Phase 4</p>
    <p class="text-em-base text-gray-600 mt-1">評価</p>
  </div>
</div>

---

<!-- パターン11: 5カラム（成熟度レベル） -->

# 11. 5カラム（成熟度レベル）

<p class="text-em-base text-gray-600">段階的な進化をグラデーションで表現</p>

<div class="grid grid-cols-5 gap-2 mt-6">
  <div class="bg-gray-100 rounded p-2 text-center">
    <p class="text-em-lg font-bold">Level 1</p>
    <p class="text-em-base">初期</p>
  </div>
  <div class="bg-gray-200 rounded p-2 text-center">
    <p class="text-em-lg font-bold">Level 2</p>
    <p class="text-em-base">管理</p>
  </div>
  <div class="bg-gray-300 rounded p-2 text-center">
    <p class="text-em-lg font-bold">Level 3</p>
    <p class="text-em-base">定義</p>
  </div>
  <div class="bg-gray-400 rounded p-2 text-center text-white">
    <p class="text-em-lg font-bold">Level 4</p>
    <p class="text-em-base">定量</p>
  </div>
  <div class="rounded p-2 text-center text-white"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <p class="text-em-lg font-bold">Level 5</p>
    <p class="text-em-base">最適化</p>
  </div>
</div>

---

<!-- パターン12: 2x2グリッド（画像+テキスト） -->

# 12. 2x2グリッド（画像+テキスト）

<p class="text-em-base text-gray-600">4つの要素を2行2列で整理</p>

<div class="grid grid-cols-2 gap-4 mt-6">
  <div class="bg-gray-50 rounded-lg shadow p-4 flex items-center">
    <div class="text-em-2xl mr-3">📊</div>
    <div>
      <p class="text-em-lg font-bold text-gray-700">データ分析</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 flex items-center">
    <div class="text-em-2xl mr-3">🔄</div>
    <div>
      <p class="text-em-lg font-bold text-gray-700">イノベーション</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 flex items-center">
    <div class="text-em-2xl mr-3">💡</div>
    <div>
      <p class="text-em-lg font-bold text-gray-700">応用戦略</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 flex items-center">
    <div class="text-em-2xl mr-3">🎯</div>
    <div>
      <p class="text-em-lg font-bold text-gray-700">目標達成</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
</div>

---

<!-- パターン13: 2x3グリッドレイアウト -->

# 13. 2x3グリッドレイアウト

<p class="text-em-base text-gray-600">6つの要素を2行3列で整理</p>

<div class="grid grid-cols-3 gap-3 mt-4">
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目1</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目2</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目3</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目4</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目5</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-3">
    <p class="text-em-lg font-bold text-gray-700">項目6</p>
    <p class="text-em-base text-gray-600">説明文</p>
  </div>
</div>

---

# C. 縦並びリスト系

---

<!-- パターン14: 縦3つステップ -->

# 14. 縦3つステップ

<p class="text-em-base text-gray-600">番号付きで順序を明示する</p>

<div class="space-y-3 mt-6">
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">01</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">ステップ1</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">02</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">ステップ2</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">03</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">ステップ3</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
</div>

---

<!-- パターン15: 番号付きステップ（横型） -->

# 15. 番号付きステップ（横型）

<p class="text-em-base text-gray-600">矢印でプロセスの流れを示す</p>

<div class="flex items-center justify-between mt-8">
  <div class="text-center">
    <div class="bg-gray-100 rounded-full w-16 h-16 flex items-center justify-center mx-auto">
      <span class="text-em-xl font-bold">1</span>
    </div>
    <p class="text-em-base mt-2">ステップ1</p>
    <p class="text-em-base text-gray-500">説明</p>
  </div>
  <div class="text-em-xl text-gray-400">→</div>
  <div class="text-center">
    <div class="bg-gray-100 rounded-full w-16 h-16 flex items-center justify-center mx-auto">
      <span class="text-em-xl font-bold">2</span>
    </div>
    <p class="text-em-base mt-2">ステップ2</p>
    <p class="text-em-base text-gray-500">説明</p>
  </div>
  <div class="text-em-xl text-gray-400">→</div>
  <div class="text-center">
    <div class="bg-gray-100 rounded-full w-16 h-16 flex items-center justify-center mx-auto">
      <span class="text-em-xl font-bold">3</span>
    </div>
    <p class="text-em-base mt-2">ステップ3</p>
    <p class="text-em-base text-gray-500">説明</p>
  </div>
  <div class="text-em-xl text-gray-400">→</div>
  <div class="text-center">
    <div class="bg-gray-100 rounded-full w-16 h-16 flex items-center justify-center mx-auto">
      <span class="text-em-xl font-bold">4</span>
    </div>
    <p class="text-em-base mt-2">ステップ4</p>
    <p class="text-em-base text-gray-500">説明</p>
  </div>
</div>

---

<!-- パターン16: タイムラインレイアウト -->

# 16. タイムラインレイアウト

<p class="text-em-base text-gray-600">時系列で出来事を並べる</p>

<div class="space-y-2 mt-6">
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">2020</div>
    <div class="flex-1 bg-gray-100 rounded p-3">
      <p class="text-em-lg font-bold text-gray-700">イベント1</p>
    </div>
  </div>
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">2022</div>
    <div class="flex-1 rounded p-3"
         style="background: linear-gradient(to right, #1B4565, #3E9BA4);">
      <p class="text-em-lg font-bold text-white">イベント2</p>
    </div>
  </div>
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">2024</div>
    <div class="flex-1 bg-gray-100 rounded p-3">
      <p class="text-em-lg font-bold text-gray-700">イベント3</p>
    </div>
  </div>
</div>

---

<!-- パターン17: アイコン付きリスト -->

# 17. アイコン付きリスト

<p class="text-em-base text-gray-600">絵文字やアイコンで視覚的に区別</p>

<div class="space-y-4 mt-6">
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">📊</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">項目タイトル1</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">🎯</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">項目タイトル2</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">💡</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">項目タイトル3</p>
      <p class="text-em-base text-gray-600">説明文がここに入ります、詳細など...</p>
    </div>
  </div>
</div>

---

# D. パネルデザイン系

---

<!-- パターン18: 基本パネル（画像ヘッダー付き） -->

# 18. 基本パネル（画像ヘッダー付き）

<p class="text-em-base text-gray-600">画像とテキストを組み合わせカード</p>

<div class="grid grid-cols-2 gap-4 mt-6">
  <div class="bg-gray-50 rounded-xl shadow-lg overflow-hidden">
    <div class="bg-gray-200 h-20 flex items-center justify-center">
      <span class="text-em-2xl">📊</span>
    </div>
    <div class="p-4">
      <p class="text-em-lg font-bold text-gray-700">パネルタイトル1</p>
      <p class="text-em-base text-gray-600 mt-1">説明文がここに入ります。</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-xl shadow-lg overflow-hidden">
    <div class="bg-gray-200 h-20 flex items-center justify-center">
      <span class="text-em-2xl">🔄</span>
    </div>
    <div class="p-4">
      <p class="text-em-lg font-bold text-gray-700">パネルタイトル2</p>
      <p class="text-em-base text-gray-600 mt-1">説明文がここに入ります。</p>
    </div>
  </div>
</div>

---

<!-- パターン19: 強調パネル（左ボーダー付き） -->

# 19. 強調パネル（左ボーダー付き）

<p class="text-em-base text-gray-600">左端のラインで重要度を示す</p>

<div class="grid grid-cols-2 gap-4 mt-6">
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #1B4565;">
    <p class="text-em-lg font-bold text-gray-700">パネルタイトル1</p>
    <p class="text-em-base text-gray-600 mt-2">説明文がここに入ります。</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #3E9BA4;">
    <p class="text-em-lg font-bold text-gray-700">パネルタイトル2</p>
    <p class="text-em-base text-gray-600 mt-2">説明文がここに入ります。</p>
  </div>
</div>

---

<!-- パターン20: ガラス風パネル -->

# 20. ガラス風パネル

<p class="text-em-base text-gray-600">背景を透かして洗練された印象に</p>

<div class="mt-6 rounded-xl p-6"
     style="background: rgba(30, 58, 82, 0.85); backdrop-filter: blur(10px);">
  <h2 class="text-em-xl font-bold text-white mb-4">ガラス風パネルタイトル</h2>
  <p class="text-em-lg text-white opacity-90">
    説明文がここに入ります。背景は透過である。究極はまだ遠い。どこで
    されるかによって全くことなるが、同じと違いつつおなじ何か知らという
    ニュアンスだけを待っている彼だけは記述している。
  </p>
</div>

---

<!-- パターン21: グラデーションパネル -->

# 21. グラデーションパネル

<p class="text-em-base text-gray-600">色の変化で目を引くパネル</p>

<div class="grid grid-cols-2 gap-4 mt-6">
  <div class="rounded-xl p-6"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <h2 class="text-em-lg font-bold text-white mb-2">濃いグラデーション</h2>
    <p class="text-em-base text-white opacity-90">
      説明文がここに入ります。
    </p>
  </div>
  <div class="rounded-xl p-6"
       style="background: linear-gradient(135deg, #3E9BA4 0%, #6BC4CE 100%);">
    <h2 class="text-em-lg font-bold text-white mb-2">薄いグラデーション</h2>
    <p class="text-em-base text-white opacity-90">
      説明文がここに入ります。
    </p>
  </div>
</div>

---

<!-- パターン22: カード型レイアウト（画像付き） -->

# 22. カード型レイアウト（画像付き）

<p class="text-em-base text-gray-600">製品やサービスを紹介するカード</p>

<div class="grid grid-cols-3 gap-4 mt-6">
  <div class="bg-gray-50 rounded-xl shadow overflow-hidden">
    <div class="bg-gray-200 h-24 flex items-center justify-center">
      <span class="text-em-2xl">🚀</span>
    </div>
    <div class="p-3">
      <p class="text-em-lg font-bold text-gray-700">高速開発</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-xl shadow overflow-hidden">
    <div class="bg-gray-200 h-24 flex items-center justify-center">
      <span class="text-em-2xl">🔒</span>
    </div>
    <div class="p-3">
      <p class="text-em-lg font-bold text-gray-700">自動連携</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
  <div class="bg-gray-50 rounded-xl shadow overflow-hidden">
    <div class="bg-gray-200 h-24 flex items-center justify-center">
      <span class="text-em-2xl">📈</span>
    </div>
    <div class="p-3">
      <p class="text-em-lg font-bold text-gray-700">詳細管理</p>
      <p class="text-em-base text-gray-600">説明</p>
    </div>
  </div>
</div>

---

# E. 背景・画像系

---

<!-- パターン23: 背景画像全画面 -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 23. 背景画像全画面

<p class="text-em-lg mt-4">画像でインパクトを最大化</p>

<p class="text-em-xl mt-8">
  ここを中央のテキスト
</p>

<p class="text-em-base mt-4">※実際は背景画像を使用</p>

---

<!-- パターン24: 背景画像右側配置 -->

# 24. 背景画像右側配置

<p class="text-em-base text-gray-600">左にテキスト、右に画像を配置</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div>
    <ul class="text-em-lg space-y-3 text-gray-700">
      <li>・説明文がここに入ります。</li>
      <li>・複数のポイントを記載します。</li>
      <li>・重要な情報を左側に配置。</li>
    </ul>
    <p class="text-em-base text-gray-600 mt-4">
      ※画像はここに入ります。右側は軽やかで、
      左側は詳しく書いても問題。
    </p>
  </div>
  <div class="bg-gray-200 rounded-xl flex items-center justify-center h-48">
    <p class="text-gray-500">画像エリア</p>
  </div>
</div>

---

<!-- パターン25: 引用スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 25. 引用スライド

<p class="text-em-base mt-2 opacity-80">印象的な言葉を引用する</p>

<div class="flex items-center justify-center mt-8">
  <div class="bg-white/10 rounded-xl p-8 text-center">
    <p class="text-em-xl italic">
      「引用の文章がここに入ります。心に響く言葉は相手にとっての...」
    </p>
    <p class="text-em-base mt-4 opacity-80">
      — 引用元
    </p>
  </div>
</div>

---

<!-- パターン26: 複数画像・分割背景 -->

# 26. 複数画像・分割背景

<p class="text-em-base text-gray-600">複数の画像を横並びで見せる</p>

<div class="grid grid-cols-2 gap-4 mt-6">
  <div class="bg-gray-200 rounded-xl h-40 flex items-center justify-center">
    <p class="text-gray-500">画像1</p>
  </div>
  <div class="bg-gray-200 rounded-xl h-40 flex items-center justify-center">
    <p class="text-gray-500">画像2</p>
  </div>
</div>

<p class="text-em-base text-gray-600 text-center mt-4">
  複数の画像を横並びで配置するパターン
</p>

---

# F. 強調・特殊系

---

<!-- パターン27: 統計強調スライド -->

# 27. 統計強調スライド

<p class="text-em-base text-gray-600">大きな数字でインパクトを出す</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-100 rounded-xl p-6 text-center">
    <p class="text-em-3xl font-bold text-gray-700">50%</p>
    <p class="text-em-lg mt-2">コスト削減</p>
  </div>
  <div class="rounded-xl p-6 text-center"
       style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <p class="text-em-3xl font-bold text-white">200%</p>
    <p class="text-em-lg mt-2 text-white">効率向上</p>
  </div>
</div>

---

<!-- パターン28: 中央配置メッセージ -->

# 28. 中央配置メッセージ

<p class="text-em-base text-gray-600">シンプルに一言を伝える</p>

<div class="flex items-center justify-center mt-12">
  <div class="text-center">
    <p class="text-em-2xl font-bold text-gray-800">
      大きなメッセージが<br>ここに入ります
    </p>
    <p class="text-em-lg text-gray-500 mt-6">
      English message here
    </p>
  </div>
</div>

---

<!-- パターン29: Q&Aスライド -->

# 29. Q&Aスライド

<p class="text-em-base text-gray-600">質問応答への誘導</p>

<div class="flex items-center justify-center mt-8">
  <div class="bg-gray-50 rounded-xl shadow-lg p-8 text-center">
    <p class="text-em-3xl font-bold text-gray-700">Q&A</p>
    <p class="text-em-lg text-gray-600 mt-4">ご質問をお待ちしています</p>
    <div class="mt-6 space-y-2 text-em-base text-gray-500">
      <p>質問があればお気軽にどうぞ</p>
      <p>後ほど個別に質問も可能です</p>
    </div>
  </div>
</div>

---

# G. 応用パターン系

---

<!-- パターン30: QRコード付き紹介 -->

# 30. QRコード付き紹介

<p class="text-em-base text-gray-600">資料やサイトへ誘導する</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div>
    <p class="text-em-lg text-gray-700">
      本日の資料は以下からダウンロードできます。
    </p>
    <p class="text-em-base text-gray-600 mt-4">
      https://example.com/slides
    </p>
  </div>
  <div class="bg-white rounded-xl p-8 flex items-center justify-center shadow">
    <div class="text-center">
      <div class="bg-gray-200 w-32 h-32 flex items-center justify-center mx-auto">
        <p class="text-gray-500 text-sm">QRコード</p>
      </div>
      <p class="text-em-base text-gray-500 mt-2">https://example.com</p>
    </div>
  </div>
</div>

---

<!-- パターン31: 問いかけスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 31. 問いかけスライド

<p class="text-em-base mt-2 opacity-80">聴衆への問いを中央配置で投げかける</p>

<div class="flex items-center justify-center mt-8">
  <div class="bg-white/10 rounded-xl p-8 text-center">
    <p class="text-em-xl">
      「問いかけの文章がここに入ります」
    </p>
    <p class="text-em-base mt-4 opacity-80">
      補足説明があれば書きます。想像したい時分になります。
    </p>
  </div>
</div>

---

<!-- パターン32: 映画引用スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 32. 映画引用スライド

<p class="text-em-base mt-2 opacity-80">映画のセリフを引用して印象づける</p>

<div class="mt-8 bg-white/10 rounded-xl p-6">
  <p class="text-em-lg italic">
    "引用の文章がここに入ります。<br>
    映画のセリフは大長い。"
  </p>
</div>

<p class="text-em-base mt-6">
  解説文がここに入ります。<br>
  この「キーワード」について考えてみましょう。
</p>

---

<!-- パターン33: インライン画像スライド -->

# 33. インライン画像スライド

<p class="text-em-base text-gray-600">画像とテキストを並べて解説する</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-200 rounded-xl h-40 flex items-center justify-center">
    <p class="text-gray-500">画像</p>
  </div>
  <div>
    <p class="text-em-lg font-bold text-gray-700">画像をコンテンツ内に配置</p>
    <p class="text-em-base text-gray-600 mt-2">
      説明文がここに入ります。画像とテ
      キストを横並びで配置するパターン
      です。
    </p>
    <ul class="text-em-base text-gray-600 mt-2 space-y-1">
      <li>・HTML/imgタグで配置</li>
      <li>・Tailwindでサイズ調整</li>
      <li>・flexで整列</li>
    </ul>
  </div>
</div>

---

<!-- パターン34: 統計比率スライド -->

# 34. 統計比率スライド

<p class="text-em-base text-gray-600">数値データを視覚的に比較する</p>

<div class="grid grid-cols-4 gap-4 mt-6">
  <div class="bg-gray-100 rounded-xl p-4 text-center">
    <p class="text-em-2xl font-bold" style="color: #1B4565;">25%</p>
    <p class="text-em-base text-gray-600 mt-2">項目A</p>
  </div>
  <div class="bg-gray-100 rounded-xl p-4 text-center">
    <p class="text-em-2xl font-bold" style="color: #2A6B7C;">35%</p>
    <p class="text-em-base text-gray-600 mt-2">項目B</p>
  </div>
  <div class="bg-gray-100 rounded-xl p-4 text-center">
    <p class="text-em-2xl font-bold" style="color: #3E9BA4;">25%</p>
    <p class="text-em-base text-gray-600 mt-2">項目C</p>
  </div>
  <div class="bg-gray-100 rounded-xl p-4 text-center">
    <p class="text-em-2xl font-bold" style="color: #5FBCC5;">15%</p>
    <p class="text-em-base text-gray-600 mt-2">項目D</p>
  </div>
</div>

<p class="text-em-base text-gray-500 text-center mt-4">
  統計文がここに入ります。一緒に解
</p>

---

<!-- パターン35: テキスト+統計パネル混合 -->

# 35. テキスト+統計パネル混合

<p class="text-em-base text-gray-600">説明文と数値を組み合わせる</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div>
    <p class="text-em-lg font-bold text-gray-700">解説テキスト</p>
    <p class="text-em-base text-gray-600 mt-2">
      ここに説明文を記載します。数値データと
      組み合わせて説得力のある情報を提供。
    </p>
  </div>
  <div class="grid grid-cols-2 gap-3">
    <div class="bg-gray-100 rounded-lg p-3 text-center">
      <p class="text-em-xl font-bold text-gray-700">120%</p>
      <p class="text-em-base text-gray-600">指標A</p>
    </div>
    <div class="bg-gray-100 rounded-lg p-3 text-center">
      <p class="text-em-xl font-bold text-gray-700">85%</p>
      <p class="text-em-base text-gray-600">指標B</p>
    </div>
  </div>
</div>

---

<!-- パターン36: まとめスライド（ガラス風縦並び） -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 36. まとめスライド（ガラス風縦並び）

<p class="text-em-base opacity-80">セクションの要点をガラス風パネルで整理</p>

<div class="space-y-3 mt-6">
  <div class="bg-white/20 rounded-lg p-4">
    <p class="text-em-lg font-bold">🎯 まとめポイント1のタイトル</p>
  </div>
  <div class="bg-white/20 rounded-lg p-4">
    <p class="text-em-lg font-bold">📊 まとめポイント2のタイトル</p>
  </div>
  <div class="bg-white/20 rounded-lg p-4">
    <p class="text-em-lg font-bold">💡 まとめポイント3のタイトル</p>
  </div>
</div>

---

<!-- パターン37: シンプルリスト+補足パネル -->

# 37. シンプルリスト+補足パネル

<p class="text-em-base text-gray-600">箇条書きに補足情報を添える</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div>
    <ul class="text-em-lg space-y-3 text-gray-700">
      <li>・リスト項目がここに入ります。短い説明をするなら...</li>
      <li>・リスト項目がここに入ります。短い説明をするなら...</li>
      <li>・項目したいリスト項目がここに入ります。</li>
    </ul>
  </div>
  <div class="bg-gray-50 rounded-xl shadow p-4">
    <p class="text-em-lg font-bold text-gray-700">補足情報</p>
    <p class="text-em-base text-gray-600 mt-2">
      ここに補足的な情報を記載します。
      関連アクションへの誘導も可能。
    </p>
  </div>
</div>

---

<!-- パターン38: 対比+結論スライド -->

# 38. 対比+結論スライド

<p class="text-em-base text-gray-600">二項対立から結論を導く</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-50 rounded-xl shadow p-4">
    <p class="text-em-lg font-bold text-gray-700">観点A</p>
    <p class="text-em-base text-gray-600 mt-2">説明文</p>
  </div>
  <div class="bg-gray-50 rounded-xl shadow p-4">
    <p class="text-em-lg font-bold text-gray-700">観点B</p>
    <p class="text-em-base text-gray-600 mt-2">説明文</p>
  </div>
</div>

<div class="mt-4 rounded-xl p-4 text-white text-center"
     style="background: linear-gradient(to right, #1B4565, #3E9BA4);">
  <p class="text-em-lg font-bold">結論: 両方を組み合わせることが重要</p>
</div>

---

<!--
==========================================================
パターン集終了
合計38パターン（7カテゴリ）

A. タイトル・セクション系: 1-5
B. カラムレイアウト系: 6-13（7bを含む）
C. 縦並びリスト系: 14-17
D. パネルデザイン系: 18-22
E. 背景・画像系: 23-26
F. 強調・特殊系: 27-29
G. 応用パターン系: 30-38
==========================================================
-->
