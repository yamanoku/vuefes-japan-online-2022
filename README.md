# Vue.js でアクセシブルなコンポーネントをつくるために

## はじめに

最初に自己紹介です。yamanoku と申します。<br>
マークアップエンジニアとして本格的に Web 技術に触れはじめ、アクセシビリティ領域に徐々に関心をもち、現在は Web 標準の技術を活用しユーザー体験を向上させていきたいと考えております。

今回の Vue Fes Japan Online 2022 ではアクセシビリティと Vue.js にまつわるテーマで発表させていただきます。

なぜこのテーマで発表するのかというと、Vue.js は他のライブラリやフレームワークと比較してアクセシビリティを考慮する機会が少ないのではないかと思ったからです。

アクセシビリティについて簡単に説明すると、だれもが情報にアクセスできるようにする考え方で、Web においてそれを実践するのが Web アクセシビリティと言われております。<br>
昨今のフロントエンド開発において、良い意味での差別化ができる領域になってきています。

Vue3 のドキュメントからアクセシビリティに関するドキュメントが追加されました。<br>
アクセシビリティの基礎や、実際のユースケースの一部を学べるコンテンツになっています。<br>
Vue2 以前では草案という形でマージされることなく終わってしまったので、こうしたドキュメントが追加されたことは良いことです。

ですが、米国の非営利団体 WebAIM の調査によると Vue.js を用いたサイトではアクセシビリティ対応が足りていない報告があります。<br>
これは特定のライブラリやフレームワークを使用するとアクセシブルでなくなるわけではないのですが、去年からの引き続き悪い結果のため気になっております。<br>
調査されたサイトに日本のものがいくつ含まれているかは定かではありませんが、この状況が野放しになっていいとは思っていません。

このセッションでは３つのテーマに沿って Vue.js を用いた開発でアクセシビリティに取り組みやすくなる内容を紹介いたします。<br>
対象は「Web アクセシビリティを学びはじめた人」や「Vue.js でアクセシビリティの向上をしていきたい人」向けの発表になります。

今回の事例で紹介するコードは Vue3 をベースにしたものとなっております。その点ご了承ください。

## 誤ったアクセシビリティ対応について

まずは「誤ったアクセシビリティ対応」について紹介します。

アクセシビリティを意識するようになってから WAI-ARIA についてを知った人は多いでしょう。

WAI-ARIA はスクリーンリーダーといった支援技術に HTML だけでは表現しきれないものを補完する技術仕様です。

ブラウザから DOM ツリー・CSS のレンダリングを解析してアクセシビリティに関する情報をもつアクセシビリティツリーに反映されます。<br>
そして OS ごとのアクセシビリティ API を通じて支援技術をもってユーザーに情報が届きます。<br>
WAI-ARIA はアクセシビリティツリーで認知できるように意味づけしてあげるという形です。

「これを使うことによってアクセシビリティを高めることができる」と使い出す人がいます。私もその１人でした。

ですが WAI-ARIA を使用するに辺り気をつけなければならないことがあります。<br>
それは「*NO ARIA is better than BAD ARIA（ARIA 無しのほうが、悪い ARIA よりも良い）*」ということです。<br>
つまり、WAI-ARIA を誤って使うなら、それを使わないほうが良いということです。

今回はそんな WAI-ARIA の使用について、誤りやすいものをいくつか紹介します。

### aria-label

こちらはテキストがない場合、支援技術に伝えたい情報がある場合に使用されるものです。

具体例としてはアイコンだけのボタンが挙げられます。<br>
このパターンではボタンの内部にテキストが存在しないため「メニューボタン」として認識されます。

```html
<button type="button" aria-label="メニューボタン">
  <span class="fa-solid fa-bars"></span>
</button>
```

では次のパターンではどうでしょうか。<br>
リンクの内部に画像テキストがある場合ですが、これは `alt` の情報を上書きしてしまいます。

```html
<a href="https://v3.ja.vuejs.org/" aria-label="リンク">
  <img src="link-text.png" alt="Vue.js 日本語版サイト">
</a>
```

また、次のような `<label>` の `for` 属性も無効化してしまいます。

```html
<label for="text-field">名前</label>
<input type="text" aria-label="入力フォーム" id="text-field">
```

