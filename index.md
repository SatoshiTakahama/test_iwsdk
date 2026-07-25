---
layout: default
title: Immersive Web SDKによるWebXR開発
---

<div class="page-layout" markdown="1">

  <!-- 目次エリア -->
  <nav id="toc-container" class="toc-sidebar" markdown="1">

## 目次
* TOC
{:toc}

  </nav>

  <!-- 本文エリア -->
  <main class="content-body" markdown="1">

Immersive Web SDKによるWebXR開発について調査した内容について記載しています。

## Immersive Web SDK関連ページへのリンク
- [Immersive Web SDKのGitHub Pages](https://iwsdk.dev/)
- [Immersive Web SDKのGitHub](https://github.com/facebook/immersive-web-sdk)
- [MetaによるImmersive Web SDKの紹介記事1](https://developers.meta.com/horizon/blog/immersive-web-sdk-new-era-spatial-web-development/)
- [MetaによるImmersive Web SDKの紹介記事2](https://developers.meta.com/horizon/blog/accelerate-vr-development-with-ai-and-immersive-web-sdk/)

## ビルド済みページへのリンク
Immersive Web SDKで作成したプロジェクト雛形をビルドしたページへのリンクです。
- [Virtual Reality](./dist-examples/vr/index.html)
- [Augmented Reality](./dist-examples/ar/index.html)

## 開発Tips
自分がImmersive Web SDKで作業したときに対応した内容を紹介します。
2026年7月時点での情報です。
### AIエージェント
Immersive Web SDKのプロジェクト雛形を生成するときに、連携するAIエージェントを選択できます。
複数設定も可能で選択したAIエージェント向けのファイルが追加されます。
自分はClaude Codeを選択しており、windows上でLM Studio + qwen3.6-35b-a3bと組み合わせて使用しています。

自分の環境では、LM Studioの応答が遅すぎるためにClaude Codeが処理を途中で強制終了することが頻発しました。
settings.jsonに以下の設定を追加することで対策しています。
```
  "env": {
    "ANTHROPIC_BASE_URL": "http://localhost:1234",
    "ANTHROPIC_AUTH_TOKEN": "lmstudio",
    "CLAUDE_CODE_ATTRIBUTION_HEADER": "0",
    "API_TIMEOUT_MS": "3600000",
    "CLAUDE_ASYNC_AGENT_STALL_TIMEOUT_MS": "3600000",
    "BASH_DEFAULT_TIMEOUT_MS": "1800000",
    "BASH_MAX_TIMEOUT_MS": "3600000",
    "CLAUDE_ENABLE_STREAM_WATCHDOG": "0",
    "API_FORCE_IDLE_TIMEOUT": "0"
  }
```

現時点ではLM StudioのAnthropic互換APIがテキスト生成のみ対応していることが原因で、
画面キャプチャ(browser_screenshot)を実行するとエラーが発生する制限事項があります。

### トラブル対応
#### `npm run dev`で起動したときにブラウザからhttps://localhost:8081/でアクセスできない
- 原因
自分の環境では8081は別の用途で使用されており競合発生していた。
- 対策
`vite.config.js`のポート設定を競合しない番号に変更した。
```
server: { host: "0.0.0.0", port: 8081 },
これを以下に変更
server: { host: "0.0.0.0", port: 18081 },
```

#### `npm run build` + `npm run preview` で XR セッションが開始できない
- 原因
`vite.config.ts` の`iwsdkDev`プラグインは `injectOnBuild: false`（デフォルト値）に設定されているため、`npm run build`時にIWER (Immersive Web Emulation Runtime)がHTMLに注入されない。
- 対策
`vite.config.ts`のemulatorオプションに`injectOnBuild:true`を設定。
```
iwsdkDev({
  emulator: {
    device: "metaQuest3",
    injectOnBuild: true,  // ← これを追加
  },
  ai: { mode: "agent" },
  verbose: true,
}),
```

#### `npm run build`を実行すると`metaspatial/components/` ディレクトリ内のファイル（`IWSDKPhysicsBody.xml` など）が消失する
- 原因
`vite.config.ts`で使われている `discoverComponents`プラグインのデフォルト設定`clean: true`が原因。
- 対策
`vite.config.ts`で`clean: false`を設定する。
```
discoverComponents({
  outputDir: "metaspatial/components",
  include: /\.(js|ts|jsx|tsx)$/,
  exclude: /node_modules/,
  verbose: false,
  clean: false,  // ← これを追加
}),
```

