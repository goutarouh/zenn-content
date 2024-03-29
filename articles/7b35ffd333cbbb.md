---
title: "単体テストの考え方/使い方 読みました"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["test", "テスト", "単体テスト"]
published: true
---

# はじめに

[単体テストの考え方/使い方](https://www.amazon.co.jp/単体テストの考え方-使い方-Vladimir-Khorikov/dp/4839981728)を読みましたので、私が重要だと思った内容を私の意見と共にまとめました。
記事の対象は

- 単体テストの考え方/使い方を読んでみようと思っているけど、どんな内容なんだろう？と考えている方
- 単体テストを手っ取り早くいい感じに書きたい方

などです。

また、この記事は書籍のほんの一部です。他にも内容は盛りだくさんなので、興味のある方は実際に書籍を読むことをおすすめします。

# 古典学派?ロンドン学派？

書籍ではまず単体テストとは何たるかを定義しています。様々なテストがある中で単体テストとは以下の 3 つの条件を満たします。

1. 単体と呼ばれる少量のコードを検証する
2. 実行時間が短い
3. 隔離された状態で実行される

これらの条件は曖昧であることから、捉え方によって思想が分かれます。書籍では二つの思想を古典学派とロンドン学派と呼んでいます。

例えば 1 に関しては
古典学派は「サービスとしての一つの振る舞い」と捉えます。
ロンドン学派は「一つのクラス」と捉えます。

つまり古典学派では単体テストが一つのクラスに縛られることはないです。

これによって 3 に関しても
ロンドン学派では「テスト対称システム」と「協力者オブジェクト」を隔離します。
一方で古典学派は、むやみやたらにテスト・ダブルは使用しないことが多いです。

:::message
私自身もぼんやりと古典学派でしたが、読了後は明確に古典学派になっとと思います。単体テストはそれ自体のメンテナンスコストも考慮する必要があるため、実装の詳細はテストするべきではありません。実装の詳細をテストすると、テストが壊れやすくなるため、リファクタリング耐性が非常に低くなります。単体テストは「今正確に動くこと」だけを検証しているわけではないので、ここを考慮すると、私も古典学派を推したいと思います。
それと単純にテスト・ダブルばかりの、準備フェーズが肥大化した単体テストは非常に読みずらく、何を検証しているか理解することが容易ではありません。
「private メソッドは実装の詳細だからテストしなくていいよ」という意見に対して、何で？と思っていた新卒一年目の時に知りたかったです。
:::

# 単体テストの書き方を統一化する

単体テストを書くとき、AAA パターンなどを使用して全てのテストケースが統一されることを意識しましょう。AAA パターンは準備(arrange)、実行(act)、検証(assert)の頭文字です。こうすることで以下のメリットを獲得することができます。

- 可読性が劇的に向上し、保守コストが下がる
- 同じフェーズが複数存在しないので、複数の振る舞いを検証するようなことを避ける
- ビジネスロジックのテスト(=act フェーズ)を 1 行にすることで、ビジネスロジックのカプセル化を強制できる。

また個人的には以下を意識するとより可読性が上がります。

- システム対称クラスには統一した名前、例えば `sut` などと命名する
- 各フェーズ間に空行やコメントを記載する

以上を意識すると下記スニペットのようなテストを書くことができます。この書き方が必ずしも良いかはケースによりますが、統一化することの恩恵は大きいです。

```kotlin
@Test
fun XXX_TEXT() {
    // Arrange
    val fake = createFake()
    val sut = TestTarget(fake)

    // act
    val actual = sut.do()

    // assert
    Assert.equal(EXPECT, actual)
}
```

:::message
書き方を統一化すると受けられる恩恵は、思っている以上に大きいように思います。一見、可読性の向上にのみ繋がるように思えますが、実際に試した限りでは、複雑な依存を避けるような設計を考えるなど、プロダクションコードにも良い影響が現れました。
:::

# 単体テストの価値を言語化する

「価値のある単体テストが何か」を言語化することができますか？何が単体テストの価値を最大化するか考えてみましょう。
著者は以下を「良い単体テストを構成する 4 本の柱」と述べています。

- 退行に対する保護
- リファクタリングへの保護
- 迅速なフィードバック
- 保守のしやすさ

**退行に対する保護**
退行やバグをいかに発見できるかを示す性質。テストによって実行されるプロダクションコードが多いほど、この性質を備える。

**リファクタリングへの耐性**
テストが偽陽性を生み出さずに、リファクタリングを行うことができる性質。
偽陽性とは、実際には意図通りの振る舞いをしているのに、テストが失敗することを指す。テストが、テスト対象のコードの詳細と深く紐づくと、リファクタリングへの耐性がなくなる。

**迅速なフィードバック**
テストの実行時間が短いこと、開発サイクルで頻繁に行うことができること

**保守のしやすさ**
可読性、依存把握の容易さ、実行の容易さなど

:::message
一般的に単体テストは、「退行に対する保護」をベースに考えられてしまっていると私は思います。それ自体は悪くないですが、「開発したそのときに正しく動くこと」のみを保障したテストが散見されるようになります。単体テストの究極の目的は持続可能な開発を行うことです。つまり退行に対する保護と同じくらい、**リファクタリングへの耐性は重要**だと思います。これを意識すると、単体テストの価値が向上すると思います。
:::

# まとめ

書籍は非常に分厚く文字も多いので、記事に記載していない内容もたくさんあります。
私も全て読んだわけではありません。
本記事では、書籍をある程度読んで実際に単体テストに適用してみた過程を通じて、手っ取り早く単体テストの価値を上げることができそうな、私が重要だと思う内容をまとめました。

私は本書籍を通じて、単体テストの重要性を学びました。一方でむやみやたらに単体テストを導入すれば必ずしも良いわけではないことはお分かりだと思います。この記事でも紹介したように、単体テストはそれ自体がコードです。
誰かがメンテナンスする必要があります。偽陽性が発生すれば、誰かがそれに対応する必要があります。これはその分だけ開発が遅れるリスクに繋がります。そして何より、テストの開発者がテストの重要性を低く感じることになるかもしれません。何故なら、正しい振る舞いをしているコードを書いても、テストが通らないからです。
つまり価値の低い単体テストなら、書かない方がマシなのかもしれません。

この書籍から学んだ知識が、自分とみなさんの今後の単体テストに活かせることができればと思います。
