# Vue.js でアクセシブルなコンポーネントをつくるために

## はじめに

最初に自己紹介です。yamanoku と申します。<br>
マークアップエンジニアとして本格的に Web 技術に触れはじめ、途中からアクセシビリティ領域に関心をもち、現在は Web 標準の技術を活用しユーザー体験を向上させていきたいと考えております。

今回の Vue Fes Japan Online 2022 ではアクセシビリティと Vue.js にまつわるテーマで発表させていただきます。

なぜこのテーマで発表したかったかというと、Vue.js は他のライブラリ・フレームワークと比較してアクセシビリティの考慮が足りていないと感じたからです。

米国の非営利団体 WebAIM の調査によると Vue.js を用いたサイトではアクセシビリティ対応が足りていない報告もあります。<br>
これは Vue.js を使用するとアクセシブルでなくなるということではないのですが、気になる結果ではあります。

このセッションでは３つのテーマに沿って Vue.js を用いた開発でアクセシビリティに取り組みやすくなる内容を紹介いたします。<br>
対象は「Web アクセシビリティを学びはじめた人」や「Vue.js でアクセシビリティ向上をしていきたい人」向けの発表になります。<br>
アクセシビリティ対応についてを知らない人には考慮されていない内容となっておりますのでその点ご了承ください。

## 誤ったアクセシビリティ対応について

まずは「誤ったアクセシビリティ対応」について紹介します。

アクセシビリティ対応を意識するようになってから WAI-ARIA についてを知るようになる人は多いと思います。

WAI-ARIA は HTML だけではスクリーンリーダーといった支援技術に表現しきれないものを補完する技術仕様です。<br>
これを使うことによってアクセシビリティを高めることができると意気揚々と使い出す人もいると思います。私もその１人でした。

ですが WAI-ARIA を使用するに辺り気をつけなければならないことがあります。<br>
それは「NO ARIA is better than BAD ARIA」、つまり「ARIA 無しのほうが、悪い ARIA よりも良い」ということです。

今回はそんな WAI-ARIA の使用について誤りやすいものについてを紹介します。

### aria-label

こちらはテキストがない場合に支援技術に伝えたいテキストがある場合に使用されるものです。

具体例としてはアイコンだけのボタンが挙げられます。<br>
このパターンではボタンの内部にテキストが存在しないため「メニューボタン」として読まれます。

```html
<button type="button" aria-label="メニューボタン">
  <span class="fa-solid fa-bars"></span>
</button>
```

では次のパターンではどうでしょうか。

ボタンの内部に画像テキストがある場合ですが、これは `alt` の読み上げを上書きしてしまいます。

```html
<button type="submit" aria-label="送信">
  <img src="submit-text.png" alt="送信する">
</button>
```

また次のような `<label>` の `for` も無効化してしまいます。

`aria-label` は本来伝えたいテキストをオーバーラップしてしまうことがあります。<br>
支援技術と視覚上の情報とで齟齬が生まれないためにもできる限りテキストで提供できるようにしてください。

```html
<label for="text-field">名前</label>
<input type="text" aria-label="入力フォーム" id="text-field">
```

### aria-hidden

こちらは視覚的には伝わるが、支援技術には伝えない場合に使用されるものです。<br>
具体例としてはモーダルが表示されたときにそれ以外のコンテンツを隠すときなどが挙げられます。

```html
<main class="content" aria-hidden="true">
  コンテンツ自体は隠す
</main>
<div class="dialog-container" id="dialog" aria-labelledby="dialog-title" aria-describedby="dialog-description" aria-modal="true" role="dialog">
  <div>
    <h1 id="dialog-title">タイトル</h1>
    <p id="dialog-description">説明</p>
  </div>
</div>
```

この `aria-hidden` を使う場合は影響範囲をきちんと理解して使ってください。<br>
たとえばモーダルをラップする親要素に `aria-hidden` を使ってしまうとすべて隠されてしまうからです。

