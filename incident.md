---
marp: true
slide: true
---

<script src="theme/tailwindcss-3.0.16.js"></script>
<script>tailwind.config = { corePlugins: { preflight: false } }</script>

<!-- パターン1: タイトルスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# インシデント対応のビジョン

<p class="text-em-xl mt-4">AIエージェントが切り拓く新しい対応の形</p>

<p class="text-em-lg mt-8">2026年1月31日</p>

---


# 自己紹介

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-200 w-80 h-80 rounded-full flex items-center justify-center">
    <p class="text-gray-500">
        <img src="" />
    </p>
  </div>
  <div>
    <p class="text-em-xl font-bold text-gray-700">戸田 麻陽</p>
    <p class="text-em-lg text-gray-600 mt-2">とだ あさひ</p>
    <p class="text-em-base text-gray-600 mt-4">
      所属部署や役職など<br>
      簡単な経歴を記載
    </p>
  </div>
</div>

---


<!-- パターン4: 目次スライド -->

# アジェンダ

<p class="text-em-base text-gray-600 mt-2">本日お話しする内容</p>

<div class="grid grid-cols-4 gap-4 mt-6">
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">01</p>
    <p class="text-em-base">本日の目的</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">02</p>
    <p class="text-em-base">現状と課題</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">03</p>
    <p class="text-em-base">ビジョンと解決策</p>
  </div>
  <div class="bg-gray-50 rounded-lg p-4 border-l-4 border-gray-400 text-center">
    <p class="text-em-xl font-bold text-gray-700">04</p>
    <p class="text-em-base">デモとロードマップ</p>
  </div>
</div>

---


<!-- パターン2: セクション開始スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 1.本日の目的

---

# 本日の目的

<p class="text-em-base text-gray-600">皆さまへのお願い事項</p>

<div class="grid grid-cols-1 gap-4 mt-3">
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #1B4565;">
    <p class="text-em-lg font-bold text-gray-700">インシデント対応の課題の共感</p>
    <p class="text-em-base text-gray-600 mt-2">本日のデモをご覧いただき、ご意見やご懸念をお聞かせください</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #3E9BA4;">
    <p class="text-em-lg font-bold text-gray-700">インシデント対応の未来像への共感</p>
    <p class="text-em-base text-gray-600 mt-2">実運用での検証にご協力いただけるチームを募集しています</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #3E9BA4;">
    <p class="text-em-lg font-bold text-gray-700">プロダクトのプロトタイプの活用</p>
    <p class="text-em-base text-gray-600 mt-2">実運用での検証にご協力いただき、フィードバックをいただきたいです</p>
  </div>
</div>


---

<!-- パターン2: セクション開始スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 2.現状と課題

---

<!-- パターン17: アイコン付きリスト -->

# 現状の課題

<p class="text-em-base text-gray-600">インシデント対応における問題点</p>

<div class="space-y-4 mt-6">
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">⏰</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">対応時間の長期化</p>
      <p class="text-em-base text-gray-600">専門知識を持つ担当者への依存度が高く、初動対応に時間がかかる</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">👤</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">属人化した対応</p>
      <p class="text-em-base text-gray-600">経験豊富なエンジニアしか一次対応ができない現状</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-2xl mr-4">📚</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">ナレッジの分散</p>
      <p class="text-em-base text-gray-600">過去の対応履歴やノウハウが体系的に蓄積されていない</p>
    </div>
  </div>
</div>

---

<!-- パターン27: 統計強調スライド -->

# これからの課題

<p class="text-em-base text-gray-600">今後予想される問題</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-100 rounded-xl p-6 text-center">
    <p class="text-em-3xl font-bold text-gray-700">運用と開発の分離</p>
    <p class="text-em-lg mt-2">専門知識の無いシステムの一次対応</p>
  </div>
  <div class="bg-gray-100 rounded-xl p-6 text-center">
    <p class="text-em-3xl font-bold text-gray-700">システムの増加</p>
    <p class="text-em-lg mt-2">システムの複雑化</p>
  </div>
</div>

<div class="mt-6 bg-gray-50 rounded-xl p-4">
  <p class="text-em-base text-gray-700 text-center">
    人的リソースだけでは対応しきれない未来が迫っている
  </p>
</div>

---

<!-- パターン31: 問いかけスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# 課題解決のビジョン


<div class="flex items-center justify-center mt-8">
  <div class="bg-white/10 rounded-xl p-20 text-center">
    <p class="text-em-xl">
      「誰でもインシデントの<br>一次対応ができる世界へ」
    </p>
    <p class="text-em-base mt-4 opacity-80">
      インシデント対応のパラダイムシフトを起こしたい
    </p>
  </div>
</div>

---

<!-- パターン6: 2カラム比較（Before/After） -->

# ビジョン実現への手段

<p class="text-em-base text-gray-600">AIエージェントによる変革</p>

