---
layout: post
title: "@bitwarden/cli が消えた90分 ― Shai-Hulud第3波と1000万ユーザーの朝"
date: 2026-04-23 12:00:00 +0900
categories: [security, supply-chain]
tags: [npm, bitwarden, shai-hulud, supply-chain, teampcp]
---

> [!NOTE]
> **この記事について**
> 本記事は、セキュリティリサーチブログ [watchTowr Labs](https://labs.watchtowr.com/) のライティングスタイル（ユーモアと皮肉を交えた独特の文体）に影響を受けて書いたパロディ記事です。
> 技術的事実（タイムライン・侵入経路・IOC・公式声明）はすべて一次/二次ソースで裏取り済みですが、文体・演出・括弧内ツッコミなどは watchTowr Labs へのリスペクトを込めたオマージュです。
> 文末の脚注からソースに飛べます。

> [!IMPORTANT]
> **TL;DR（時間がない人向け）**
> - 2026年4月22日 21:57 UTC（17:57 ET）、`@bitwarden/cli@2026.4.0` が **トロイの木馬化された状態でnpmに公開**された
> - 約**90分後**（厳密には93分、メディアにより「約90分」「93分」「約1.5時間」と表記揺れ）の23:30 UTC（19:30 ET）に削除。修正版は `2026.4.1`
> - マルウェアは **Shai-Hulud キャンペーン第3波**。攻撃グループは **TeamPCP**
> - 起点は **Checkmarx KICS Docker image** の侵害。Hackreadによれば **Dependabotが自動でトロイ化イメージを取り込んだ** ことでBitwardenのCI/CDが汚染された（Aikidoは「Checkmarx侵害から得たアクセスを利用した可能性が高い (likely)」と慎重表現）
> - 標的は **GitHub/npm/SSH/AWS/Azure/GCP の認証情報、AI assistants（Claude Code/Cursor/Aider/MCP等）の設定**
> - **Bitwardenのvaultデータには影響なし**。本番システムも無事
> - 影響を受けた可能性のある開発者は推定 **334人**

---

## プロローグ：火曜日の夜、誰も気づかなかった90分強

> "no one was watching."
>
> ―― **Hackread**が引用する研究者の発言[^hackread]

2026年4月22日（火曜日）、米東部時間の午後5時57分。
ヨーロッパは深夜に入りかけ、アジアは水曜日の朝6時を迎えようとしていた。

その時刻、`npm install -g @bitwarden/cli` を打った世界中の開発者は、知らず知らずのうちに **AWS/GitHub/SSHのトークンとAI assistantsの設定ファイルを暗号化された箱に詰めて、攻撃者の管理する公開GitHubリポジトリにアップロードしていた**（複数の一次調査ベンダーの観測による）[^aikido][^paloalto][^sophos]。

およそ90分後（**厳密には93分後**：17:57 ET → 19:30 ET）、悪意あるパッケージは削除された。
しかし、その期間に `2026.4.0` を `npm install` してしまった人々に対して、OX Securityは「**自分のマシンと、自分が触れる全クレデンシャルが侵害されたものとして扱うべき**」と勧告している[^ox]。

これは、その90分強の間に何が起きたのかの記録だ。

そして、なぜそんなことが可能だったのかの記録でもある。

---

## 第1章：舞台 ―― 1000万ユーザーが信頼するBitwarden

Bitwardenをご存じだろうか。
オープンソースのパスワードマネージャーで、**1000万人以上の個人ユーザーと5万社以上の企業が利用している**[^gigazine]。

CLI版である `@bitwarden/cli` は、`bw login` `bw unlock` で認証情報をvaultから取り出し、シェルスクリプトやCI/CDから自動的に秘密情報を扱える。**npmで配布され、週間ダウンロード数は7万〜7.8万、月間で25万を超える**[^aikido][^sophos]。

開発者にとってBitwarden CLIは、「シェルから直接vaultを叩けるからCI/CDで使う」 ―― つまり**特権コンテキストで動く前提**のツールだ。当然、攻撃者にとっても旨味は最大化される。CLIを乗っ取れば、それを呼んでいるパイプラインのSecretsまで一気に取れる。

「特権コンテキストで動くツールを狙う」 ―― これは今回の攻撃が、ただの愉快犯ではなく、**サプライチェーン横展開を見据えた組織的な作戦**だと言える理由のひとつだ[^sophos]。

---

## 第2章：序曲 ―― `helloworm00` という名の足音（4月20日）

事件は4月22日のように見えるが、攻撃者の足跡はもう少し前から残っている。

GitGuardianの観測によれば、**2026年4月20日 20:41**に GitHubで `helloworm00/hello-world` という不審なリポジトリへのコミットが記録された[^gitguardian]（タイムゾーンの明示はGitGuardianの公開記事中になし。GitHubのcommit timestampは通常UTC基準）。

ユーザー名がいい。`helloworm00`。**「ハロー、ワーム、ゼロゼロ番」**。

中身は何の変哲もないように見える「最初のリポジトリ」風のものだったが、コミットメッセージに `beautifulcastle` という文字列が紛れ込んでいた。21日には2回目のコミット、リポジトリ名は `helloworm00/my-first-repo`。

これらのコミットは、後に **Shai-Huludペイロードが盗み出した認証情報の暗号化ダンプを「公開GitHubリポジトリに偽装したC2」として保存した痕跡**であることがわかる[^gitguardian][^aikido]。攻撃者は、自分が使う墓場を、被害が出る2日前から掘っていた。

しかしこの時点では、誰もそれが何の前触れか気づいていなかった。

---

## 第3章：トロイの木馬の組立 ―― Checkmarx KICSが侵入経路だった

ここで一度、舞台をBitwardenから離す。

実は、4月22日の朝までに **静的解析ツールベンダーCheckmarxの配布パイプラインが先に侵害されていた**。これがすべての発端だ[^cyberinsider][^itnews]。

Checkmarx KICS（Infrastructure-as-Codeセキュリティスキャナ）の **Docker Hubイメージが汚染された**。具体的に汚染されたタグは以下：

- `checkmarx/kics:v2.1.20`
- `checkmarx/kics:v2.1.21`
- `checkmarx/kics:debian`
- `checkmarx/kics:alpine`
- `checkmarx/kics:latest`

**累計500万プル超**[^sophos]。さらにCheckmarxのVS Code拡張2種、`ast-github-action` リポジトリのv2.3.35リリースまで汚染されていた。

これらの汚染版イメージは、内部に `mcpAddon.js` という名の追加ペイロードを仕込み、**起動時に約10MBの難読化済みJavaScriptを取得して実行する**設計になっていた[^cyberinsider]。

そして、ここからが真の地獄だ。

---

## 第4章：Dependabotという名の自動配達員（4月22日）

Bitwardenの `github.com/bitwarden/clients` リポジトリには、`publish-ci.yml` という公開ワークフローが存在する。CLIをビルドしてnpmにpublishする正規のCI/CDパイプラインだ。このパイプラインは、依存関係のひとつとして **Checkmarx KICSのDocker imageを使っていた**[^aikido]。

**ここから先は、Hackreadの分析[^hackread]を基にした再構成である。** AikidoやSophosなど他の一次調査ベンダーは「Checkmarx侵害から得たアクセスを利用した可能性が高い (likely)」とより慎重な表現にとどめており、Dependabot関与は確定情報ではない点に留意してほしい[^aikido]。

Hackreadによれば、Bitwardenのリポジトリは **Dependabotによる依存関係の自動更新**が有効になっており、4月22日、Dependabotは「Checkmarx KICSの新しいDocker imageが出ました」とPRを作った。Hackreadが引用する研究者の言葉では、「**no one was watching**」 ―― 誰も中身を見ていなかった、と表現されている[^hackread]。

Dependabotとは、GitHub純正の「親切なボット」だ。依存関係に新しいバージョンが出ると、自動でPRを作って更新を提案してくれる。多くのプロジェクトが「Dependabotのbumpはほぼ自動マージで通す」運用にしている。なぜなら、依存関係更新は退屈で、人間がレビューすると忘れるからだ。

その結果、Bitwardenの `publish-ci.yml` は、**信頼された自社CIの中で**、トロイ化されたCheckmarx KICSをpullし、build時にマルウェアコード（後の `bw1.js`）を `@bitwarden/cli` のパッケージに注入し、**正規のpublishフローでnpmにアップロード**した（CSO Online / BleepingComputerは経路を「compromised GitHub Action」とのみ表現[^bleeping][^cso]）。

GitHub ActionsのOIDC認証は完璧に通った。
署名も整合した。
**「Bitwarden公式」を名乗るに足る、すべての条件を満たした、しかしマルウェア入りのパッケージ**が、世界中のミラーに配信された。

---

## 第5章：90分強の世界（4月22日 21:57 UTC 〜 23:30 UTC）

```
2026-04-22 21:57:00 UTC  @bitwarden/cli@2026.4.0 が npm に公開
2026-04-22 23:30:00 UTC  Bitwardenが該当バージョンを deprecate / unpublish
```

期間は **17:57 ET から 19:30 ET（厳密には93分間）**。各メディアによって表記が揺れている：
- BleepingComputer / CSO Online: "approximately 1.5 hours"（約1.5時間）[^bleeping][^cso]
- Sophos: "about 90 minutes"（約90分）[^sophos]
- 一部記事タイトル: "93 minutes"（93分）

90分強。
それだけの時間。
しかし、それは `@bitwarden/cli` を週7万回ダウンロードしているコミュニティにとって、**少なくとも数百のCI/CDパイプラインと開発機が `2026.4.0` をpullするには十分すぎる時間**だった。

Techloyによれば、**この期間に影響を受けた可能性のある開発者は約334人**と推定されている[^techloy]（ただしTechloy自体も「研究者がnpmダウンロードメトリクスから推定」とのみ書いており、具体的な特定方法は明記されていない）。

研究者の言葉を借りれば、「`@bitwarden/cli@2026.4.0` を入れた**たった一人の開発者**が、所属組織のCI/CDパイプライン全体への侵入口になり得た」[^techloy]。

---

## 第6章：マルウェア解剖 ―― "Shai-Hulud: The Third Coming"

`@bitwarden/cli@2026.4.0` のtarballには、本来存在しないはずのファイルが2つ含まれていた[^aikido]。

### Stage 1: `bw_setup.js`

クロスプラットフォーム対応のブートストラッパー。
**Bun JavaScriptランタイムをダウンロードして起動**し、Stage 2を実行する。なぜBunか？ Node.js環境ではセキュリティ製品のフックが入っている可能性があるからだ。新しいランタイムを持ち込めば、既存のEDR/監視を回避できる ―― シンプルかつ効果的なロジックだ。

### Stage 2: `bw1.js`

**約10MBの重度に難読化されたJavaScript**。これが本体だ。

主な機能：

1. **認証情報窃取**:
   - `~/.ssh/` 配下の鍵
   - `~/.npmrc` のトークン
   - `.env` ファイル
   - `~/.aws/credentials` `~/.azure/` `~/.config/gcloud/`
   - `gh auth status` で取得できるGitHub CLIトークン
   - 環境変数全般、シェル履歴、GitHub Actions secrets
   - **AI assistants の設定ファイル**: Hackreadによれば Claude Code, Gemini CLI, Codex CLI, Kiro CLI, Aider, OpenCode、Sophosは加えて Cursor, MCPサーバ設定も挙げている[^hackread][^sophos]
   - JinaCoin / BeInCryptoの報道によれば、**暗号資産ウォレット（MetaMask、Solana系）関連データ**も標的に含まれていた可能性がある（一次調査ベンダーの報告書には明記なし、暗号資産系メディアの観測）[^jinacoin][^beincrypto]

2. **AES-256-GCMで暗号化**:
   - 暗号化されたペイロードは `results-{TIMESTAMP}-{ID}.json` の形式で保存される[^ox]

3. **C2（多重化）**:
   - **メイン**: `audit.checkmarx[.]cx:443/v1/telemetry`（IP: `94.154.172.43`）[^aikido][^sophos]
   - **フォールバック**: 公開GitHubリポジトリにcommitとして書き込み（`helloworm00`配下のリポジトリなど）[^gitguardian]
   - **副副**: Cloudflare Tunnel `safely-irc-weblogs-few[.]trycloudflare[.]com`[^gitguardian]

4. **自己増殖（ワーム動作）**:
   - 盗んだGitHubトークンで、**被害者がpublish権限を持つ全npmパッケージにバックドアを仕込む**[^paloalto][^aikido]
   - 盗んだトークンで悪意あるGitHub Actionsワークフローを被害者リポジトリに注入し、**永続化**[^sophos]

5. **回避**:
   - **OS言語がロシア語の場合は実行をスキップ**（OXの分析より）[^ox]
   - Hackreadによれば、シェル起動ファイル（`.bashrc` `.zshrc`）に潜伏し、AI assistantsの実行環境に介入する仕掛けまで観測されている[^hackread]

そして、攻撃者の遊び心 ―― あるいは自己顕示欲。
GitHubのコミットメッセージには **`LongLiveTheResistanceAgainstMachines`**、リポジトリ内のマーカー文字列には **`Shai-Hulud: The Third Coming`** が埋め込まれていた[^aikido][^ox]。

「機械への抵抗は永遠に」。
皮肉なことだ。彼らは機械学習ツール（Claude, Cursor, Aider…）の認証情報を盗もうとしているのに。

---

## 第7章：何が盗まれ、何が無事だったか

### 盗まれた可能性のあるもの

| カテゴリ      | 例                                                                     | 主要ソース |
| --------- | --------------------------------------------------------------------- | --- |
| Git/CI 認証 | GitHubトークン, Actions secrets, npmトークン                                  | Aikido / Sophos / BleepingComputer |
| クラウド      | AWS/Azure/GCPクレデンシャル, secret manager APIキー                            | Aikido / Sophos / Palo Alto |
| サーバ管理     | SSH鍵, `~/.ssh/known_hosts`                                            | Aikido / BleepingComputer |
| 開発環境      | `.env`, `.npmrc`, シェル履歴                                               | Aikido / Hackread |
| AIツール     | Claude Code, Gemini/Codex/Kiro CLI, Aider, OpenCode（Hackread）／ Cursor, MCP（Sophos） | Hackread / Sophos |
| 暗号資産      | MetaMask, Solana系ウォレットの関連データ（**暗号資産系メディアの報道**、一次調査ベンダーは未明記） | JinaCoin / BeInCrypto |

[^hackread][^sophos][^aikido][^paloalto][^bleeping][^jinacoin][^beincrypto]

### 無事だったもの（Bitwardenの公式声明より）

- **エンドユーザーのvaultデータ**（パスワード本体）
- **本番データ・本番システム**
- **Chrome拡張、デスクトップアプリ、Webアプリ、MCPサーバ**[^gigazine]

つまり、**Bitwardenを「パスワード保管庫」として使っていた1000万人のうち、ほぼ全員は今回の攻撃で実害を受けない**。
被害は、**この90分強の期間に `npm install -g @bitwarden/cli` を実行してしまった、ごく一部の開発者・CI/CDパイプライン**に集中している。

---

## 第8章：Bitwardenの公式対応

Bitwardenは、攻撃検知後きわめて迅速に動いた。

### 対応タイムライン

```
21:57 UTC (Apr 22)  malicious 2026.4.0 が npm に publish される
~23:30 UTC          Bitwardenが当該バージョンを deprecate / unpublish
                    (Techloyによれば検出はSocket / JFrog等の研究者[^techloy])
~24h以内 (Apr 23)   修正版 2026.4.1 をリリース、公式声明を公表
                    (※ 正確なタイミングは公式声明側で明示されていない。Linuxiac は
                       2026-04-24 に "Bitwarden Confirms..." 記事を公開[^linuxiac])
```

### 公式声明（要旨 ※二次引用）

LinuxiacやBleepingComputer等の主要メディアが共通して引用しているBitwardenのインシデント声明：

> "The investigation found no evidence that end user vault data was accessed or at risk, or that production data or production systems were compromised."
>
> （調査の結果、エンドユーザーのvaultデータがアクセスされたり危険に晒された証拠、および本番データ・本番システムが侵害された証拠は見つからなかった）

[^linuxiac][^cyberinsider][^bleeping]

> [!WARNING]
> 本記事執筆時点で、Bitwarden公式ブログから上記声明の直接URLを特定できなかった。引用は **Linuxiac / BleepingComputer / The Hacker News等の主要メディアによる二次引用**を経由している。最新かつ完全な公式声明は [bitwarden.com](https://bitwarden.com/blog/) を直接参照されたい。

### Bitwardenが推奨したアクション

`@bitwarden/cli@2026.4.0` を**この90分強の間にinstallした人**に対して：

1. **当該バージョンを直ちにアンインストール**（`npm uninstall -g @bitwarden/cli`）
2. **npm cacheをclear**（`npm cache clean --force`）
3. **2026.4.1（または2026.3.0以下）にdowngrade**
4. **影響を受けた可能性のある全クレデンシャルをローテーション**:
   - GitHub Personal Access Token / SSH鍵
   - npmトークン
   - AWS / Azure / GCPの長期クレデンシャル
   - 各種APIキー（特にAI assistants）
5. **GitHubリポジトリの監査**:
   - 不正なworkflowが追加されていないか
   - 不審な新規リポジトリが作成されていないか（`helloworm00` パターン）
6. **CI/CDのSecretsを全部洗い直す**

そして、これは公式声明には書かれていないが、**Bitwarden Vaultのマスターパスワード自体は変える必要がない**。なぜなら、攻撃者が盗もうとしたのは「Vaultを開ける鍵」ではなく、**「あなたが開発に使っているクレデンシャル」**だからだ。

---

## 第9章：影響を受けた334人 ―― そして、それ以上

Techloyによれば、**この期間中に `2026.4.0` をinstallした可能性のある開発者は約334人**と推定されている[^techloy]。Techloy記事自体も「研究者がnpmダウンロードメトリクスから推定」と書いており、**確定値ではなく推定値**である点に留意したい。

そして、この数字は氷山の一角に過ぎない可能性がある。

なぜなら、Aikido / Palo Altoによれば、Shai-Huludは**自己増殖型ワーム**であり、被害者がpublish権限を持つnpmパッケージに自動的にバックドアを仕込む設計になっているからだ[^aikido][^paloalto]。334人のうち1人でも `npm publish` 権限を持つメンテナがいれば、その人が管理する全パッケージが二次感染源になり得る。

Techloyが引用する研究者の警告：
> 「`@bitwarden/cli@2026.4.0` を入れたたった1人の開発者が、より広いサプライチェーンの侵入口になり得る」[^techloy]

だから、Bitwardenが93分で気づいて削除したことは賞賛に値するが、それで終わりではない。**その期間に漏れたクレデンシャルが、今後数週間〜数ヶ月にわたって別のサプライチェーン攻撃の起点になる可能性がある**。

---

## 第10章：Shai-Huludの系譜 ―― 第1波・第2波・そして第3波

「Shai-Hulud」とは、フランク・ハーバートのSF小説『Dune』に登場する砂漠の巨大砂虫の名前だ。砂漠の地中を泳ぎ、地表で生命を貪り、また砂の中に消えていく ―― この命名はあまりにも的確だ。

### 第1波・第2波について

Aikidoによれば、TeamPCPは「**2025年9月以降アクティブで、2026年に入って大幅に活動を拡大した**」[^aikido]。
過去の標的について、BleepingComputerは「**TeamPCP, the threat actor behind previous Trivy and LiteLLM supply chain compromises**」と表現している（Trivy = コンテナ脆弱性スキャナ、LiteLLM = LLM gateway）[^bleeping]。CSO Onlineも過去攻撃として「Checkmarx KICS and Trivy」を挙げている[^cso]。

> [!NOTE]
> 本記事では「Shai-Huludキャンペーン第1波・第2波の技術的詳細」までは網羅できていない。各波の侵入経路や具体的なペイロードの違いについては、Aikido / Palo Alto / OX Security等の各社レポートシリーズを参照されたい。

### 第3波 ―― **今回**（2026年4月）
- **Checkmarx と Bitwarden という、開発者ツール業界の中核ベンダー2社**を同時に侵害
- **AI assistantsの設定ファイル**を新たに標的に追加
- **Cloudflare Tunnel**による多段C2を採用（GitGuardian観測）[^gitguardian]
- **Dependabotという「信頼の自動化」を逆手に取った**侵入経路（Hackread分析）[^hackread]

[^aikido][^ox][^sophos]

各波で攻撃の洗練度が上がっている。**第4波が来ないと考える理由はない**。

---

## 第11章：教訓 ―― Dependabotに「冷却期間」を

このインシデントから読み取れる教訓は、技術的なものと運用的なものに分けられる。

### 技術的教訓

1. **CI/CDの依存関係は、信頼境界の外にある**
   ―― Checkmarxのような「セキュリティベンダー」のDocker imageでさえ侵害される時代だ
2. **`publish` 権限を持つCIは、盗まれた時の被害が最大化する**
   ―― npm 2FA / Sigstore / OIDCを使っていても、「CI自体」が侵害されたら止められない
3. **AI assistantsの設定が新たな標的になっている**
   ―― `~/.config/anthropic/` `~/.aider/` `~/.cursor/` 等は今後監視対象に入れるべき

### 運用的教訓

1. **Dependabotの自動マージに「冷却期間」をつける**
   ―― 新リリースから例えば72時間は自動マージを停止する。これだけで今回の事件は防げた可能性が高い[^hackread]
2. **CIに使うDocker imageはdigest固定する**
   ―― `:latest` `:debian` は今後も狙われる
3. **CI/CDのSecretsをShort-lived化する**
   ―― 長期credentialはCI internal storeから消す。OIDC + STS で都度発行
4. **公開GitHubリポジトリを定期スキャン**
   ―― `helloworm00` のような**自社組織配下に勝手に作られたリポジトリ**を検知できる仕組みを持つ

---

## エピローグ：93分の沈黙が語ること

Bitwardenは、**業界のお手本に近い対応**をした。
攻撃から検知まで約93分（21:57 → 23:30 UTC）。修正版 `2026.4.1` を24時間以内に提供（Linuxiacが24日に確認）[^linuxiac]。透明性のある公式声明。

それでも、**その93分間で世界中のCI/CDパイプラインに毒が回った**。

これが、2026年現在のサプライチェーン攻撃の現実だ。
「**信頼**」という単語は、もはや「**検証可能**」という単語に置き換わるべきフェーズに入っている。Bitwardenを信頼するのではない。Bitwardenが配布したパッケージのhashを検証する。Dependabotを信頼するのではない。Dependabotが提案した変更を、人間か別の自動化が検証する。

そしておそらく、来週か再来月か、**Shai-Hulud第4波**がまた来る。
そのときには、今度こそ準備しておこう。

---

## 付録A：IOC（Indicators of Compromise）

| 種類                 | 値                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------- |
| 悪意あるパッケージ          | `@bitwarden/cli@2026.4.0`                                                                 |
| 悪意あるDockerイメージ     | `checkmarx/kics:v2.1.20`, `:v2.1.21`, `:debian`, `:alpine`, `:latest`                     |
| 悪意あるVS Code拡張      | Checkmarxの2種類のVS Code拡張（バージョン未特定）                                                         |
| 悪意あるGitHub Actions | `Checkmarx/ast-github-action@v2.3.35`                                                     |
| マルウェアファイル名         | `bw_setup.js`, `bw1.js`, `mcpAddon.js`                                                    |
| C2ドメイン             | `audit.checkmarx[.]cx`                                                                    |
| C2 IP              | `94.154.172.43`                                                                           |
| Cloudflare Tunnel  | `safely-irc-weblogs-few[.]trycloudflare[.]com`                                            |
| 攻撃者GitHubアカウント     | `helloworm00`                                                                             |
| マーカー文字列            | `Shai-Hulud: The Third Coming`, `LongLiveTheResistanceAgainstMachines`, `beautifulcastle` |
| ペイロード保存形式          | `results-{TIMESTAMP}-{ID}.json` (AES-256-GCM)                                             |
| 攻撃グループ             | TeamPCP（**BleepingComputer**によれば過去のTrivy/LiteLLM攻撃も同一グループ。Aikidoは「2025年9月以降アクティブ」と表現） |

---

## 付録B：影響を受けたもの・受けなかったもの

### 影響を受けたもの

- `@bitwarden/cli@2026.4.0` を `npm install` した開発者・CI/CD（**Techloy推定で約334人**）
- 当該マシンの `~/.ssh/`, `.env`, `.npmrc`, AWS/Azure/GCPクレデンシャル（Aikido / Sophos / BleepingComputer）
- AI assistants（Claude Code, Cursor, Aider, MCP等）のAPIキー・設定（Hackread / Sophos）
- 暗号資産ウォレット関連データ（MetaMask, Solana系）（**JinaCoin / BeInCryptoの報道**、一次調査ベンダーは未明記）
- Checkmarx KICSのDocker imageを使っていたCI/CD（**Sophosによれば累計500万プル超**）

### 影響を受けなかったもの

- **Bitwardenのvaultデータ**（パスワード本体）
- **Bitwardenの本番システム / 本番データ**
- Bitwarden Chrome拡張、デスクトップアプリ、Webアプリ、MCPサーバ
- `2026.4.0` 以外のバージョン（`2026.3.x` 以下、`2026.4.1` 以降）
- ロシア語OS環境（**OXによれば**マルウェアが意図的にスキップ）

---

## 参考文献・一次ソース

### 一次調査・ベンダーレポート

- **Palo Alto Networks Unit 42**: [Bitwarden CLI Impersonation Attack Steals Cloud Credentials and Spreads Across npm Supply Chains](https://www.paloaltonetworks.com/blog/cloud-security/bitwardencli-supply-chain-attack/)
- **Aikido Security**: [Is Shai-Hulud Back? Compromised Bitwarden CLI Contains a Self-Propagating npm Worm](https://www.aikido.dev/blog/shai-hulud-npm-bitwarden-cli-compromise)
- **OX Security**: [Shai-Hulud: The Third Coming — Bitwarden CLI Backdoored in Latest Supply Chain Campaign](https://www.ox.security/blog/shai-hulud-bitwarden-cli-supply-chain-attack/)
- **GitGuardian**: [@bitwarden/cli - GitGuardian Views on helloworm00](https://blog.gitguardian.com/bitwarden-cli-gitguardian-views-on-helloworm00/)
- **Sophos X-Ops**: [Supply chain attacks hit Checkmarx and Bitwarden developer tools](https://www.sophos.com/en-us/blog/supply-chain-attacks-hit-checkmarx-and-bitwarden-developer-tools)
- **CyberInsider**: [Bitwarden CLI backdoored in Checkmarx supply chain attack](https://cyberinsider.com/bitwarden-cli-backdoored-in-checkmarx-supply-chain-attack/)

### Bitwarden公式声明（二次引用元）

- **Linuxiac**: [Bitwarden Confirms Short-Lived npm Compromise Affecting CLI Package](https://linuxiac.com/bitwarden-confirms-short-lived-npm-compromise-affecting-cli-package/)
- **The Hacker News**: [Bitwarden CLI Compromised in Ongoing Checkmarx Supply Chain Campaign](https://thehackernews.com/2026/04/bitwarden-cli-compromised-in-ongoing.html)

### 主要報道

- **BleepingComputer**: [Bitwarden CLI npm package compromised to steal developer credentials](https://www.bleepingcomputer.com/news/security/bitwarden-cli-npm-package-compromised-to-steal-developer-credentials/)
- **Forbes**: [Bitwarden Confirms Compromise — Here Are The Facts For 10 Million Users](https://www.forbes.com/sites/daveywinder/2026/04/24/bitwarden-confirms-compromise-here-are-the-facts-for-10-million-users/)
- **SecurityWeek**: [Bitwarden NPM Package Hit in Supply Chain Attack](https://www.securityweek.com/bitwarden-npm-package-hit-in-supply-chain-attack/)
- **Hackread**: [TeamPCP Hijacks Bitwarden CLI, Uses Dependabot to Deploy Shai-Hulud Malware](https://hackread.com/teampcp-bitwarden-cli-dependabot-shai-hulud-malware/)
- **Techloy**: [Bitwarden CLI Compromised in 'Shai-Hulud' Supply Chain Attack; 334 Developers Exposed](https://www.techloy.com/bitwarden-cli-compromised-in-shai-hulud-supply-chain-attack-334-developers-exposed/)
- **CSO Online**: [Bitwarden CLI password manager trojanized in supply chain attack](https://www.csoonline.com/article/4162865/bitwarden-cli-password-manager-trojanized-in-supply-chain-attack.html)
- **Cybernews**: [Bitwarden CLI tool compromised: hundreds of developers pull credential-stealing malware](https://cybernews.com/security/bitwarden-cli-npm-package-compromised-with-malware/)
- **iTnews**: [Checkmarx-style supply chain attack hits password manager Bitwarden](https://www.itnews.com.au/news/checkmarx-style-supply-chain-attack-hits-password-manager-bitwarden-625331)

### 日本語報道

- **GIGAZINE**: [パスワードマネージャーのBitwardenがサプライチェーン攻撃を受ける、npmパッケージを使っていた人は要確認](https://gigazine.net/news/20260424-bitwarden-cli-supply-chain-attack/)
- **BeInCrypto**: [Bitwarden CLIへのサプライチェーン攻撃で暗号資産ウォレット鍵が危険](https://jp.beincrypto.com/bitwarden-cli-supply-chain-attack-crypto/)
- **JinaCoin**: [メタマスクやソラナ系ウォレットが標的に──パスワード管理Bitwardenにマルウェア混入](https://jinacoin.ne.jp/crypto-wallet-malware-bitwarden-20260424/)
- **WEEX**: [Bitwarden CLIがサプライチェーン攻撃を受け、悪意のあるパッケージが約1.5時間流通していました。](https://www.weex.com/ja/news/detail/bitwarden-cli-suffered-a-supply-chain-attack-with-a-malicious-package-circulating-for-about-15-hours-708137)

---

[^paloalto]: [Palo Alto Networks Unit 42 - Bitwarden CLI Impersonation Attack](https://www.paloaltonetworks.com/blog/cloud-security/bitwardencli-supply-chain-attack/)
[^aikido]: [Aikido Security - Is Shai-Hulud Back?](https://www.aikido.dev/blog/shai-hulud-npm-bitwarden-cli-compromise)
[^ox]: [OX Security - Shai-Hulud: The Third Coming](https://www.ox.security/blog/shai-hulud-bitwarden-cli-supply-chain-attack/)
[^gitguardian]: [GitGuardian - GitGuardian Views on helloworm00](https://blog.gitguardian.com/bitwarden-cli-gitguardian-views-on-helloworm00/)
[^sophos]: [Sophos - Supply chain attacks hit Checkmarx and Bitwarden](https://www.sophos.com/en-us/blog/supply-chain-attacks-hit-checkmarx-and-bitwarden-developer-tools)
[^cyberinsider]: [CyberInsider - Bitwarden CLI backdoored](https://cyberinsider.com/bitwarden-cli-backdoored-in-checkmarx-supply-chain-attack/)
[^itnews]: [iTnews - Checkmarx-style supply chain attack](https://www.itnews.com.au/news/checkmarx-style-supply-chain-attack-hits-password-manager-bitwarden-625331)
[^linuxiac]: [Linuxiac - Bitwarden Confirms Short-Lived npm Compromise](https://linuxiac.com/bitwarden-confirms-short-lived-npm-compromise-affecting-cli-package/)
[^bleeping]: [BleepingComputer - Bitwarden CLI npm package compromised](https://www.bleepingcomputer.com/news/security/bitwarden-cli-npm-package-compromised-to-steal-developer-credentials/)
[^techloy]: [Techloy - 334 Developers Exposed](https://www.techloy.com/bitwarden-cli-compromised-in-shai-hulud-supply-chain-attack-334-developers-exposed/)
[^hackread]: [Hackread - TeamPCP Hijacks Bitwarden CLI via Dependabot](https://hackread.com/teampcp-bitwarden-cli-dependabot-shai-hulud-malware/)
[^gigazine]: [GIGAZINE - Bitwarden CLIサプライチェーン攻撃](https://gigazine.net/news/20260424-bitwarden-cli-supply-chain-attack/)
[^jinacoin]: [JinaCoin - メタマスクやソラナ系ウォレットが標的](https://jinacoin.ne.jp/crypto-wallet-malware-bitwarden-20260424/)
[^weex]: [WEEX - Bitwarden CLI がサプライチェーン攻撃 (1.5時間流通)](https://www.weex.com/ja/news/detail/bitwarden-cli-suffered-a-supply-chain-attack-with-a-malicious-package-circulating-for-about-15-hours-708137)
[^cso]: [CSO Online - Bitwarden CLI password manager trojanized in supply chain attack](https://www.csoonline.com/article/4162865/bitwarden-cli-password-manager-trojanized-in-supply-chain-attack.html)
[^beincrypto]: [BeInCrypto - Bitwarden CLI攻撃が暗号資産ウォレットシークレットを脅かす](https://jp.beincrypto.com/bitwarden-cli-supply-chain-attack-crypto/)