```html
<div aria-hidden="true">
  <main class="content">
    コンテンツは隠れる
  </main>
  <div class="dialog-container" id="dialog" aria-modal="true" role="dialog">
    <div>
      モーダル自体も隠れてしまう
    </div>
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

### aria-live

こちらは動的な変化があった場合にそれを支援技術に伝える場合に使用されます。これはライブリージョンと呼ばれるものです。<br>
具体例としては、SPA における画面遷移時や通知、カルーセルなどが挙げられます。

通知は緊急性の高いものでなければ都度通知させる必要はありません。<br>
ですが aria-live の値を誤ることで通知がうるさくなってしまいます。

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

`aria-live="assertive"` は動的な変更がある際に割り込んで通知してきます。<br>
これはアラートとしての役割であればよいのですが、そうでない通知においては過剰になってしまいます。

一般的な通知であれば、あらゆる動作が完了してから読み上げてくれる `aria-live="polite"` で設定しておくと良いでしょう。

---

このように、WAI-ARIA は要素によってはそのまま使用できないものがあったり、role や ARIA プロパティに対して無効な値、知らずに使うことでアプリケーションに影響を与えうるものがあるなど、使用方法を誤ることで簡単にアクセシビリティではないものを作り上げてしまいます。

一方で HTML においてはブラウザ標準において同一の挙動がおおよそ保証されています。<br>
WAI-ARIA をよくわからず使ってしまう前に、これらを使わずに済むように HTML だけで表現できないかということも検討してみてください。

---

## 汎用性のあるアクセシブルなコンポーネントを作る

先程の事例より WAI-ARIA を慎重に使わなければならないことが分かったと思います。<br>
ですがそれらをコンポーネントのルールとして内包させることで、チーム開発において認識の齟齬を減らした開発がしやすくなります。<br>
次は汎用性のあるアクセシブルなコンポーネント作りに関するヒントを紹介いたします。

### TypeScript で補強する

Vue3 からは TypeScript 対応が改善されてきました。<br>
もし導入している場合、アクセシブルなコンポーネントを補強するために有用となるかもしれません。

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

このパターンでは見出しコンポーネントを汎用的に使えるように実装しました。<br>
`<component>` では `is` 属性にてあらゆる要素が入り込む可能性があるため、それらを props の値から制御する形にしています。

これ以外にも画像のような場合は代替テキストとなる props を設置したり、多言語対応の際は `lang` 属性にあたるものを props に設置するなど、はじめてコンポーネントを使う人にも必要だと分かる props を定義しておくとよいでしょう。

```html
<!-- aria-current は空文字を受け入れないと怒ってくれる -->
<a :href="url" :aria-current="isCurrentPage ? 'page' : ''">リンク</a>
```

また `<template>` 内の WAI-ARIA の値もチェックしてくれます。不正な値が入らないように注視してみてください。

### Visually Hidden

Visually Hidden とは視覚的には確認できないが支援技術により確認できる方法です。

前述した `aria-label` とも似ていますが CSS によって内部にフォーカスできるものがある場合は表示させることができます。<br>
これはスキップリンクというキーボードユーザーのためのツールとして使用されます。

```vue
<script setup lang="ts">
import { computed, defineProps } from 'vue';

type Props = {
  isFocusable: boolean;
};

const props = defineProps<Props>();
const focusedClass = computed(() => props.isFocusable ? 'visually-hidden-focusable' : 'visually-hidden');
</script>

<template>
  <div :class="focusedClass">
    <slot />
  </div>
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

### Fragments

Vue3 からは Fragments という概念が導入されています。<br>
Vue2 まではルート要素は単一のものでなければなりませんでしたが、Fragments によりコンポーネントにおけるマークアップとしての正しさが保証されやすくなりました。

```html
<template>
  <!-- table-columns のようなコンポーネントを想定 -->
  <!-- vue2 までは div などでラップしておく必要があった -->
  <div>
    <tr><td>項目</td></tr>
    <tr><td>内容</td></tr>
  </div>
</template>
```

```html
<template>
  <!-- Vue3 からは以下のように書けるようになった -->
  <tr><td>項目</td></tr>
  <tr><td>内容</td></tr>
</template>
```

ただしコンポーネントのルート要素に親コンポーネントから付与される属性が自動的に付与されなくなることには注意です。<br>
その場合は受け取る要素に `:bind="$attr"` を付与しておきましょう。

