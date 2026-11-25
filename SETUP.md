# SETUP.md

このテンプレートから新しいプロジェクトを作る手順と、その後の進め方。このファイルは新しいプロジェクトにはコピーしない。

## 0. 準備

使うテンプレートの版が、このリポジトリにコミットされ、push されていることを確かめる。そのコミットとこのリポジトリの URL は、手順5で最初のコミットのメッセージに書く。

## 1. Lake のプロジェクトを作る

```bash
lake +leanprover/lean4:v4.35.0 new <Project> math
cd <Project>
```

このコマンドで Lean と mathlib がともに 4.35.0 のプロジェクトを作る。

## 2. 生成されたファイルを直す

- `lakefile.toml` の `[[lean_lib]]` に `globs = ["<Project>.*"]` を足す。これで入口の `<Project>.lean` と、そこからまだ import していないものも含めた `<Project>/` 以下の全モジュールがビルド対象になる。
- `.github/workflows/` の `lean_action_ci.yml` を消す（テンプレートの `ci.yml` と重複する）。`update.yml` はルートの mathlib しか更新せず、ルートと `Blueprint/` で版を揃える規則と合わないので、消すか両方を更新するように直す。`create-release.yml` は要らなければ消す。
- テンプレートの `.gitignore` の行を、生成された `.gitignore` に足す。生成された `/.lake` はルートの `.lake` にしか当たらない。
- `<Project>/Basic.lean` を `Statement.lean` と `Main.lean` に置き換え、`<Project>.lean` の import を合わせる。
- `README.md` を書き直す。

## 3. テンプレートのファイルを置く

- `AGENTS.md`・`ARCHITECTURE.md`・`DECISIONS.md` をルートに置く。コピー先の `DECISIONS.md` は、日付付きの記録を消し、冒頭の記入規則と書式例だけを残す。

- CI の設定は、使うサービスの方だけを置く：GitHub なら `.github/workflows/ci.yml`、GitLab なら `.gitlab-ci.yml`。GitHub Pages は Settings → Pages → Source を GitHub Actions にする。
- `Blueprint/lakefile.toml` を置き、ルートと同じ `lean-toolchain` を `Blueprint/` にも置く。`<Project>Blueprint.lean`・`<Project>BlueprintMain.lean`・`<Project>Blueprint/` は、Verso Blueprint の `project_template` を参考に作る。

## 4. 置き換えと記入

- `<Project>`・`<repo>`・`<話題>`・`<CI の設定ファイル>` を、すべてのファイルで実際の値に置き換える。
- GitHub の既定ブランチ名が `main` 以外なら、`ci.yml` の `branches` と `refs/heads/main` の指定をその名前に合わせる。
- `ARCHITECTURE.md` の「固定する規約」と「配置と名前空間」の表を埋める。

## 5. 確かめる

```bash
lake exe cache get && lake build
cd Blueprint && lake update && lake exe cache get && lake build <Project>Blueprint && lake exe vbp build && lake exe vbp check
```

- `Blueprint/lake-manifest.json` の mathlib のリビジョンが、ルートの `lake-manifest.json` と一致することを確かめる。
- 最初のコミットのメッセージに、テンプレートの URL とコミットを書く。

  ```text
  Initialize from lean-formalization-template

  Template: <template-url> at <hash>
  ```

- 最初の CI で、利用するサービスの `check` と `blueprint` が成功することを確かめ、結果を `DECISIONS.md` に記録する。

## 6. 設定の後の進め方

1. 原稿で、定義・規格化・主定理の主張を固める。証明は概略でよい。定義と規格化はすべての定理に波及するので、ここが揺れると手戻りが大きい。
2. `Statement.lean` に、主定理が使う定義と主定理の文を書き、原稿と照合する。Lean で決めた型や規格化を、`ARCHITECTURE.md` の「固定する規約」に書く。
3. Blueprint に、主定理までの補題と定義をノードとして書き、`uses` で依存を、`owner`・`priority` で分担を示す。これがロードマップになり、進捗は Lean から自動で集計される。
4. 作業単位に着手するとき、複数のセッションにまたがるものだけ `plans/<作業名>.md` に計画を作る。全体の計画を前もって書く必要はない。

### 論文と形式化を並行して進める場合

原稿にまだない定理を足すときは、次の順に進める。

1. 新しい定理と、前提となる補題・定義を、Blueprint のノードとして足す（研究者）。
2. 定義と主張だけを Lean で書き、指定した補題の証明を `sorry` にしたまま、主定理をそれらの補題から証明する。これで、定義が型として成り立つこと、補題の主張が足りていること、依存の組み方を確かめる。研究者は主張を Blueprint のノードと照合する。
3. 主要な定義には、値が分かっている場合や境界の場合を確かめる補題を添え、定義の誤りを早く見つける。
4. 補題を下から順に証明する。次に着手できるものは `lake exe vbp query work-queue` で分かる。
5. 原稿の本文を書いたり改訂したりしたら、影響する定理文を照合し直す。

完成したかどうかは、主定理の `#print axioms` と CI の `check` で判断する。`sorry` の警告ではビルドは失敗しない。