<div class="grid grid-cols-2 gap-6 mt-6">
  <div class="bg-gray-50 rounded-xl shadow-lg p-6">
    <h2 class="text-em-xl font-bold mb-4 text-gray-700">従来の対応</h2>
    <ul class="text-em-lg space-y-2 text-gray-600">
      <li>・システムの開発者が調査</li>
      <li>・手動でのログ調査</li>
      <li>・経験に基づく判断</li>
      <li>・直列的な調査</li>
    </ul>
  </div>
  <div class="rounded-xl shadow-lg p-6" style="background: linear-gradient(135deg, #1B4565 0%, #3E9BA4 100%);">
    <h2 class="text-em-xl font-bold mb-4 text-white">AIエージェント活用</h2>
    <ul class="text-em-lg space-y-2 text-white">
      <li>・AIが調査する</li>
      <li>・自動ログ分析と要約</li>
      <li>・データに基づく提案</li>
      <li>・平行的な調査</li>
    </ul>
  </div>
</div>

---

<!-- パターン14: 縦3つステップ -->

# ソリューションの説明

<p class="text-em-base text-gray-600">インシデント調査AIエージェントの機能</p>

<div class="space-y-3 mt-6">
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">01</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">初動調査</p>
      <p class="text-em-base text-gray-600">インシデントから原因の仮説、関連システムのログを自動収集</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">02</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">原因の推定と要約</p>
      <p class="text-em-base text-gray-600">収集した情報を分析し、考えられる原因と対応策を提示</p>
    </div>
  </div>
  <div class="flex items-start bg-gray-50 rounded-lg p-4">
    <span class="text-em-xl font-bold text-gray-500 mr-4">03</span>
    <div>
      <p class="text-em-lg font-bold text-gray-700">調査ツールのナビゲーション</p>
      <p class="text-em-base text-gray-600">インシデントの原因究明に最適なツールを提示</p>
    </div>
  </div>
</div>

---

<!-- パターン2: セクション開始スライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# デモンストレーション

<p class="text-em-xl mt-4">実際の動作をご覧ください</p>

<p class="text-em-lg mt-6">インシデント調査AIエージェントの実演</p>

---

<!-- パターン16: タイムラインレイアウト -->

# 今後のロードマップ

<p class="text-em-base text-gray-600">開発と展開の計画</p>

<div class="space-y-2 mt-6">
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">Q1</div>
    <div class="flex-1 rounded p-3"
         style="background: linear-gradient(to right, #1B4565, #3E9BA4);">
      <p class="text-em-lg font-bold text-white">価値の検証とフィードバック収集</p>
    </div>
  </div>
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">Q2</div>
    <div class="flex-1 bg-gray-100 rounded p-3">
      <p class="text-em-lg font-bold text-gray-700">人とAIのハイブリット調査</p>
    </div>
  </div>
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">Q3</div>
    <div class="flex-1 bg-gray-100 rounded p-3">
      <p class="text-em-lg font-bold text-gray-700">調査範囲の拡張</p>
    </div>
  </div>
  <div class="flex items-center">
    <div class="w-20 text-em-lg font-bold text-gray-600">Q4</div>
    <div class="flex-1 bg-gray-100 rounded p-3">
      <p class="text-em-lg font-bold text-gray-700">インシデント駆動の調査</p>
    </div>
  </div>
</div>

---

<!-- パターン19: 強調パネル（左ボーダー付き） -->

# 最後にお願い

<p class="text-em-base text-gray-600">皆さまへのお願い事項</p>

<div class="grid grid-cols-1 gap-4 mt-6">
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #1B4565;">
    <p class="text-em-lg font-bold text-gray-700">フィードバックのお願い</p>
    <p class="text-em-base text-gray-600 mt-2">本日のデモをご覧いただき、ご意見やご懸念をお聞かせください</p>
  </div>
  <div class="bg-gray-50 rounded-lg shadow p-4 border-l-4" style="border-color: #3E9BA4;">
    <p class="text-em-lg font-bold text-gray-700">パイロット参加のご検討</p>
    <p class="text-em-base text-gray-600 mt-2">実運用での検証にご協力いただけるチームを募集しています</p>
  </div>
</div>

---

<!-- パターン28: 中央配置メッセージ -->

# まとめ

<p class="text-em-base text-gray-600">本日のメッセージ</p>

<div class="flex items-center justify-center mt-8">
  <div class="text-center">
    <p class="text-em-2xl font-bold text-gray-800">
      誰でもインシデントの<br>一次対応ができる未来を目指して
    </p>
    <p class="text-em-lg text-gray-500 mt-6">
      AIエージェントでインシデント対応のパラダイムシフトを起こす
    </p>
  </div>
</div>

---

<!-- パターン5: クロージングスライド -->
<!--
_backgroundImage: "linear-gradient(to right, #1B4565, #3E9BA4)"
_color: #fff
-->

# ご清聴ありがとうございました

<p class="text-em-xl mt-8">ご質問やフィードバックをお待ちしています</p>

<p class="text-em-base mt-4">戸田 麻陽</p>