## コンポーネントのアクセシビリティテスト・チェックをする

アクセシブルなコンポーネントを作れたでしょうか。<br>
次にそれが壊れていないかを維持するためにテストやチェックをしていきましょう。

### 静的解析

eslint を導入しているなら [eslint-plugin-vuejs-accessibility](https://github.com/vue-a11y/eslint-plugin-vuejs-accessibility) を使用することをオススメします。<br>
こちらは Vue.js でアクセシビリティを損ねるコードがあった場合に指摘してくれます。<br>
構築するサイトやアプリケーションによっては不要なルールがあるかもしれませんので、どういったルールを有効にするかを検討すると良いかもしれません。

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
  - role の使用において必要になるaria属性があるかチェック

HTML の確からしさを確認したい場合、[markuplint](https://github.com/markuplint/markuplint) を導入してみることも有用です。<br>
Vue.js のパーサーが存在するので併せて使うことで vue コンポーネント内をチェックしてくれます。

```html
<img src="image.png" :alt="null">
```

ただし markuplint は動的バインディングを評価できないため以下のような場合はスルーされてしまいます。<br>
その場合、前述した TypeScript や props で親からデータを受け取るように整形したり、Jest スナップショットテストを整形した結果をもとにした HTML を markuplint にかけてみるなどチェックすることも有用です。

### 結合テスト、ユニットテスト

テストのために Jest を導入していますでしょうか。だとするとあなたはラッキーです。<br>
結合テストの場合は　`@testing-library/vue` を使ってテストを書いてみましょう。

```js
import {render, screen, fireEvent} from '@testing-library/vue';
import Button from './Button.vue';

test('クリックしてカウントを追加する', async () => {
  render(Button);
  const button = screen.getByRole('button', {name: '追加'});
  await fireEvent.click(button);
  await fireEvent.click(button);
  expect(screen.getByRole('region', {name: /カウント: 2/i})).toBeTruthy();
});
```

`getByRole` を用いて要素の role を取得し、そこから内部の状態を確認するようにしています。
正しくマークアップや WAI-ARIA を使用できれいればそれらを活用してテストコードを取得できます。

さらに [jest-axe](https://github.com/nickcolley/jest-axe) を使用することで全体で違反がないかをチェックすることも出来ます。

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

### ブラウザ・支援技術のチェック

テストコードをもってアクセシブルな正しさを確認できたら、次は実際にどう使われているかをチェックしてみましょう。

[vue-axe](https://github.com/vue-a11y/vue-axe-next) は axe というアクセシビリティ検証ライブラリのエンジンを用いたアクセシビリティチェックツールです。<br>
開発モードにおいて実際のブラウザ上で違反している箇所がないかを指摘してくれます。

スクリーンリーダーといった支援技術において、実際にどう使われるかもチェックしておきましょう。<br>
Windows と macOS には標準でスクリーンリーダーが搭載されています。

また一部の支援技術のみですが、自動テストとして実施できるためのツールも公開されています。

- [Guidepup](https://github.com/guidepup/guidepup)
- [web-test-runner-voiceover](https://github.com/blueprintui/web-test-runner-voiceover)
- [VoiceOver.js](https://github.com/AccessLint/screenreaders)

### ユーザーテスト・アクセシビリティ試験

支援技術でチェックできたとして、実際にそれらを使ってみている人にとって使いやすいかはわかりません。<br>
その場合は地域の障害者団体やパソコンボランティア団体などにユーザーテストの依頼をしてみるとよいかもしれません。

より厳格に診断をしてもらいたい場合、株式会社サニーバンクさんでは障害当事者によるウェブアクセシビリティ診断をされていますので活用されてみてはいかがでしょうか。

## おわりに

今回紹介したこと以外にもアクセシビリティに関しては考慮することはいくつもあります。ですがアクセシビリティ対応に完璧なものはないように小さく確実に薦めていくことが大切です。

今回の発表からアクセシビリティを意識した Vue.js でのフロントエンド開発が増え、アクセシブルなサイトやアプリケーションが生まれていければと思っております。

私の発表は以上になります。ご清聴いただきありがとうございました。