このように `aria-label` は本来伝えたいテキスト情報を上書きしてしまうことがあります。<br>
支援技術と視覚上の情報とで齟齬が生まれないためにも、できる限りテキストで提供できるようにしてください。

ちなみに `aria-label` は WAI-ARIA 1.2 より `div` 要素や `span` 要素といった `generic` ロールほか、名前をつけてはならないロールの情報が追加されています。<br>
詳しくは[仕様](https://www.w3.org/TR/wai-aria-1.2/#namefromprohibited)をチェックしてみてください。

### aria-hidden

`aria-hidden` はその要素とすべての子要素をアクセシビリティツリーから削除するものです。
具体例としては装飾的な要素や重複するコンテンツ、モーダルが表示されたときにそれ以外のコンテンツを隠すときなどが挙げられます。

```html
<main class="content" aria-hidden="true">
  コンテンツ自体は隠す
</main>
<div
  id="dialog"
  class="dialog-container"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
  aria-modal="true"
  role="dialog">
  <div>
    <h1 id="dialog-title">タイトル</h1>
    <p id="dialog-description">説明</p>
  </div>
</div>
```

`aria-hidden` を使う場合は影響範囲をきちんと理解して使いましょう。<br>
たとえばモーダルをラップする親要素に `aria-hidden` を使ってしまうとすべて隠されてしまうからです。<br>
せっかくの支援技術サポートが無駄になってしまいます。

```html
<div aria-hidden="true"><!-- 親が aria-hidden で隠れている -->
  <main class="content">
    コンテンツは隠れる
  </main>
  <div
    id="dialog"
    class="dialog-container"
    aria-modal="true"
    role="dialog">
    モーダル自体も隠れてしまう
  </div>
</div>
```

またフォーカスさせる要素やそれを内包する要素に使用しないように気をつけてください。<br>
フォーカス自体はできてもその要素を認識できなくなるからです。

```html
<a href="https://vuefes.jp/2022/" aria-hidden="true">
  Vue Fes Japan Online 2022
</a>
```

一時的に隠すのではなく完全に要素を消したい場合、`display: none` や `hidden` 属性を使うことも検討してみてください。

### aria-live

こちらは動的な変化があった場合、それを支援技術へ伝える場合に使用されます。これはライブリージョンと呼ばれるものです。<br>
具体例としては、SPA における画面遷移時や通知、フォームの警告部分やカルーセルの変化時などが挙げられます。

通知は緊急性の高いものでなければ都度通知させる必要はありません。<br>
ですが `aria-live` の値を誤ってしまうと、通知がうるさくなってしまいます。

```vue
<script setup lang="ts">
import { ref } from 'vue'
const count = ref<number>(0)
</script>

<template>
  <label for="field-countup">カウントアップ</label>
  <input id="field-countup" v-model="count" type="number">
  <div aria-live="assertive">
    {{ count }}
  </div>
</template>
```

`aria-live="assertive"` は動的な変更がある際に他の操作を割り込んで通知してきます。<br>
これはアラートとしての役割であればよいのですが、そうでなければ過剰な通知となります。

ただの通知であれば、あらゆる動作が完了してから読み上げてくれる `aria-live="polite"` で設定しておくと良いでしょう。

ライブリージョンの実装について参考になるものとして [Vue Announcer](https://github.com/vue-a11y/vue-announcer/tree/next) というライブラリがあります。こちらはデフォルトで `aria-live="polite"` の設定になっておりますので実装が不安な場合は使用してみると良いでしょう。


### aria-current

こちらは現在のページの状態を伝える場合に使用されます。<br>
使用例としてはページネーションやナビゲーション、パンくずのリンクにおいて現在の項目を伝えるときが挙げられます。

単体の WAI-ARIA としてはこれまで説明したものと比較するとあまり見かけたことはないかもしれません。<br>
ですが、Vue.js にて開発する人たちであればよく見かけるライブラリで使用されています。<br>
それは [Vue router](https://router.vuejs.org/) です。

単体で使用されることもあれば、Nuxt.js での標準リンクコンポーネントでの `nuxt-link` でも機能として使用されています。<br>
現在の URL パスと合致するリンクがある場合、そのリンクには `aria-current="page"` が付与されます。

```Vue
<!-- /about にアクセスすると aria-current="page" が付与された a 要素で描画される -->
<nuxt-link to="/about">事例紹介</nuxt-link>
```

リンクテキスト自体がそのページタイトルと一致していれば有用な使い方と言えます。

ですが、この要素がコンテナ内に無作為に置いてあると情報が煩雑になってしまいます。<br>
マークアップする際にはどういったグルーピングするかを意識しましょう。

ちなみに Vue router には `aria-current` 部分を API として制御できます。<br>
値は複数存在しており、ステップや日付・現在時刻の情報を示すことが可能です。<br>
`aria-current` の値が意図したものとなっているか確認してみましょう。

---

このように WAI-ARIA は要素によってはそのまま使用できないものがあったり、知らずに使うことでアプリケーションに影響を与えうるものがあるなど、使用方法を誤ることで簡単にアクセシビリティではないものを作り上げてしまいます。

一方で HTML においてはブラウザ標準において同一の挙動がおおよそ保証されています。<br>
WAI-ARIA をよくわからず使ってしまいそうな時は、一旦立ち止まって使わずに済むように HTML だけで表現できないかということも検討してみてください。

HTMLをより詳細に学ぶにあたり、HTML解体新書を読むことをおすすめします。
現時点でもっとも HTML の仕様書に密接な書籍ですので、実装やレビュー時のリファレンスとしても活用してみてください。

---

## 汎用性のあるアクセシブルなコンポーネントを作る

先程の事例より WAI-ARIA は慎重に使う必要があることを説明いたしました。<br>
ですがそれらをいちいち確認しながらの開発は大変です。

そこでコンポーネント設計でルールを内包させることで、チーム開発において認識の齟齬を減らした開発がしやすくなります。<br>
次は汎用性のあるアクセシブルなコンポーネント作りに関するヒントを紹介いたします。

### TypeScript で補強する

Vue3 からは TypeScript 対応が改善されてきました。これを期に導入しだした人もいるでしょう。<br>
もし導入している場合、アクセシブルなコンポーネントを補強するために有用な手段となります。

```vue
<script setup lang="ts">
import { computed, defineProps } from 'vue';

type Props = {
  headlineLevel: 1 | 2 | 3 | 4 | 5 | 6;
  headlineText: string;
}
const props = defineProps<Props>();

const headlineTag = computed(() => `h${props.headlineLevel}`);
const headingText = computed(() => props.headlineText);
</script>

<template>
  <component :is="headlineTag">
    {{ headingText }}
  </component>
</template>
```

このパターンでは見出しコンポーネントを汎用的に使えるよう実装しました。

`<component>` では `is` 属性によってあらゆる要素を入れられるため、それらを props の値から制御する形にしています。

これ以外にも画像のような場合は代替テキストとなる `props` を設置したり、多言語対応の際は `lang` 属性にあたるものを `props` に設置するなど、はじめてコンポーネントを使う人にも必要だと分かる `props` を定義しておくとよいでしょう。

```html
<!-- aria-current は空文字が使用できないと怒ってくれる -->
<a :href="url" :aria-current="isCurrentPage ? 'page' : ''">リンク</a>
```

また vue-tsc といった型検査ツールを使用すると `<template>` 内の属性値もチェックしてくれます。<br>
不正な値が入らないように注視してみてください。

### Visually Hidden

Visually Hidden とは視覚的には確認できないが支援技術により確認できる方法です。

前述した `aria-label` とも似ていますが、以下のようなコンポーネントとすることで汎用的に使い回せます。

```vue
<script setup lang="ts">
import { computed, defineProps } from 'vue';

type Props = {
  isFocusable?: boolean;
  tag?: 'div' | 'span';
  innerText: string;
};

const props = defineProps<Props>();
const focusedClass = computed(() => props.isFocusable ? 'visually-hidden-focusable' : 'visually-hidden');
const tag = computed(() => props.tag ? props.tag : 'span');
</script>

<template>
  <component :is="tag" :class="focusedClass">
    {{ innerText }}
  </component>
</template>

<style scoped>
.visually-hidden,
.visually-hidden-focusable:not(:focus):not(:focus-within) {
  position: fixed;
  top: 0px;
  left: 0px;
  width: 4px;
  height: 4px;
  opacity: 0;
  overflow: hidden;
  border: none;
  margin: 0;
  padding: 0;
  display: block;
  visibility: visible;
}
</style>
```

また、CSS によってフォーカスさせる場合は表示させることができます。<br>
これはスキップリンクというコンテンツにスキップ移動できるキーボードユーザーのための手段として使用されます。

### Fragments

Vue3 からは Fragments という概念が導入されています。

Vue2 までは `<template>` 内のルート要素は単一のものでなければなりませんでした。
それによりコンポーネントによってはマークアップが間違っている場合がありました。

```vue
<template>
  <!-- table-columns のようなコンポーネントを想定 -->
  <!-- vue2 までは div などでラップしておく必要があったが以下のマークアップは不適応 -->
  <div>
    <tr><td>項目</td></tr>
    <tr><td>内容</td></tr>
  </div>
</template>
```

Fragments によりルートが単一でなくてもよくなり、コンポーネントにおけるマークアップの正しさが保証されやすくなりました。

```vue
<template>
  <!-- Vue3 からは以下のように書けるようになった -->
  <tr><td>項目</td></tr>
  <tr><td>内容</td></tr>
</template>
```

ただしコンポーネントのルート要素に親コンポーネントから付与される属性が自動的に付与されなくなることには注意です。<br>
その場合は `inheritAttrs` を `false` にして、属性を受け取る要素に `:bind="$attr"` を付与しておきましょう。

```vue
<SubmitButton disabled />
```

```vue
<template inheritAttrs="false">
  <!-- root にある div 要素自体に属性は継承されず、 -->
  <div>
    <!-- 指定した button 要素に属性が継承されるようになる -->
    <button type="submit" v-bind="$attrs">
      送信ボタン
    </button>
  </div>
</template>
```

## コンポーネントのアクセシビリティテスト・チェックをする

アクセシブルなコンポーネントが作れたでしょうか。素晴らしいですね。<br>
ですが、コンポーネントを作ること以上に大切なことがあります。それはきちんと動作しているかを確認することです。

というわけで、コンポーネント自体が本当に壊れていないかを確認するためにテストやチェックをしていきましょう。

### 静的解析

eslint を導入しているなら [eslint-plugin-vuejs-accessibility](https://github.com/vue-a11y/eslint-plugin-vuejs-accessibility) を使用することをオススメします。<br>
こちらは Vue.js でアクセシビリティを損ねるコードがあった場合に指摘してくれます。<br>
構築するサイトやアプリケーションによっては不要なルールが場合、どういったルールを有効にするかを検討しておくと良いでしょう。

記載しているものはルールの一例になります。

---

- `alt-text`
  - 代替の内容が必要な際に alt、aria-label などがあるかのチェック
- `aria-props`
  - 定義されていない aria 属性（タイポ含む）をチェック
- `aria-role`
  - 存在しないもの、abstract な role の使用をチェック
- `aria-unsupported-elements`
  - aria 属性を使用できないタグで使用していないかチェック
- `no-redundant-roles`
  - タグに重複した role を付与していないかチェック
- `role-has-required-aria-props`
  - role の使用において必要になる aria 属性があるかチェック

---

HTML の確からしさを確認したい場合、[markuplint](https://github.com/markuplint/markuplint) を導入してみることも有用です。<br>
Vue.js のパーサーが存在するので併せて使うことで vue コンポーネント内をチェックしてくれます。

ただし markuplint は動的バインディングを評価できないため以下のような場合はスルーされてしまいます。

```html
<img src="image.png" :alt="null">
```

この場合、前述した TypeScript や props で親から正しい内容を受け取るように型定義したり、静的な HTML の出力時や Jest スナップショットテストを整形した結果をもとにした HTML を markuplint にかけてチェックすることも有用です。

### 結合テスト

テストのために Jest を導入していますでしょうか。だとするとあなたはラッキーです。<br>
`@testing-library/vue` を使った振る舞いを意識した結合テストを書いてみましょう。

```js
import {render, screen, fireEvent} from '@testing-library/vue';
import Button from './Button.vue';

test('クリックしてカウントを追加する', async () => {
  render(Button);
  const button = screen.getByRole('button', {name: '追加'});
  await button.focus();
  await fireEvent.keyDown(button);
  await fireEvent.keyDown(button);
  expect(screen.getByRole('region', {name: /カウント: 1/i})).toBeTruthy();
});
```

掲載しているテストコードでは `getByRole` を用いて要素の `role` を取得し、内部の状態を確認するようにしています。

単なる ID 名やクラス名でも問題はないのですが、その場合は要素がセマンティクスでなくても取得できてしまいます。<br>
更に検証したい要素を容易に変更できてしまう可能性もあります。

正しくマークアップや WAI-ARIA を使用できれば、セマンティクスを尊重した上でテストコードが書けます。

さらに [jest-axe](https://github.com/nickcolley/jest-axe) を使用することで全体的に違反がないかをチェックすることも出来ます。<br>
これは [axe](https://www.deque.com/axe/) というアクセシビリティ検証ライブラリのエンジンを使用した Jest の matcher です。

```js
import { render } from '@testing-library/vue';
import { axe, toHaveNoViolations } from 'jest-axe';
import App from './App.vue';

expect.extend(toHaveNoViolations);

it('レンダリング結果がアクセシビリティ違反していない', async () => {
  const { container } = render(<App/>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

eslint 同様、ルールは独自で調整できるので必要に応じて精査してみてください。

### ブラウザ・支援技術のチェック

テストコードをもってアクセシビリティの正しさを確認できたら、次は実際にどう使われているかをチェックしてみましょう。

[vue-axe](https://github.com/vue-a11y/vue-axe-next) は axe を用いたブラウザ上でアクセシビリティチェックができるツールです。<br>
開発モードにおいて実際のブラウザ上で違反している箇所がないかを指摘してくれます。

キーボード操作ができるようになっているかも確認してみましょう。<br>
フォームやリンクなどフォーカス可能な領域がある場合はタブキーで意図通り遷移できているかチェックをしてみます。

Firefox ブラウザでは標準の機能でフォーカス順を確認できるようになっています。<br>
E2E テストを通してフォーカスを含めた一連の挙動をチェックするのもよいでしょう。

スクリーンリーダーといった支援技術において、実際にどう使われるかもチェックしておきましょう。<br>
Windows と macOS には標準でスクリーンリーダーが搭載されています。Windows のみですが OSS で開発されている NVDA というスクリーンリーダーもあります。

また一部の支援技術のみですが、自動テストとして実施できるためのツールも公開されています。

- [Guidepup](https://github.com/guidepup/guidepup)
- [web-test-runner-voiceover](https://github.com/blueprintui/web-test-runner-voiceover)
- [VoiceOver.js](https://github.com/AccessLint/screenreaders)

### ユーザーテスト・アクセシビリティ試験

支援技術でチェックできたとして、実際にそれらを使ってみている人にとって使いやすいかはわかりません。<br>
その場合は地域の障害者団体やパソコンボランティア団体などにユーザーテストの依頼をしてみるとよいでしょう。

より厳格に診断をしてもらいたい場合、株式会社サニーバンクさんでは[障害当事者によるウェブアクセシビリティ診断](https://sunnybank.jp/business/service/accessibility_diagnosis/)をされていますのでこちらも活用されてみてはいかがでしょうか。

## おわりに

今回紹介したこと以外でもアクセシビリティに考慮することはいくつもあります。フロントエンド開発だけ考慮すればよいものではありません。<br>
ですがアクセシビリティ対応に完璧なものはないので、小さく確実に進めていくことが大切です。

さらに Vue.js 開発とアクセシビリティの領域に足を踏み入れたくなりましたか？<br>
Marcus Herrmann 氏が [Accessible Vue](https://accessible-vue.com/) というコンテンツを無料で公開していますので、より探求していきたい人はこちらも併せてご覧になってみてください。

今回の発表からアクセシビリティを意識した Vue.js でのフロントエンド開発が増え、アクセシブルなサイトやアプリケーションが生まれていければと思っております。

私の発表は以上になります。ご清聴いただきありがとうございました。