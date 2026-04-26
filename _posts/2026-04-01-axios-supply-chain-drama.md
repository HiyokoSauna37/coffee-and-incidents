---
layout: post
title: "npm install したら北朝鮮が来た話 ―― あるいは、月曜日の朝に世界が終わるとはこういうことだ"
date: 2026-04-01 12:00:00 +0900
categories: [security, supply-chain]
tags: [npm, axios, dprk, lazarus, supply-chain]
---

> [!NOTE]
> **この記事について**
> 本記事は、セキュリティリサーチブログ [watchTowr Labs](https://labs.watchtowr.com/) のライティングスタイル（ユーモアと皮肉を交えた独特の文体）に影響を受けて書いたパロディ記事です。
> 技術的事実はすべて一次ソースに基づいていますが、文体・演出・括弧内ツッコミなどは watchTowr Labs へのリスペクトを込めたオマージュです。
>
> 技術的に正確な情報だけが必要な方はこちら → **[週1億DLのnpmパッケージaxiosが侵害された日——2026年3月31日 サプライチェーン攻撃の全記録](https://zenn.dev/hiyoko_sauna/articles/4e2c1162224752)**

> [!WARNING]
> **事実確認済み**
> この記事はフィクションではない。すべての技術的事実は確認済みだ。だからこそしんどい。

---

椅子を引いて座ってほしい。

お茶でもコーヒーでも、何か飲み物を手元に用意してほしい。アルコールでも構わない。むしろアルコールの方がいいかもしれない。

2026年3月31日、月曜日の朝に何が起きたか、これから話す。

我々はこの記事を書きながら、何度かため息をついた。笑った。また溜め息をついた。

あなたも同じような気持ちになると思う。

---

## プロローグ：祝・月曜日（という名の地獄への入り口）

少し想像してほしい。

月曜日の朝、あなたはコーヒーを飲みながらターミナルを開く。新しいプロジェクト、新しい週、新しい希望。`npm install` を実行する。

そのたった1行が、あなたのマシンを北朝鮮に開放していたかもしれない。

月曜日が台無しだ。

axiosをご存じだろうか。まあご存じだろう。週あたりのダウンロード数は[約1億件](https://www.npmjs.com/package/axios)。Wizによるとクラウド環境の約80%に存在する。[^wiz] JavaScriptを書いたことがある人間なら、`import axios from 'axios'` と書いた回数はおそらく自分の年齢より多い。

つまりこれは、インターネットの酸素だ。

酸素が毒になった。

それが、2026年3月31日に起きたことだ。

---

## 第一章：侵入 ―― 「メールアドレスを変えるだけで世界を支配できる」という事実

strap in。

攻撃の起点は、axiosの主要メンテナ `jasonsaayman` のnpmアカウントだった。

攻撃者がまず何をしたか。

**メールアドレスを変えた。**

以上だ。（以上ではないが、まずここから始める必要がある）

登録メールを `ifstap@proton.me`（ProtonMailですよ）に書き換え、**盗んだnpmアクセストークンで直接パッケージを公開した**。[^aikido]

GitHub Actions OIDCによる正規のリリースフロー？ スキップ。コードレビュー？ スキップ。対応するコミット？ タグ？ リリースノート？ 全部スキップ。

GitHubリポジトリには何も残らない。axiosのソースには何も変わっていない。記録も、履歴も、証拠も何もない。

Sigh.

ついでに攻撃者は `jasonsaayman` の **GitHubアカウントにも侵入し、axios宛に提出されていたセキュリティレポートを削除した**。[^bleeping]

証拠隠滅まで完璧だ。

We do not envy Jason Saayman. 本当に。

さらに `nrwise`（メール: `nrwise@proton.me`）という別のnpmアカウントが、悪意あるパッケージを事前に用意していた。[^stepsecurity]

両方ProtonMailだ。無関係に、コメントなしに申し上げておくと ―― ProtonMailは別に悪くない。ただ「攻撃者はProtonMailが好き」という傾向はある。

---

## 第二章：準備 ―― 「日曜日から仕込んでいた」という事実が我々をさらに傷つける

3月30日、日曜日の午後2時57分（JST）。

あなたが週末の昼寝をしていた頃、攻撃者は `plain-crypto-js@4.2.0` という無害なパッケージをnpmに公開していた。[^stepsecurity]

**無害なやつを、わざわざ。**

なぜか。アカウントの「信頼スコア」を積み上げるためだ。「このアカウントは真面目な開発者ですよ」という実績を事前に作るためだ。

18時間後、本番を差し込んだ。

| 時刻 (JST) | 出来事 |
|---|---|
| 3/30 14:57 | `plain-crypto-js@4.2.0`（おとり）を公開 |
| 3/31 08:59 | `plain-crypto-js@4.2.1`（本物）を公開 |
| 3/31 09:21 | `axios@1.14.1` 公開。感染窓、開く |
| 3/31 10:00 | `axios@0.30.4` も公開。念には念を |
| 3/31 ~12:15 | npmが削除 |
| 3/31 12:25 | セキュリティホールド適用 |
| 3/31 13:26 | スタブ公開、完全封じ込め |

**セキュリティホールドとは**: npmが悪意あるパッケージを封じ込めるメカニズム。npmセキュリティアカウントが無害なスタブ（`0.0.1-security.0`）を公開し、以降 `npm install plain-crypto-js` を実行するとセキュリティ通知が返るようになる。[^stepsecurity]

`axios@1.14.1` の感染窓は約2時間53分。週1億DLのライブラリにとってその間に何が起きていたか ―― あえて計算はしない。[^stepsecurity]

Wizは「影響を受けた環境の約3%で実際の実行が観測された」と言っている。[^wiz_note] 3%。週1億DLの3%。

これについては何も言わない。数字をそのまま置いておく。

---

## 第三章：仕掛け ―― 「axiosのコードには悪意が1行もない」という、むしろ怖い事実

「axiosのソースコードに悪意ある変更はない」

これは褒め言葉ではない。

攻撃者が加えた変更はたった1つ ―― `package.json` へのこの1行だ：[^aikido]

```json
"plain-crypto-js": "^4.2.1"
```

このパッケージはaxiosのソース全体で **一度もインポートされていない**。存在意義はただひとつ ―― **`postinstall` フックを実行すること**。

`npm install` が走ると、npmは「あ、postinstallスクリプトあるね」と判断して自動実行する。そういう仕様だ。

`setup.js` は難読化されていた。逆順Base64、修正されたパディング文字、XOR暗号（キー: `OrDeR_7077`、定数: `333`）の組み合わせで。[^snyk]

キー名 `OrDeR_7077` について、我々はしばらく黙って画面を見つめた。

そして実行後、このスクリプトは自分自身を消した。3ステップで：[^thn]

1. `setup.js`（postinstallスクリプト）を削除
2. postinstallフックを参照している `package.json` を削除
3. あらかじめパッケージに `package.md` という名前で同梱しておいたクリーンなマニフェストを `package.json` にリネーム

最後の手順が好きだ（好きではない）。

事前に「クリーンなpackage.json」を `package.md` という名前で同梱しておき、犯行後にすり替える。これを誰かが事前に設計した。設計書に書いた。

我々はこれを発見したとき、「neat」と言った。それ以外の言葉が出なかった。

事後に `node_modules/plain-crypto-js/` を確認しても `setup.js` は消えている。ネットワークログやEDRテレメトリなしでは検出がほぼ不可能だ。[^elastic]

---

## 第四章：RAT ―― 「設計書から作ったんだろうな」と言うしかないクロスプラットフォーム対応

このRATについて話す前に、Elasticの一言を引用しておく：

> 「この一貫性は、単一の開発者または密接に連携したチームが**共通の設計書から作業していた**ことを強く示唆する」[^elastic]

設計書。攻撃者に設計書がある。

……。

気を取り直して続ける。

macOS・Windows・Linuxの3プラットフォームに対して、**同一のC2プロトコルと同一のコマンド体系**を持つ実装が用意されている。

### C2通信仕様

| 項目 | 値 |
|---|---|
| トランスポート | HTTP POST |
| エンコーディング | Base64エンコードJSON |
| User-Agent | `mozilla/4.0 (compatible; msie 8.0; windows nt 5.1; trident/4.0)` |
| ビーコン間隔 | 60秒 |
| C2 | `sfrclak[.]com:8000` / IP: `142[.]11[.]206[.]73` |
| プラットフォーム識別 | macOS→`packages.npm.org/product0`、Windows→`product1`、Linux→`product2`[^thn] |

[^elastic]

`packages.npm.org` というホスト名でnpm正規トラフィックを偽装している。

ここでいったん、あなたのネットワーク監視ツールに話しかけたい。

`packages.npm.org` をブロックしたら何が起きるか考えてみよう。大丈夫、我々も考えたくない。

### プラットフォーム別ペイロード

**macOS: C++製 "macWebT"**[^bleeping]

`/Library/Caches/com.apple.act.mond` にドロップされる。`com.apple.act.mond`。Appleのキャッシュデーモンっぽい名前だ。[^elastic]

Gatekeeperを通過するために `codesign --force --deep --sign` でad-hocコード署名する。[^snyk]

これについては一言だけ言わせてほしい。

Gasp.

**Windows: VBScript、PowerShell、そしてWindowsTerminal詐称**

VBScriptが `.ps1` をダウンロードし、PowerShellインタープリタを `%PROGRAMDATA%\wt.exe` にコピーして実行する。[^snyk]

`wt.exe`。Windows Terminalの実行ファイル名だ。

さらに `%PROGRAMDATA%\system.bat` という download cradle を作成し、ログインのたびにC2からRATを再フェッチする。レジストリ Run キーに `MicrosoftUpdate` という名前で登録する。[^thn][^elastic]

`MicrosoftUpdate`、`wt.exe`、`system.bat`。

Some would say this is disingenuous.

**Linux: Pythonで簡潔に**

`/tmp/ld.py` を `nohup python3` で常駐。シンプルだが**同一のC2プロトコルで動く**。[^elastic]

### コマンド体系

- `kill`: RATを終了。ただしWindowsでは持続性が残る。killで終わらない
- `runscript`: コマンド実行（Windows: PowerShell、macOS: AppleScript、Linux: shell）
- `peinject`: バイナリを配信・実行
- `rundir`: ファイル一覧を返す

[^elastic]

Huntressの John Hammond は報告した。「すべての感染ホストで、RATは即座にシステム偵察を実行した ―― ユーザーディレクトリ、ファイルシステム、実行中プロセスをすべて列挙してC2に送信した」[^thn]

月曜日の朝、あなたのマシンで何が開いていたか ―― すべて送られていたかもしれない。

考えるのをやめよう。

---

## 第五章：C2の現在 ―― 「死んでいる。誰が殺したかは不明」

記事執筆時点（2026年4月1日。エイプリルフールだが本当の話だ）：

```
$ proxy-web.exe "http://sfrclak.com:8000/6202033"
-> ERR_NAME_NOT_RESOLVED

$ proxy-web.exe "http://142.11.206.73:8000/6202033"
-> ERR_CONNECTION_REFUSED
```

死んでいる。

攻撃者の自主撤退か、ホスティング会社による停止か、当局の介入か。現時点で公開情報がない。

ThreatFoxにも登録はない。

謎は謎のまま残る。我々はこれに慣れることにした。慣れるしかない。

---

## 第六章：帰属 ―― 「やっぱりそうか」コーナー

さて。

Elastic Security Labsの分析によると、macOSバイナリはMandiantが追跡する **WAVESHAPER** という既知のC++バックドアと重大なオーバーラップを示す。WAVESHAPERは **UNC1069** ―― 北朝鮮（DPRK）関連の脅威クラスタ ―― に帰属されている。[^elastic]

翌4月1日、**Google Threat Intelligence Group（GTIG）** がElasticとは独立に同じ結論を発表。UNC1069への帰属と、macOS RAT（macWebT）がBlueNoroffインフラに接続していることを確認した。[^therecord][^bleeping]

2つの独立した大手セキュリティ機関が同じ結論に達した。

Right.

GTIGの主席アナリスト **John Hultquist** は述べた：

> 「北朝鮮のハッカーはサプライチェーン攻撃に深い経験を持っており、歴史的にそれを暗号資産の窃取に活用してきた」[^therecord]

**深い経験。歴史的実績。**

SentinelOneはすでに2023年、同グループが**偽Zoomキャンペーン**で暗号資産企業を標的にしていたことを把握していた。[^therecord] 偽Zoom、偽npmパッケージ。一貫している。方向性がぶれていない。

Mandiant CTOの **Charles Carmakal** はこう警告した。「今回の侵害で盗まれた認証情報が、今後数日・数週間・数か月にわたってさらなるサプライチェーン攻撃を可能にするだろう」と。[^therecord]

We do not envy anyone reading this on a Monday morning.

TechCrunchも「北朝鮮が関与」と報じた。[^techcrunch] もはや誰も驚いていない。

帰属は常に確率的だ。ただ現時点では「DPRK系、高確率」という認識が複数の独立した機関によって支持されている。

---

## 第七章：なぜ6分で見つかったのか ―― 「最後に人間が必要だった」という構造的問題

これだけ精巧な攻撃が、なぜ3時間以内に封じ込められたのか。

答えには希望と絶望が同居している。

Snykのタイムラインによると、**Socketのスキャナーが公開から約6分後（~00:27 UTC）に検出した**。[^snyk2]

**npmの公式機構ではなく、サードパーティの監視サービスによって。**[^aikido2]

6分。自動化の勝利だ。3つのシグナルが機能した：

**① GitHubに記録のないnpmリリース**
axiosは通常 GitHub Actions OIDC で公開される。今回は対応するコミットもタグもない。「GitHubの歴史と一致しないリリース」は自動スキャナーにとって赤信号だ。[^aikido]

**② インポートされていない依存のpostinstall**
使われていない依存関係が postinstall フックだけを目的として存在する。供給チェーン攻撃の教科書的パターン。[^aikido]

**③ 新規アカウント＋ProtonMail**
`nrwise` は新規で、`jasonsaayman` のメールも直前にProtonMailに変わっていた。[^stepsecurity]

### 検出から削除まで：1時間25分のギャップ

ElasticがGitHub Security Advisoryをファイルしたのが 01:50 UTC。npmが削除したのが ~03:15 UTC。[^elastic][^stepsecurity]

**6分で検出。1時間25分で削除。**

自動化は仕事をした。その後は人間が処理した。

「最終的な封じ込めは人間に依存する」という構造 ―― これは今後の課題として業界全体に残っている。我々はこれを指摘するたびに少しだけ疲れる。でも指摘し続ける。

---

## 第八章：npmの対応 ―― 「侵害されたアカウントの代わりに消しました」

削除を実施したのは **npmチーム**だ。`jasonsaayman` は侵害されていたので自分では動けなかった。[^stepsecurity]

npmの対応：[^stepsecurity]
1. 悪意あるaxiosバージョン（1.14.1・0.30.4）をunpublish
2. `plain-crypto-js` にセキュリティホールドを適用
3. `plain-crypto-js@0.0.1-security.0` スタブを公開して後続インストールを無害化

Elasticがaxiosリポジトリ宛にGitHub Security Advisoryをファイルしたのは 01:50 UTC（JST: 10:50）。[^elastic] 月曜日が終わる前に対応は完了した。

これは本当によかった。本当に。

---

## 第九章：広がる汚染 ―― 「axiosだけじゃなかった」という後日談

ところで。

Socketの調査で、**同一マルウェアを配布していた別パッケージが2件**発見された。[^thn]

| パッケージ名 | バージョン | 手口 |
|---|---|---|
| `@shadanai/openclaw` | 複数 | `plain-crypto-js` ペイロードを直接同梱 |
| `@qqbrowser/openclaw-qbot` | 0.0.130 | 改ざんした `axios@1.14.1` を `node_modules/` に同梱 |

axiosの正規依存関係は `follow-redirects`・`form-data`・`proxy-from-env` の3つだ。`plain-crypto-js` の追加はそれだけで明白な改ざんの証拠だ。[^thn]

これはaxiosだけを狙った作戦ではなかった可能性がある。全容はまだわかっていない。

Put that in your threat models.

---

## エピローグ：あなたがすべきこと

まず確認：

```bash
npm ls axios
ls node_modules/plain-crypto-js
```

痕跡の確認：

| OS | 確認場所 |
|---|---|
| macOS | `/Library/Caches/com.apple.act.mond` |
| Windows | `%PROGRAMDATA%\wt.exe`、`%PROGRAMDATA%\system.bat`、レジストリ Run キー |
| Linux | `/tmp/ld.py` |

[^elastic]

見つかった場合：

1. `node_modules/plain-crypto-js` を削除
2. `axios@1.14.0` または `0.30.3` にダウングレード[^stepsecurity]
3. **すべての認証情報をローテーション**（npmトークン、クラウドクレデンシャル、SSHキー、CI/CDシークレット、本当にすべて）[^aikido]
4. **Windows固有**: レジストリ Run キー `MicrosoftUpdate`、`%PROGRAMDATA%\wt.exe`、`%PROGRAMDATA%\system.bat` を削除[^thn]

`kill` コマンドでRATを止めてもWindowsの持続性は残る。念のため。

再発防止：

- `npm install --ignore-scripts`（postinstallを無効化）[^flatt]
- npmの `min-release-age`（7日以上推奨）[^flatt]
- pnpmの `trustPolicy: no-downgrade`[^flatt]

---

## まとめ ―― あるいは、我々が毎週月曜日に繰り返し学ぶこと

このインシデントの本質は **信頼の搾取** だ。

axiosは何年もかけて信頼を積み上げた。だから誰も疑わない。攻撃者はその信頼を正確に理解し、axiosのコードに悪意を1行も書かずに、世界中の開発環境にRATを送り込もうとした。

6分で検出された。1時間25分で封じ込められた。

それでも感染窓は実在した。週1億DLのエコシステムに向けて矢は放たれた。

**次は6分で検出されないかもしれない。**

`npm install` を実行するとき ―― 少しだけ立ち止まって考えてほしい。あなたが信頼しているのは、誰が、いつ、どのように公開したパッケージなのかを。

それだけでいい。

我々はこの記事を書き終えて、ため息をついた。

そしてまた来週も、月曜日が来る。

> [!NOTE]
> **2026年4月1日追記**: C2インフラは現時点で停止確認済み。エイプリルフールだが本当の話だ。ここだけは笑えない。

---

[^wiz]: [Axios NPM Distribution Compromised in Supply Chain Attack | Wiz Blog](https://www.wiz.io/blog/axios-npm-compromised-in-supply-chain-attack)
[^wiz_note]: Wiz調査による数値（根拠・測定方法の詳細は非開示）。
[^aikido]: [axios npm compromised: maintainer account hijacked, RAT deployed | Aikido](https://www.aikido.dev/blog/axios-npm-compromised-maintainer-hijacked-rat)
[^aikido2]: [axios npm compromised: maintainer account hijacked, RAT deployed | Aikido](https://www.aikido.dev/blog/axios-npm-compromised-maintainer-hijacked-rat) — "The attack's detection appears to have relied on vigilant third-party security monitoring rather than automated npm safeguards."
[^stepsecurity]: [axios Compromised on npm - Malicious Versions Drop Remote Access Trojan | StepSecurity](https://www.stepsecurity.io/blog/axios-compromised-on-npm-malicious-versions-drop-remote-access-trojan)
[^elastic]: [Inside the Axios supply chain compromise - one RAT to rule them all | Elastic Security Labs](https://www.elastic.co/security-labs/axios-one-rat-to-rule-them-all)
[^techcrunch]: [North Korean hackers blamed for hijacking popular Axios open source project | TechCrunch](https://techcrunch.com/2026/03/31/hacker-hijacks-axios-open-source-project-used-by-millions-to-push-malware/)
[^flatt]: [axios ソフトウェアサプライチェーン攻撃の概要と対応指針 | Flatt Security Blog](https://blog.flatt.tech/entry/axios_compromise)
[^snyk]: [Axios npm Package Compromised: Supply Chain Attack Delivers Cross-Platform RAT | Snyk](https://snyk.io/blog/axios-npm-package-compromised-supply-chain-attack-delivers-cross-platform/)
[^snyk2]: [Axios npm Package Compromised: Supply Chain Attack Delivers Cross-Platform RAT | Snyk](https://snyk.io/blog/axios-npm-package-compromised-supply-chain-attack-delivers-cross-platform/) — "Socket's scanner detects malicious version (within ~6 minutes)"
[^therecord]: [Google links axios supply chain attack to North Korean group | The Record](https://therecord.media/google-links-axios-supply-chain-attack-north-korea)
[^bleeping]: [Hackers compromise Axios npm package to drop cross-platform malware | BleepingComputer](https://www.bleepingcomputer.com/news/security/hackers-compromise-axios-npm-package-to-drop-cross-platform-malware/)
[^thn]: [Axios Supply Chain Attack Pushes Cross-Platform RAT via Compromised npm Account | The Hacker News](https://thehackernews.com/2026/03/axios-supply-chain-attack-pushes-cross.html)
