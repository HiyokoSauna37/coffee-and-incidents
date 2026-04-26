---
layout: post
title: "GitHub Issue #1 が C2 になる日 ― Tropic Trooper の AdaptixC2 + VS Code Tunnels 作戦（2026年4月）"
date: 2026-04-26 09:00:00 +0900
categories: [security, threat-intel]
tags: [apt, china, c2, github, vscode, tropic-trooper]
---

> [!NOTE]
> **この記事について**
> 本記事は、セキュリティリサーチブログ [watchTowr Labs](https://labs.watchtowr.com/) のライティングスタイル（ユーモアと皮肉を交えた独特の文体）に影響を受けて書いたパロディ記事です。
> 技術的事実はすべて公開ソースに基づいていますが、文体・演出・括弧内ツッコミなどは watchTowr Labs へのリスペクトを込めたオマージュです。
> 文末の脚注からソースに飛べます。

> [!IMPORTANT]
> **TL;DR（時間がない人向け）**
> - 中国系APT **Tropic Trooper**（KeyBoy / Earth Centaur / Pirate Panda / Bronze Hobart）が、台湾・韓国・**日本** に対し新キャンペーンを展開[^csn][^gbhackers]
> - 入口は **trojanized SumatraPDF**（OSSのPDFリーダ）を GitHub 経由で配布[^thn]
> - C2 は **GitHub Issue #1 への RC4暗号化 POST**。応答は Base64ファイルとして書き戻され、**10秒以内に削除**される（フォレンジック観点では犯罪現場が完全に掃除されている）[^csn]
> - 高価値ターゲットだけが **`code tunnel user login --provider github`** で **VS Code Tunnels** に昇格[^gbhackers]
> - 永続化スケジュールタスク名: **`\MicrosoftUDN`** **`\MSDNSvc`**（あなたが管理者なら絶対にスルーする命名）[^gbhackers]
> - **すべての通信先が「Microsoftまたは GitHub の正規サービス」** で完結している。これが今回の肝で、これが今回の絶望

---

椅子を引いて座ってほしい。

なんでもいいので飲み物を手元に用意してほしい。コーヒーでも、紅茶でも、何かもう少し強いものでも構わない。

これから話すのは、2026年4月、中国系APTがあなたの組織のSOCを **完全に無力化する** ためにどんな道具を選んだか、という話だ。

我々はこの記事を書きながら、何度かため息をついた。

**ため息は伝染する。** あなたも準備しておいてほしい。

---

## プロローグ：開発者のツールチェーンで完結する攻撃

少し想像してほしい。

ある朝、社内ネットワークの監視ダッシュボードに `github.com` への HTTPS POST が表示される。

あなたは何も思わない。

エンドポイントから `code.exe tunnel` が起動している。

あなたは何も思わない。

`api.ipify.org` ではないが `ipinfo.io` への問い合わせがある。

あなたはやはり、何も思わない。

開発者だらけのオフィスなら、これらは全部「正常」だからだ。

しかし2026年4月の Tropic Trooper キャンペーンは、その「正常」を全部繋ぎ合わせて C2 を組み立てた。

GitHub は「コードホスティング」ではなく「メッセージボード」になり、VS Code は「エディタ」ではなく「リバースシェルの代替」になり、SumatraPDF は「PDFビューア」ではなく「初期実行のための封筒」になった。

すべて、もともとそこにあった道具だ。

**neat.**

---

## 第1章：登場人物 ― Tropic Trooper という名の常連、あるいは「5つの名前を持つ男」

Tropic Trooper は新人ではない。

Trend Micro が 2011年頃から追跡している中国系 APT だ。**つまり、我々は彼らを15年間眺めてきた**。それでも止められていない。Sigh.

別名は時期とベンダーによって変わる。**KeyBoy**（Rapid7）、**Pirate Panda**（CrowdStrike）、**Earth Centaur**（Trend Micro 後期）、**Bronze Hobart**（SecureWorks 旧名）。

同一グループに 5 つの名前が付くというのは、APT界隈ではよくある話だが、それだけ複数チームが追っている証拠でもある。**5チームが追って、それでも15年間活動し続けている**、と言い換えると味わいが深まる。

過去の標的は、東南アジア（特に台湾・フィリピン・香港）の政府・軍・ヘルスケア・技術系。

地理的に「中国の周辺で、中国にとって関心の高い情報を持っている組織」が綺麗に並ぶ。**綺麗に並ぶ**、という事実そのものが、彼らがどれだけ作戦立案に予算を割いているかを示している。

今回の 2026年4月キャンペーンも、その地理ロジックは変わっていない。

**標的: 台湾の中国語話者、韓国、日本**[^gbhackers]。

ただし、技術スタックの方は **盛大にアップグレード** された。

---

## 第2章：入口 ― SumatraPDF という「無害な」封筒、あるいは「IT管理者を狙うとはこういうことだ」

最初の侵入ベクタは、**trojanized SumatraPDF** だった[^thn]。

SumatraPDF はオープンソースの軽量 PDF リーダで、Adobe Reader を嫌う人間にとっての定番だ。Tropic Trooper はそれを改竄したバイナリとして GitHub 経由で配布した。

ここで攻撃者の選択を整理しよう。

- **Adobe Reader を改竄しなかった**: 署名が要求されるし、検知も多い
- **Office マクロを使わなかった**: 2026年現在、警戒が強すぎる
- **メール添付の `.lnk` も使わなかった**: 主要 EDR がほぼフラグを立てる

代わりに「**OSSの軽量PDFリーダ**」を選んだ。

理由は推測になるが妥当だろう ―― これを使う層は **「Microsoft 公式じゃないものを進んで入れる開発者・パワーユーザ・IT管理者」** であり、その層は **権限が高いことが多く、社内に展開する力もある**。

つまり初期侵入時点で、Tropic Trooper は「フィッシングで一般職員を引っかける」のではなく、「**技術リテラシーが高く、しかし好奇心も高く、しかも社内に対する展開権限を持っている人**」を狙った。

これは作戦設計として、不快なくらい正しい。

我々はここでもう一回ため息をついた。

---

## 第3章：C2 ― GitHub Issue #1 という合法的な掲示板、あるいは「攻撃者がついにここまで来た日」

**ここが一番面白い**（一番絶望でもある）。

Tropic Trooper は **AdaptixC2** という OSS の C2 フレームワーク[^csn] をベースに、**カスタムビーコンリスナー** を独自開発した。

AdaptixC2 自体はパブリックに存在する Sliver / Mythic 系の後発フレームワーク。OSSなので、誰でも使える。誰でも改造できる。

攻撃者にとっての利点は明確だ ―― **アトリビューション（誰がやったか）が薄まる**。

Cobalt Strike のクラックビーコンを使えば即座に「あ、APT/IAB系ね」と判定されるが、AdaptixC2 ベースの独自ビルドは「誰の作品か」が即座にわからない。

**Right.**

そして、彼らがその上に組んだ C2 通信フローがこれだ[^csn]:

1. ビーコンが起動 → **`ipinfo.io`** で被害者の外部IP取得
2. 攻撃者の **GitHub リポジトリの Issue #1 に POST** で初期ビーコン送信
3. データは **RC4 暗号化、Base64 エンコード**
4. 攻撃者は別の Issue（タイトルパターン: `upload`, `fileupload` など）にコマンドを書き込む
5. ビーコンはそれをポーリング → 復号 → 実行
6. 結果は **Base64 ファイルとして Issue にアップロード**
7. **10秒以内に削除** ―― フォレンジック妨害

これは美しい。

**皮肉抜きで美しい。**

GitHub への HTTPS POST は、社内のあらゆる開発者端末から日常的に飛んでいる。`git push`、API 呼び出し、CI、Dependabot、Copilot ―― 区別がつかない。

Issue へのコメント書き込みは GitHub API の正規エンドポイントを叩くだけだから、**TLS の中身は当然見えない**。

10秒削除で **後追いの IR チームを永久に苦しめる** という設計判断も、過去にこの手法をやった攻撃者の経験則を踏襲しているように見える。GitHub の Issue 削除は、API audit log には残るが **被害組織からは見えない**。

つまり残るのは「**うちの端末から GitHub に POST が飛んだ**」という記録だけだ。

それだけでは何も判断できない。それは社内の開発者全員が毎分やっていることだからだ。

ipinfo.io も同じだ。クラウド系の社内ツールやインストーラが普通に叩く。アラートは出ない。

つまり ―― そして、**ここからが重要だ** ――

**「攻撃者が悪意を表現する瞬間」が、社内のテレメトリ上に存在しない。**

これが今回のキャンペーンの設計思想だ。

我々はこれを発見したとき、しばらく画面を黙って見つめた。

それ以外、できることがなかった。

---

## 第4章：昇格 ― VS Code Tunnels という「最高のリバースシェル」、あるいは「Microsoftが攻撃者に贈った最高の贈り物」

ビーコンで足場ができた後、Tropic Trooper は **全ターゲットを VS Code Tunnels に移すわけではない**[^gbhackers]。

「**high-value victims**（高価値ターゲット）」と判断された後、初めてオペレータは次のコマンドを打ち込む:

```bash
# VS Code CLI をダウンロードして配置
# その後、tunnel コマンドで GitHub アカウントを使ってログイン
code tunnel user login --provider github
```

そして VS Code Tunnels が開通する。

**...**

VS Code Tunnels は Microsoft が公式に提供している、**「VS Code のセッションを `*.devtunnels.ms` ドメイン経由でリモートから繋ぐ」機能**だ。

本来は「自宅のPCを会社から開く」「コラボレータと画面共有する」のような用途のために作られた。

**Right.**

それを攻撃者が悪用するとどうなるか。整理する:

- 通信先は **`*.devtunnels.ms`** ―― Microsoft の正規サービス、TLS、当然ブロックされない
- 認証は **GitHub アカウント**。攻撃者は使い捨ての GitHub アカウントを用意するだけ（無料、3秒）
- 通信プロトコルは VS Code 独自 ―― **既存のリバースシェル検知ルールがほぼ反応しない**
- **Microsoft 公式の証明書チェーン** で全部署名されている

これは Cobalt Strike Beacon や Sliver の HTTPS C2 と全く違う。

「攻撃者のインフラを叩いている」のではなく、「**Microsoft のインフラを経由して攻撃者と通信している**」のだ。

**Some would say this is disingenuous.**

防御側からすると、これに対抗するには「`code.exe tunnel` の起動を許可するか/しないか」という、ツールそのものへのポリシー判断が必要になる。

VS Code を使う組織で、これを禁止するのはなかなか難しい。

我々はそのことを Microsoft が知っていて、それでもこの機能を出した、ということについて深く考えるのをやめた。

---

## 第5章：永続化 ― 名前の芸術、あるいは「あなたのタスクスケジューラはもう信用できない」

Tropic Trooper は永続化に **スケジュールタスク** を使った。タスク名は次の通り[^gbhackers]:

- `\MicrosoftUDN`
- `\MSDNSvc`

「Microsoft」「MSDN」「Svc」 ―― **何も知らない管理者なら絶対にスルーする命名**。

Microsoft Update のデーモンか何かに見える。MSDN Subscription関連サービスかもしれないと思う。"Svc" って書いてあれば service だろうと思う。

これも昔ながらの戦術だが、未だに効く。

タスクスケジューラを開いて 100 個近い `\Microsoft\Windows\` 配下のタスクを眺めた経験がある人なら、`\MSDNSvc` がそこに紛れ込んでいたとき、即座に異常と判断できる自信があるだろうか。

我々はない。

**あなたは？**

---

## 第6章：なぜ日本が標的か、あるいは「ホームルーターはまだ EDR を持っていない」

Dark Reading と gbhackers の記事は、日本のホームルーターと日本の組織が標的に含まれていると報じている[^csn][^gbhackers]。

Tropic Trooper の従来の関心領域（中国周辺の地政学的に重要な情報）と、近年の日中・台日関係を考えれば、**日本が標的に入ること自体は驚きではない**。

むしろ「いつ来るか」だった。今年来た、というだけだ。

注目すべきは「**ホームルーター**」という標的選定だ。これは恐らく:

- リモートワーク端末への中継インフラとして
- VPN 経由で社内に入る前のフットプリント拡大
- IoT/小規模デバイスは EDR が入らないので滞留しやすい

という意図だろう。

日本の中小企業や個人事業主のホームルーターは、ファームウェア更新がほぼされない領域で、Tropic Trooper クラスの相手にとっては **美味しい滞留拠点** になる。

**Put that in your threat models.**

---

## 第7章：SOC が見るべきもの ― あるいは「観測可能なものが減っていく」現実

ここまで読んで「**で、何を見ればいいんだ**」と思っているなら、それは正しい反応だ。

CSN の記事[^csn] と公開情報をベースに、検知・防御の観点を整理する:

### ネットワーク層

- **`*.devtunnels.ms`** への通信を、**開発者以外のセグメントで観測したら即フラグ**
- **`ipinfo.io`** への問い合わせを、社内基幹システム / 一般職員端末からのものに限ってフラグ（開発者端末だと誤検知が多すぎて消耗する）
- GitHub API への POST トラフィックの **異常な頻度**（特に Issue endpoint へのポーリング）

### エンドポイント層

- **`code.exe tunnel` の実行** を Sysmon Event ID 1 で記録、開発者以外で出たらアラート
- **`\MicrosoftUDN`** や **`\MSDNSvc`** といった、`\Microsoft\Windows\` 配下に置かれていない "Microsoft っぽい名前" のタスク
- SumatraPDF の実行プロセスから **子プロセスとしてシェル / PowerShell が起動** するパターン

### 組織ポリシー層

- **VS Code Tunnels の利用ポリシー** を明示する（許可するなら誰が、どの条件で）
- GitHub 個人アカウントから組織端末へのアクセスを監査
- OSS インストーラ（特に PDF / 軽量ツール系）を **アプリケーション許可リスト** で管理

最後のは現実的に難しい組織が多いと思う。

だが、Tropic Trooper の今回の作戦は **「OSS を正規ルートで配布する」前提で組まれている** ので、ここを無視すると **入口が完全に開いたままになる**。

これを書いていて、我々は SOC アナリストの友人を何人か思い出した。彼らは月曜日が来るたびに、こういう新手のキャンペーンと向き合うことになる。

**We do not envy them.**

---

## エピローグ：信頼を悪用する時代、あるいは「なぜ我々は同じ記事を毎月書いているのか」

Tropic Trooper の今回のキャンペーンは、**技術的に新しい要素は実はそれほどない**。

- AdaptixC2 → 既存 OSS
- GitHub Issue を C2 に → 過去にも複数の APT がやった
- VS Code Tunnels 悪用 → 2024年頃から散発的に観測されていた
- スケジュールタスク偽装 → 古典中の古典

しかしこれらを **「すべて正規サービス・正規ツールの上で完結させる」** という統合のしかたが、ここまで完成度高くまとまっているのは新しい。

**Sigh.**

我々（防御側）は、「悪意ある通信先を block する」という発想から、「**正規サービスがどう悪用されているかを文脈で判断する**」という方向に、もう一段ギアを上げる必要がある。

これは EDR や NDR の問題というより、

- **「VS Code Tunnels を会社で許可するのか」**
- **「ipinfo.io を全員に許可するのか」**
- **「OSSビルドの実行をどう統制するのか」**

といった、**組織ポリシーレイヤの判断** が問われる話だ。

技術で解決しきれない部分が増えている。

それが、Tropic Trooper が 2026年4月に教えてくれたことだと思う。

---

そして、おそらく来月か再来月、また誰かが **GitHub Issue #2 を C2 にする方法** を発明するだろう。

そのときには、今度こそ `*.devtunnels.ms` のポリシーくらい、組織で決めておこう。

我々はこの記事を書き終えて、もう一度ため息をついた。

**月曜日はもうすぐだ。**

---

## 参考情報

- 攻撃グループ: Tropic Trooper（別名: KeyBoy, Earth Centaur, Pirate Panda, Bronze Hobart）
- 観測時期: 2026年4月下旬に複数メディアで報道
- 標的地域: 台湾、韓国、日本
- 主要TTP: AdaptixC2 + GitHub Issues C2、VS Code Tunnels悪用、trojanized SumatraPDF、スケジュールタスク永続化

[^thn]: The Hacker News, "Tropic Trooper Uses Trojanized SumatraPDF and GitHub to Deploy AdaptixC2", 2026-04-24, https://thehackernews.com/2026/04/tropic-trooper-uses-trojanized.html
[^csn]: CyberSecurityNews, "New Tropic Trooper Attack Uses Custom Beacon Listener and VS Code Tunnels for Remote Access", 2026-04-23, https://cybersecuritynews.com/new-tropic-trooper-attack-uses-custom-beacon-listener/
[^gbhackers]: gbhackers.com, "Tropic Trooper Uses Custom Beacon and VS Code Tunnels for Stealthy Remote Access", 2026-04-23, https://gbhackers.com/tropic-trooper-uses-custom-beacon/
