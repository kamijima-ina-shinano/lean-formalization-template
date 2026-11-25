# ARCHITECTURE.md

ファイル構成と、Lean で固定した規約を記す。現在の状態だけを書き、理由は DECISIONS.md に書く。import の関係が三層以上になったり分岐・合流したりするときは、Mermaid の `flowchart` で示す。

## ファイル構成

```text
<repo>/
├── AGENTS.md, ARCHITECTURE.md, DECISIONS.md
├── README.md                     # 公開用の案内
├── lakefile.toml, lake-manifest.json, lean-toolchain
├── <CI の設定ファイル>
├── <Project>/
│   ├── Statement.lean            # 主定理が使う定義と、主定理の文
│   ├── Main.lean                 # 主定理の証明
│   └── <話題>/                   # 証明の本体
├── Blueprint/                    # 独立した Lake パッケージ。`path = ".."` で <Project> に依存する
│   ├── lakefile.toml, lake-manifest.json, lean-toolchain
│   ├── <Project>Blueprint.lean       # 章をまとめ、依存グラフと進捗の要約を置く
│   ├── <Project>BlueprintMain.lean   # 生成の入口。`lake exe vbp build` はこの名前で探す
│   └── <Project>Blueprint/           # 章
├── Paper/                        # 原稿
└── plans/                        # 作業中の計画（Git の管理外）
```

## 固定する規約

| 項目 | 内容 |
| --- | --- |
| 定理文 | `Statement.lean` は mathlib だけを import し、主定理が使う定義と、主定理の文（`Prop` の定義）を置く。ほかのファイルはこれらの定義をそのまま使い、複製しない |
| 定義と規格化 | |
| 型とパラメータの範囲 | |

## 配置と名前空間

| 配置 | 役割 | 名前空間 |
| --- | --- | --- |
| `<Project>/` | | `<Project>` |

下位の名前空間は、ファイルの配置ではなく数学的な話題で分ける。汎用の層は、対象に固有の層を import しない。

## CI

既定のブランチへの push のうち Lean と Lake の設定を変えるものと、手動の実行で動く。

| job | 内容 |
| --- | --- |
| lean | ルートのパッケージをビルドする |
| check | `lake check` で、カーネルによる再検査と公理の確認をする |
| blueprint | Blueprint をビルドし、サイトを生成して検査する |
| 公開 | 既定のブランチで lean と blueprint が成功した後、サイトを Pages に公開する |
