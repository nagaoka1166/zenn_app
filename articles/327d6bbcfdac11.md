---
title: "Web-Speed-Hackathonでパフォーマンスチューニングを学んだ"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["フロントエンド", "パフォーマンスチューニング", "TypeScript", "React"]
published: true
published_at: 2023-03-12 18:50
---

## はじめに
CyberAgentさんが主催するWeb-Speed-Hackathon-2023に参加してきました！
とても面白く学びがいがあり充実したイベントでした！
CAの社員さんが2ヶ月もかけて作り込んだ重々アプリに苦戦してましたがたくさん勉強できて充実したハッカソンになりました。
アプリがデプロイするまでコーヒー1杯飲めちゃうくらいの遅さでした
学んだことは業務に役立てることができると思うので面白いとこや所感、苦難などを殴り書きして残します。
今回書ききれない改善点もたくさんあります。(GraphQLや郵便番号入力など)
https://www.cyberagent.co.jp/careers/students/career_event/detail/id=28369

## パフォーマンスがいいと何がいいか
CAのエンジニアさんは以下のようないい影響があるとおしゃってました
  - ビジネスや事業面へのいい影響
  - kpiのような指標などにたいする影響
  - ユーザー体験の向上
    ・さまざまな媒体にも同様のパドーマンスを意識する
    ・低速回線ユーザーにも使いやすいwebを体験させる


ビジネス面に影響してくるのは聞いたことがある人が多いと思います。ECサイトが1秒レスポンスが遅れるだけで、売上の10%が減るなど利益に直結すると聞いたことがあります。

https://knowledge.cpi.ad.jp/.assets/redbox-ogawa-cdn.pdf



 この資料でも読み込み速度が1秒遅いと
    → 顧客満足度は16%下がる
    →  ページビューは11%減少
    → コンバージョン率は7%下がる
と言われています

それらのことからwebページの表示速度を上げることがめちゃくちゃ重要だとイメージできます。


## パフォーマンスの確認方法

### lighthouse
lighthouseを使用してwebページのパフォーマンスを評価することが可能とのことです。lighthouseはGoogle Chromeの拡張機能によって図ることができます。
https://developer.chrome.com/docs/lighthouse/performance/performance-scoring/


lighthouseが評価する五つの指標です。
#### 1. Performance
  以下の指標で測定されています
  - ページの最初のコンテンツが表示されるまでの時間。
  - ページの最大のコンテンツ要素が表示されるまでの時間。
  - ページが読み込まれるまでの時間。
  - ユーザーがページを操作できるようになるまでの時間。


#### 2. アクセシビリティ
  LighthouseにあるAccesibilityという評価項目は項目は検索エンジンのロボットやwebサイトにアクセスするユーザーが使用した時に最適かどうかを測ります。
  その評価対象はテキストや背景の色やHTMLの構造によって来るとのことです.

#### 3.  PWA
 Service Workerに基づいて評価、Webアプリマニフェスト、セキュリティ設定の有無で評価。

#### 4. SEO
webサイトのSEO対策

#### 5. BestPracices
Web開発のベストプラクティスに基づいて評価された指標。その中に以下が含まれています
- アセット最適化:
- 画像/フォント/キャッシュの最適化:


## webページの表示速度上げるために

### react-router-dom
今回のハッカソンで作られた`Anchorタグ`を押したときに全ページを読み込んでしまうために、React-Router-Domdeの`Linkタグ`で実装するとSPAの画面遷移ができる

### vite
 viteはbundleの設定がほ飛んどされているため、
 bundleを圧縮する

### アセット最適化
 #### svgoでsvgの最適化
   Base64が余分に重かったので消す。


![logo.svg](https://storage.googleapis.com/zenn-user-upload/d7286cedfe70-20230312.png)

 #### 遅延読み込み
 imgタグにloadingプロパティがあることを知りました。imgタグを読み込むタイミングを決めれるみたいです。
   ファーストビューに移らない要素のloadingプロパティ引数にlazyを与える。
   反対にファーストビューに移る要素は、遅延ロードしてしまうといけないためloading=lazyしてはならない。

#### フォーマットの最適化
  sharpやimagemagickと言ったライブラリで最適化ができることを知りました。
  どちらのライブラリも多くの画像処理を実現することができjpgやpngなどの重たい画像をwebpに圧縮してくれたりサイズを調整してくれたりしてくれます。

 重要でないCSSを先送りする
  初回のレンダリングに必要なcssはrel=preloadやonloadを活用してloadCSSを使用して非同期ファイルに読み込むことを学びました。
  ```
   <link rel="preload" href="stylesheet.css" as="style" onload="this.rel='stylesheet'">
  ```

#### レイアウトシフト
  widthRestrectionコンポーネントをなくし widthを制限するコンポーネントをmin-widthで対応する
  ```
import * as styles from './WidthRestriction.styles';

export const WidthRestriction: FC<Props> = ({ children }) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const [clientWidth, setClientWidth] = useState<number>(0);

  const isReady = clientWidth !== 0;

   
    <!-- 省略あり -->


  return (
    <div ref={containerRef} className={styles.container()}>
      <div className={styles.inner({ width: clientWidth })}>{isReady ? children : null}</div>
    </div>
  );
};
```

ブラウザによって何かが変わるためブラウザを意識し手パッケージを管理しにあと行けない

### Suspenceを使うコンポーネントを最低限にする
  Suspenceとは、二つの特徴がありローディング中にレンダリングできないようにして、レンダリング失敗した時のコンポーネントもハンドリングしてくれます。
  ハッカソンではトップレベルなコンポーネントで使われておりSuspenceをトップレベルに入れてしまうと不必要なものもロードされるまで時間がかかるとのことです。

### ReDos対策
 ReDosとは正規表現の脆弱性を利用した攻撃とのことです。正規表現の処理速度が遅いとサービスが停止したりする可能性があります。 ハッカソンではそこも出題されてました。
 ##### valid値を検証する際はないよりあるかで検索
メールアドレスの@や電話番号のハイフンがあるかで検索するのはないかよりあるか！で検索してほしいとのことです。以下の資料が参考になりました。
 https://yamory.io/blog/about-redos-attack/
 
 また、`String.prototype.includes()`や`eslint-plugin-redos`などを導入して検知させるといいとのことです。



## 感想
今回は東京に関西からwatnowのサークルメンバーで飛び込みました。
アベマタワーで作業できたのもあってとても捗り、また最後の発表でCA社員さんの知識の深さが見えまたり最終日には美味しいピザと寿司をご馳走していただいたりと充実した２日間でした！
そこでは社員さんや学生とラフに会話ができてよかったです！
これらの学びはこれからのWebにも絶対に活かすことができると思うのでアウトプットしていきたいです！！

参考文献
https://web.dev/i18n/ja/optimize-lcp/
