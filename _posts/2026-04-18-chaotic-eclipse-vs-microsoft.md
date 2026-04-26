---
layout: post
title: "セキュリティ研究者がMicrosoftにキレて Windows Defender のゼロデイを3つばらまいた話"
date: 2026-04-18 12:00:00 +0900
categories: [security, vulnerability]
tags: [windows, microsoft, defender, zero-day, disclosure]
---

> [!NOTE]
> **この記事について**
> 本記事は、セキュリティリサーチブログ [watchTowr Labs](https://labs.watchtowr.com/) のライティングスタイル（ユーモアと皮肉を交えた独特の文体）に影響を受けて書いたパロディ記事です。
> 技術的事実（CVE番号・攻撃チェーン・タイムライン・被害状況）はすべて Help Net Security、BleepingComputer、CloudSEK、Huntress Labs、TechCrunch等の公開情報に基づいていますが、文体・演出・括弧内ツッコミなどは watchTowr Labs へのリスペクトを込めたオマージュです。

> [!WARNING]
> **重要：この記事は研究者側の視点に偏っています**
>
> MSRCとのやり取りの詳細（脅迫・クレジット問題・報告却下など）は、すべて研究者「Chaotic Eclipse」が公開した一方的な主張に基づいています。Microsoftは研究者の具体的な主張に対して個別の反論・確認・否定を公式に行っておらず、「責任ある脆弱性開示を支持する」という一般的な声明を出しているのみです。
> Microsoft側に何があったかは現時点では不明であり、本記事の記述が事実の全体像とは限りません。我々はその点を **何度でも** 強調しておきます。

---

椅子を引いて座ってほしい。

これから話すのは、ある匿名の研究者が、Microsoftにキレた結果、**Windows Defender それ自体を SYSTEM 昇格用ツールに変える方法を3つ、世界に向けてばらまいた** という、2026年4月の話だ。

少し違う言い方をすると、こういうことだ。

**あなたの組織のエンドポイントで動いている Microsoft Defender は、現在、関係ない組織を侵害するために使われている。** Defender が悪用されている、ではない。**Defender が攻撃ツールとして機能している。**

我々はこの記事を書くにあたって、何度かため息をついた。それから、ある種の畏怖を感じた。

あなたも同じような気持ちになると思う。

---

## プロローグ：脆弱性を報告された側が、報告した側を「路頭に迷わせた」と主張される世界

2026年4月7日。

`Nightmare-Eclipse` という、作成からわずか11日のGitHubアカウントが、`BlueHammer` という名のリポジトリを公開した。

中身は **Microsoft Defender の特権昇格エクスプロイトの完全な PoC** だった。

[Huntress Labs](https://www.helpnetsecurity.com/2026/04/17/microsoft-defender-zero-days-exploited/) はその3日後、それを使った攻撃試行を野生で観測した。

**そして、それはまだ序章だった。**

研究者「Chaotic Eclipse」（同人物）はブログにこう書いた:

> "they told personally by them that they will ruin my life and they did"
>
> （彼らは私に直接、私の人生を台無しにすると言った。そして、実際にそうした）

> "left me homeless with nothing"
>
> （私を路頭に迷わせた）

「彼ら」とは、Microsoft Security Response Center ―― つまり MSRC のことだ。

これは、ある研究者が、Microsoftに復讐するために **世界中の組織のエンドポイント防御を武器に変えた** 話だ。

そして、**その結果として、何の関係もない組織が現に侵害されている**という話でもある。

Sigh.

---

## 第1章：登場人物 ―― あるいは「我々が今からあなたに紹介する人々全員、なんらかの形で困っている」

| 人物・組織 | 役割 |
|---|---|
| **Chaotic Eclipse / Nightmare Eclipse** | 匿名の脆弱性研究者。GitHub と[ブログ](https://deadeclipse666.blogspot.com)で活動。新しいGitHubアカウントを作るつもりはなかった、と書いている。それは事実なのか、彼/彼女のドラマの一部なのか、我々にはわからない |
| **Microsoft Security Response Center（MSRC）** | Microsoftへの脆弱性報告窓口。今回の件について、研究者の具体的な主張に対する個別回答はしていない |
| **Huntress Labs** | 実際の攻撃を最初に観測したセキュリティ企業。彼らの月曜日も大変だった |
| **Will Dormann（Tharros）** | RedSun の動作を独立検証したセキュリティアナリスト。「**完全にパッチが当たった2026年4月のシステムでも約100%の再現率で動作する**」と確認した。100%。round number だ |
| **Zen Dodd / Yuanpei Xu** | CVE-2026-33825（BlueHammer）のパッチでクレジットされた研究者。彼らは無関係に同じ脆弱性を報告した可能性があるが、**それを我々が知る術はない** |

---

## 第2章：MSRCとの対立 ―― 「人生を台無しにする」という言葉が真実かどうか、我々は知らない

ここで一度、強くお願いしておく。

**この章に書く全ては、研究者本人の発言だけが情報源だ。**

Microsoft が実際にこういうことを言ったかどうか、我々にも、TechCrunchにも、BleepingComputerにも確認する手段がない。

その上で、Chaotic Eclipse の主張を整理する:

- MSRC に Windows Defender の脆弱性を報告した
- MSRC側で「**誤って分類・却下された**」
- その後のやり取りで、MSRC の人物から **「人生を台無しにする」と直接言われた**
- そして、**実際にそうされた**
- 「**ありとあらゆる幼稚な手を使われた**」
- 結果として **路頭に迷った**

Right.

研究者は2026年3月27日、`Nightmare-Eclipse` という新しいGitHubアカウントを作った。

プロフィールは空。経歴情報なし。**意図的な匿名化**だ。

[ブログ](https://deadeclipse666.blogspot.com)の冒頭にはこうある:

> 「ブログを再開して新しいGitHubアカウントを作るつもりはなかった」

つまり、この人物は **以前から別の活動歴を持つ研究者** だ。本名と経歴を捨てて、新しいハンドルで戻ってきた、ということになる。

これが何を意味するか、考えるのをやめよう。

我々はこれを書きながら、どちら側にも明確に肩入れできない居心地の悪さを感じている。

---

## 第3章：タイムライン ―― 「クレジットから名前が外された日」から「世界に火がついた日」まで

| 日付 | 出来事 |
|---|---|
| 2026-03-27 | Nightmare-Eclipse GitHub アカウント作成。**実弾装填フェーズ** |
| 2026-04-07 | BlueHammer PoC を GitHub / X に公開。**最初の弾、発射** |
| 2026-04-10 | Huntress が BlueHammer を使った攻撃試行を観測（Defender がブロック） |
| 2026-04-14 | Microsoft が CVE-2026-33825（BlueHammer）をパッチ。**クレジットは Zen Dodd・Yuanpei Xu のみ** |
| 2026-04-16 | **RedSun・UnDefend を同時公開**。報復、開幕 |
| 2026-04-17 | Huntress がRedSun・UnDefend の **野生での悪用** を確認 |

ここで、4月14日のクレジット欄に注目してほしい。

`Zen Dodd, Yuanpei Xu`

Chaotic Eclipse の名前はない。

**4月14日にパッチが出た。クレジットから外された。4月16日にもう2つ公開した。**

「もう2つ」とは、**未パッチで、Defender 自体を SYSTEM 昇格に変える** PoC のことだ。

この時系列を眺めると、ある仮説が頭に浮かぶ。我々はそれを口にしないことにする。Microsoft 側に何があったか、我々は知らないからだ。

ただ、起きたことは観測可能だ。

そして、起きたことの結果は、**現実の組織が侵害されている**ということだ。

---

## 第4章：3つの脆弱性 ―― あるいは「Defender がDefender でなくなる3つの方法」

### BlueHammer（CVE-2026-33825）― ペイドオフされた1発目

Defender の LPE（ローカル権限昇格）脆弱性。CVSS 7.8。

2026年4月14日のパッチで修正済み[^socradar]。

つまり、**この1つだけは決着がついている**。Microsoft が修正した。あなたが Patch Tuesday を真面目に当てている組織なら、これはもう問題ではない。

でも、これは前菜だ。

### RedSun（CVE未割当・未パッチ）― 「Defenderに自分自身を撃たせる方法」

Defender の **修復処理** を悪用して **SYSTEM権限** を取得する脆弱性だ。

仕組みを書く。心の準備をしてほしい。

Defender は、クラウドタグの付いたファイルを検知すると、**SYSTEM権限で修復（書き直し）** を行う。RedSun はこの動作を以下の手順で悪用する:

```
① Cloud Files API を使って偽のクラウド同期プロバイダーを登録する
② クラウドタグ付きのプレースホルダーファイルを作成し、Defender のスキャンをトリガーする
③ OPLOCK（オポチュニスティックロック）でDefenderの実行を特定タイミングで一時停止させる
④ 作業ディレクトリをジャンクションポイントで C:\Windows\System32 に向け直す
⑤ Defender が SYSTEM権限で「元のパス」にファイルを書き込もうとする
⑥ カーネルが書き込みを System32 にリダイレクトする
⑦ TieringEngineService.exe を上書き → SYSTEM 権限での任意コード実行
```

7ステップだ。

7ステップで、**あなたの組織の防御製品が、攻撃者のSYSTEMシェルになる**。

Tharros の Will Dormann が独立検証を行った結果は、こうだ[^cloudsek]:

> **「完全にパッチが当たった2026年4月のシステムでも約100%の再現率で動作する」**

100%。

100%。

我々はこの数字を見たとき、しばらく何も言えなかった。

影響範囲: **Windows 10 / 11 / Server 2019以降（Defender 有効環境）**

つまり、**Defender が走っている、ほぼ全ての企業端末**だ。

Defender を切れば防げる。Defender を切れば。

We do not envy any CISO reading this.

### UnDefend（CVE未割当・未パッチ）― 「ところで、Defender 自体も止められます」

最後に、おまけのように、UnDefend が公開された。

これは **一般ユーザー権限で Defender のシグネチャ更新をブロック** できるツールだ。

Microsoft が大規模な Defender アップデートを配信したタイミングでは、**完全な無効化** も可能とされている。

RedSun と組み合わせて使われる。

順序を整理すると、こうなる:

1. **UnDefend で Defender のシグネチャ更新を止める**（古いシグネチャのまま固定）
2. **RedSun で Defender に自分自身を System32 に書き込ませて SYSTEM 取得**
3. **そして、その後は何でもできる**

Some would say this is disingenuous.

We would say this is the worst thing Defender has ever been used for.

---

## 第5章：Huntress 観測 ―― 「現実に侵害された組織が存在する」という決定的な事実

[Huntress は2026年4月17日時点で以下の攻撃パターンを観測している](https://www.helpnetsecurity.com/2026/04/17/microsoft-defender-zero-days-exploited/):

- 初期侵入：**SSLVPN の侵害済みユーザーアカウント** を経由
- Downloads / Pictures フォルダに `RedSun.exe` などを配置
- **UnDefend で Defender の更新を停止**
- **RedSun で SYSTEM 権限を取得**
- 資格情報のダンプ、Active Directory の列挙、横展開

少なくとも1つの組織への侵害が確認されている。

**少なくとも1つ。**

これが、研究者がブログに「人生を台無しにされた」と書いた数日後に起きていることだ。

そして、これは続く。

なぜなら、PoC は GitHub に公開されており、誰でも武器化できるからだ。

---

## 第6章：考察 ― 「誰が悪い」を綺麗に言い切れない種類の事件

この件は、単純に「研究者が悪い」とも「Microsoft が悪い」とも言い切れない。

それが、この事件が居心地悪い最大の理由だ。

### 研究者側に立つなら

MSRCへの報告が不当に扱われ、クレジットも得られなかったことへの怒りは理解できる、と書ける。Microsoft のような巨人を相手にした個人研究者の交渉力は、ほぼない。

### Microsoft 側に立つなら

研究者の主張は **片方の視点だけだ**。MSRC の内部で何があったか、我々は知らない。同じ脆弱性を別の研究者が独立報告していた可能性もある（Zen Dodd と Yuanpei Xu の名前がパッチ欄に載っているのは事実だ）。MSRC が「人生を台無しにする」とは絶対に言わなかった可能性も、もちろんある。

### 業界全体の構造に立つなら

ここからは、**この事件単体ではなく、業界全体の問題** だ。

- AIツールの普及により脆弱性の発見・報告件数が **爆発的に増加** している
- [HackerOne は2026年3月27日に Internet Bug Bounty プログラムの新規受付を停止した](https://www.darkreading.com/application-security/ai-led-remediation-crisis-prompts-hackerone-pause-bug-bounties)
- 理由は「**AIによる脆弱性発見の速度が上がる一方、修正側の対応能力がスケールしていない**」
- curl も同様の理由でバグバウンティプログラムを終了した
- トリアージ側が報告を「マニュアル通りに処理するだけ」の作業になりつつある、という[批判](https://borncity.com/win/2026/04/17/windows-defender-0-days-bluehammer-patched-and-redsun-unpatched/)がある

つまり、こういうことだ:

**MSRC が Chaotic Eclipse の報告を「誤って分類・却下」したのは、おそらく業界全体に蔓延する処理能力不足が背景にある。**

それは、Microsoft 個別の悪意ではなく、**世界中のセキュリティベンダーが等しく抱えている、構造的問題** だ。

そして、**その構造の中で、たまたま追い詰められた1人の研究者が、関係のない組織を巻き添えにしながら反撃に出た**。

これが今回の事件の輪郭だ。

我々はこれを書きながら、何度かため息をついた。

笑えない。

---

## 第7章：あなたがすべきこと ―― 「Patch Tuesday を待つ」では遅い

RedSun・UnDefend は、本記事執筆時点（**2026年4月18日**）で **未パッチ** だ。

| 緊急度 | アクション |
|---|---|
| **即時** | EDR で `RedSun.exe` `UnDefend.exe` という名前の実行ファイルをブロック（攻撃者は名前を変えるが、最初の波には効く） |
| **即時** | Defender の **シグネチャ更新が止まっている端末** を Hunting で洗い出す |
| **即時** | **SSLVPN ユーザー credential の rotation**（Huntress 観測の初期侵入経路）|
| **即時** | Cloud Files API の異常使用を検知（`CfRegisterSyncRoot` 系の API call ログ） |
| **数日以内** | TieringEngineService.exe の writeに対する file integrity monitoring |
| **次の Patch Tuesday** | Microsoft の緊急パッチ／out-of-band release を即適用 |

[TechCrunch](https://techcrunch.com/2026/04/17/hackers-are-abusing-unpatched-windows-security-flaws-to-hack-into-organizations/)によれば、Microsoftへの **緊急パッチ公開を求める声があがっており、次のPatch Tuesdayを待たずに対応される可能性** がある。

**可能性。**

確定ではない。

そして、可能性の議論をしている間に、Huntress が観測した「少なくとも1つの組織」のリストは、おそらく増えている。

---

## エピローグ ―― あるいは「我々がこの事件から学ぶこと」

この事件には、たくさんの教訓がある。

技術的なもの:

- **EDR / 防御製品は、攻撃面でもある**。あなたが信頼してインストールしたものは、それ自体が SYSTEM 権限で動いている特権ソフトだ
- **OPLOCK + ジャンクションの組み合わせ** は、Windows のファイルシステム周りで何度も悪用されてきた。これを使うソフトは、設計時点で **TOCTOU を疑え**
- **Cloud Files API は、Defender が SYSTEM で書き込む経路を提供する**。これを記憶しておこう

組織運営的なもの:

- **研究者を怒らせると、何が起きるかわからない**
- **クレジット欄は外交だ**
- **トリアージ能力が報告速度に追いつかなくなった世界では、研究者と Vendor の関係はもっと壊れる**

そして、研究者の倫理について:

- **どれだけ Microsoft を恨んでも、関係ない組織を巻き添えにすることは正当化されない**
- **しかし、それを「やめろ」と言える立場にあるのは誰か？** 我々はそれを言えるほど、Chaotic Eclipse が経験したことを知らない

我々は、この事件の倫理について明確な判決を下せる立場にはない。

ただ、観測可能な事実だけ書いておく:

- **Defender が SYSTEM 昇格ツールになる方法が、現在2つ、誰でもダウンロード可能な形で公開されている**
- **少なくとも1つの組織が、すでに侵害された**
- **Microsoft からの公式パッチは、本記事執筆時点では出ていない**

そして、**来週には Patch Tuesday が来る**。

それまで、あなたの組織のエンドポイントで動いている Defender は、依然として 7 ステップの手順で SYSTEM シェルに変換可能な状態だ。

我々はこの記事を書き終えて、もう一度ため息をついた。

そして、次に同じような事件を書くのは何ヶ月先か考えた。

おそらく、それほど遠くない。

---

*本記事は2026年4月18日時点の情報に基づいています。*
*Sources: BleepingComputer, Help Net Security, CloudSEK, Huntress Labs, CSO Online, BornCity, deadeclipse666.blogspot.com, TechCrunch*

[^socradar]: [SOCRadar - BlueHammer / RedSun / UnDefend Windows Defender 0-days](https://socradar.io/blog/bluehammer-redsun-undefend-windows-defender-0days/)
[^cloudsek]: [CloudSEK - RedSun Windows 0day: When Defender Becomes the Attacker](https://www.cloudsek.com/blog/redsun-windows-0day-when-defender-becomes-the-attacker)
