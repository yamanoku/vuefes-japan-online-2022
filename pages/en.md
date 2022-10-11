---
layout: layout
title: To make accessible components in Vue.js
description: yamanoku's presentation at Vue Fes Japan Online 2022
lang: en
---

![Thumbnail: To make accessible components in Vue.js - 2022](../images/to-make-accessible-components-in-vuejs-2022.png)

## Translation Articles

[English page](https://yamanoku.net/vuefes-japan-online-2022/en/) / [日本語ページ](https://yamanoku.net/vuefes-japan-online-2022/ja/)

## Introduction

First, let me introduce myself.

<img src="../images/yamanoku-icon.png" alt="Photo: yamanoku" width="256" height="256">

My name is [yamanoku](https://twitter.com/yamanoku). I started working as a web designer and then began to get more seriously involved with web technologies as a markup engineer, developing a growing interest in the field of accessibility.

Currently, I work for a business company, and I would like to improve the user experience by utilizing web standards technology.

I will be giving a presentation on Vue.js and accessibility at Vue Fes Japan Online 2022, but first, let me explain why I am giving this presentation.

First, I would like to explain why I am presenting this topic. It is because I think that there are few opportunities to consider accessibility when developing with Vue.js.

To explain accessibility simply, it is the concept of making information accessible to everyone, and Web accessibility is the implementation of this concept on the Web.

In recent front-end development, being able to focus on this area has become an area where we can value and differentiate our sites and products from others.

[A section on accessibility](https://vuejs.org/guide/best-practices/accessibility.html) has been added from the Vue3 documentation. The content is designed to help you learn the basics of accessibility and some of the actual use cases.

It is good to see the addition of this type of documentation, [as it was never merged into](https://github.com/vuejs/v2.vuejs.org/pull/1002) Vue2 or before.

However, according to a survey by [WebAIM](https://webaim.org/), a U.S. nonprofit organization, sites using Vue.js lack accessibility support.

![The WebAIM Million 2022 results for JavaScript frameworks, showing that accessibility support in Vue.js is 24.8% lower than average.](../images/the-webaim-million-2022.png)

This does not mean that using a particular library or framework is not accessible, but it is a fact that support has been below average[^1] since the year before last.

I am not sure how many Japanese sites are included in the surveyed sites, but as a Vue.js developer, I do not think this situation should be left unchecked.

In this session, we will present three themes that will make it easier to address accessibility in future development with Vue.js.

The target audience is "people who are just starting to learn about web accessibility" and "people who want to improve accessibility with Vue.js".

The code introduced in this case study is based on Vue3. Please note that the code shown in this case study is based on Vue3.

## About Wrong Accessibility Support

In accessibility support, what you do for good without much understanding may have no effect, or on the contrary, it may make things worse.

Let's learn about the most common mistakes in order to eliminate bad accessibility supports.

![Screenshot: WAI-ARIA 1.1 Specification](../images/wai-aria-1-1.png)

Many of you may have heard about WAI-ARIA before you became aware of accessibility.

WAI-ARIA is a technical specification that supplements assistive technologies such as screen readers with accessible information that cannot be expressed in HTML alone.

![Flow from application UI to accessibility tree to assistive technology to user](../images/accessibility-tree.png)

Information about accessibility is reflected in the accessibility tree by parsing the DOM tree and CSS rendering from the browser.

Then, information is delivered to users with assistive technology from accessibility APIs for each OS.

WAI-ARIA is a form of semantics that can be recognized in this accessibility tree.

Some people start using WAI-ARIA as a way to increase accessibility. I was one of them.

However, there are some things to keep in mind when using WAI-ARIA.

That means "**NO ARIA is better than BAD ARIA**"[^2].

In other words, if you use WAI-ARIA wrongly, it is still better to not use it as an accessibility measure.

In this article, we will introduce some of the things that can easily go wrong when using WAI-ARIA, as well as some of the things that we tend to forget to be aware of when developing WAI-ARIA.

### aria-label

First is the aria-label, which is used when there is textual information to be conveyed to assistive technology.

An example of this is a button with only an icon.

In this pattern, there is no text within the button, so it is recognized as a "menu, button.

```html
<button type="button" aria-label="menu">
  <span class="fa-solid fa-bars"></span>
</button>
```

So what happens in this pattern?

```html
<a href="https://vuejs.org/" aria-label="Vue">
  <img src="link-text.png" alt="Vue.js official site">
</a>
```

If there is image text inside the link, this will override the `alt` information and read "Link, Image, Vue".

This also overrides the `label` element, which reads "Input form, edit text".

```html
<label for="text-field">Name</label>
<input type="text" aria-label="Input form" id="text-field">
```

Thus, aria-label may overwrite textual information that was originally intended to be conveyed.

In order to provide equivalent information to all people, rather than giving different information only to users of assistive technology, please provide the information in visible text as much as possible.

Incidentally, since WAI-ARIA 1.2, aria-label defines `roles` that must not be named, such as `generic roles` for `div` elements and `span` elements. Please check [the specification](https://www.w3.org/TR/wai-aria-1.2/#namefromprohibited) for details.

### aria-hidden

Next, let's take a look at aria-hidden.

aria-hidden removes the element and all its children from the accessibility tree.

Examples include decorative elements that do not need to be communicated to assistive technology, duplicate content, and hiding other content when a modal is displayed.

```html
<main class="content" aria-hidden="true">
  The content itself is hidden.
</main>
<div
  id="dialog"
  class="dialog-container"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
  aria-modal="true"
  role="dialog">
  <div>
    <h1 id="dialog-title">title</h1>
    <p id="dialog-description">description</p>
  </div>
</div>
```

aria-hidden is very powerful, so be sure you know the scope of its effect before you use it.

For example, if you use aria-hidden on a parent element that covers a modal, everything will be hidden, including the modal.

This is a loss of assistive technology support.

```html
<div aria-hidden="true"><!-- Parent is hidden by aria-hidden -->
  <main class="content">
    Content is hidden.
  </main>
  <div
    id="dialog"
    class="dialog-container"
    aria-modal="true"
    role="dialog">
    The modal itself is also hidden.
  </div>
</div>
```

Also, be careful not to use it on the element to be focused or the element that contains it.

Even if the focus itself is possible, you will not be able to recognize what the element is.

```html
<a href="https://vuefes.jp/2022/" aria-hidden="true">
  Vue Fes Japan Online 2022
</a>
```

Consider using the CSS `display: none` or `hidden` attributes if you want to completely remove the element rather than hide it temporarily.

### aria-live

Next, let's talk about aria-live.

This is a live region that is used to communicate dynamic changes within a page to assistive technologies.

Examples include changes in form input, screen transitions in SPA, warning notifications, and carousel changes.

```html
<script setup lang="ts">
import { ref, computed } from 'vue'
const count = ref<number>(0);
const personNumber = computed(() => {
  if (count.value < 0) return count.value = 0;
  return count.value;
});
</script>

<template>
  <button type="button" @click="count++">Increment</button>
  <button type="button" @click="count--">Decrement</button>
  <div role="status" aria-live="polite">
    ﹛{ personNumber }} persons
  </div>
</template>
```

For normal notifications, it is recommended to use `aria-live="polite"`, which reads out the operation after it is completed.

The `aria-live="assertive"` will interrupt other operations to notify you when there are dynamic changes.

This is fine as an alert, but should not be used with the wrong value, as constant notification of information during an automatically switching carousel or form entry would severely damage the user experience.

### aria-current

Finally, let's look at aria-current.

This is used to convey the current position.

Examples include pagination, navigation, and breadcrumb links.

As a stand-alone WAI-ARIA, it will not be used as often as the ones we have described so far.

However, it is used in libraries that are often seen by those who develop with Vue.js.

It is the [Vue Router](https://router.vuejs.org/).

Sometimes used in `<router-link>` and sometimes in `<nuxt-link>`, the standard linking component of Nuxt.js.

```html
<!-- When /about is accessed, it is drawn with the a element with aria-current="page" -->
<router-link to="/about">About page</router-link>
<!-- In the case of Nuxt.js -->
<nuxt-link to="/about">About page</nuxt-link>
```

If the link matches the current URL path, the link will have `aria-current="page"`.

This is useful if the link text itself matches the page title.

However, if this element is placed randomly, the information about what is the current location will be complicated.

```html
<div>
  <a href="/about" aria-current="page">About page</a>
  <!-- ... -->
  <!-- ... -->
  <!-- ... -->
  <a href="/about" aria-current="page">About page</a>
  <!-- ... -->
  <!-- ... -->
  <a href="/about" aria-current="page">About page</a>
</div>
```

When markup, be aware of the grouping of links and the areas where they will be placed.

By the way, Vue Router has an API to control the aria-current part[^3].

There are multiple values that can indicate step, date, and current time information. Let's check to see if the aria-current value is what you intended.

---

Thus, some elements of WAI-ARIA cannot be used as they are, and their unintentional use may affect the application.

The wrong use of WAI-ARIA can easily result in a non-accessible product.

On the other hand, using HTML roughly guarantees the same behavior in browser standards.

If you are likely to use WAI-ARIA without understanding it, please stop and consider whether it is possible to use HTML alone without using WAI-ARIA.

[MDN Web Docs](https://developer.mozilla.org/en-US/), a collection of web technology information for developers, lists accessibility concerns on the HTML elements page.

This is a useful site for understanding if you are using HTML incorrectly in your everyday writing.

If you want to learn HTML in more detail, we also recommend "[HTML Kaitai Shinsho](https://www.borndigital.co.jp/book/25999.html)".

It is the closest Japanese-language book to the HTML specification at this time, so you can use it as a reference for implementation and review.

---

## Make versatile, accessible components

The previous case study illustrates the need to use WAI-ARIA with caution.

However, it is difficult to develop while confirming each and every one of them.

Therefore, by including the rules in the component design, it will be easier for the team development to reduce the discrepancies in understanding.

Next, we will introduce some hints on how to create accessible components that can be used versatile.

### Reinforce with TypeScript

TypeScript support has been improved since Vue3. Some of you may have taken this opportunity to start using it.

If you use, this is a useful way to reinforce the creation of accessible components.

```html
<script setup lang="ts">
import { computed, defineProps } from 'vue';

type Props = {
  // Type annotate the level of the heading element with union type
  headlineLevel: 1 | 2 | 3 | 4 | 5 | 6;
  headlineText: string;
}
const props = defineProps<Props>();

// Use props heading level to represent like h1
const headlineTag = computed(() => `h${props.headlineLevel}`);
const headingText = computed(() => props.headlineText);
</script>

<template>
  <component :is="headlineTag">
    ﹛{ headingText }}
  </component>
</template>
```

This pattern implements a generic use of the heading component.

The `<component>` element can contain any element by the `is` attribute, which is controlled by the value of `props`.

In addition to this, it is recommended to define `props` that are necessary for new users of the component, such as `props` for alternative text in case of image components, `props` for `lang` attribute in case of multi-language support, and so on.

```html
<script setup lang="ts">
import { defineProps } from 'vue';

type Props = {
  altText: string;
  lang: 'ja' | 'en' | 'fr';
}

defineProps<Props>();
</script>
```

You can also use type checking tools such as [vue-tsc](https://github.com/johnsoncodehk/volar) to check attribute values in `<template>`.

```html
<!-- The type check tells me that aria-current cannot use empty characters -->
<a :href="url" :aria-current="isCurrentPage: ? 'page' : ''">
  Link
</a>
```

Watch for wrong values here.

### Visually Hidden

Visually Hidden is a method that cannot be seen visually but can be seen with assistive technology.

It is sometimes defined in UI frameworks such as [Bootstrap](https://getbootstrap.com/docs/5.0/helpers/visually-hidden/).

It is similar to the aria-label component mentioned above, but this type of component can be used universally.

```html
<script setup lang="ts">
import { computed, defineProps } from 'vue';

type Props = {
  isFocusable?: boolean;
  tag?: 'div' | 'span';
};

const props = defineProps<Props>();
const focusedClass = computed(() => props.isFocusable ? 'visually-hidden-focusable' : 'visually-hidden');
const tag = computed(() => props.tag ? props.tag : 'span');
</script>

<template>
  <component :is="tag" :class="focusedClass">
    <slot />
  </component>
</template>
```

It can be used to hide the Button component when there is no text, or the label portion of a form part.

It can also be made visible when the focus is controlled by CSS.

```html
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

This can be used as a "skip link", a means for assistive technology users to skip to content.

![Screenshot: Skip link in GitHub](../images/skip-link.png)

They do not usually appear on the screen, but allow users to skip directly to content when there is a lot of header content.

### Fragments

Vue3 introduces the concept of [Fragments](https://v3-migration.vuejs.org/new/fragments.html).

Until Vue2, the root element in a `<template>` had to be a single one. This led to incorrect markup for some components.

```html
<template>
  <!-- Assuming a component like table-columns -->
  <!-- in Vue2, you have to root divs, etc. -->
  <!-- but the following markup is not applicable -->
  <div>
    <tr><td>Item</td></tr>
    <tr><td>Contents</td></tr>
  </div>
</template>
```

Fragments allow for a single route and help ensure the correctness of markup in components.

```html
<template>
  <!-- Since Vue3, you can now write the following -->
  <tr><td>Item</td></tr>
  <tr><td>Contents</td></tr>
</template>
```

However, that the root element of the component will no longer automatically be assigned attributes from the parent component.

```html
<SubmitButton disabled />
```

```html
<template inheritAttrs="false">
  <!-- Attributes are not inherited by the div element itself at the root -->
  <div>
    <!-- By the button element you specify -->
    <button type="submit" v-bind="$attrs">
      Submit
    </button>
  </div>
</template>
```

In that case, set `inheritAttrs` to `false` and add `v-bind="$attrs"` to the element that will receive the attribute.

## Accessibility Testing and Checking

Have we created an accessible component? That's very nice!

But there is something more important than creating a component. But there is something more important than creating a component: making sure it works properly.

So, let's do some testing and checking to make sure that the component we have created is really OK.

### Linter

First, check with a static analysis tool.

If you have eslint installed, I recommend using [eslint-plugin-vuejs-accessibility](https://github.com/vue-a11y/eslint-plugin-vuejs-accessibility).

This is an eslint plugin that will point out if there is code in your Vue.js code that compromises accessibility.

Depending on the site or application you are building, there may be rules that are unnecessary. It is a good idea to consider what rules to enable before implementing.

If you want to be more markup oriented and check the accuracy of your HTML, it is also useful to introduce [markuplint](https://github.com/markuplint/markuplint).

![Screenshot: markuplint official site](../images/markuplint.png)

There is a Vue.js parser that can be used in conjunction with markuplint to check within the vue component.

However, markuplint is not able to evaluate the content of attributes that are dynamic bindings, so they will not be passed through.

```html
<!-- The alt contains nothing, but is not evaluated by markuplint. -->
<img src="image.png" :alt="null">
```

In this case, you may want to use the aforementioned TypeScript type definitions and `props` to receive the correct content from the parent, or use markuplint to check static HTML output or snapshot test HTML.

### Integration test

Have you deployed [Jest](https://jestjs.io/) for front-end testing? If so, you are in luck.

Let's use [testing-library](https://testing-library.com/) to write a coupled test of the behavior for an assistive technology user.

```js
import { render, screen } from '@testing-library/vue';
import keyboardOnlyUserEvent from 'keyboard-testing-library/dist/require';
import IncrementButton from './IncrementButton.vue';

test('Click events are functioning with keyboard operation', async () => {
  render(IncrementButton);
  const button = screen.getByRole('button', {name: 'increment'});

  await keyboardOnlyUserEvent.navigateToAndPressEnter(button);
  await keyboardOnlyUserEvent.navigateToAndPressEnter(button);

  expect(screen.getByRole('region', {name: 'Times clicked: 2')).toBeTruthy();
});
```

This test case uses `getByRole` to get the `role` of an element and check the internal text.

It also uses the [testing-library extension that does keyboard testing](https://github.com/Grunet/keyboard-testing-library) to check if it works correctly.

You can also use `userEvent` to move tabs and click, but this is simpler and easier to understand.

If you can use the correct markup and WAI-ARIA, you can write your test code in a way that respects the semantics.

In addition, you can use [jest-axe](https://github.com/nickcolley/jest-axe) to check for overall accessibility violations.

```js
import { render } from '@testing-library/vue';
import { axe, toHaveNoViolations } from 'jest-axe';
import App from './App.vue';

expect.extend(toHaveNoViolations);

it('Rendered results do not violate accessibility', async () => {
  const { container } = render(<App/>);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

This is a Jest matcher that uses the accessibility verification library engine called [axe](https://www.deque.com/axe/).

Like eslint, the rules can be adjusted on your own, so you can scrutinize them as needed.

### Check Browser and Assistive Technology

If you have verified accessibility correctness with the test code, the next step is to check it with actual browsers and assistive technologies as well.

[vue-axe](https://github.com/vue-a11y/vue-axe-next) is a plugin that allows you to check accessibility in the browser.

![Screenshot: Development screen using vue-axe](../images/vue-axe.png)

This plugin is useful for squashing problems while doing local development.

Keep an eye on areas you may not have noticed in the code, including contrast ratios.

#### Check keyboard operation

The previous test described keyboard operation, but let's also check if keyboard operation is possible in the actual browser.

For form elements and links, check if the tab key can be used for transitions and operations.

Alternatively, you can check the focus behavior through E2E testing.

Check that you can focus in the expected order.

You can use the extension [taba11y](https://chrome.google.com/webstore/detail/taba11y/aocppmckdocdjkphmofnklcjhdidgmga) to visualize the focus order in the browser.

Incidentally, Firefox's DevTools provides a standard function to check the focus order.

#### Screen reader

Let's check how screen reader, one of the assistive technologies, are used in practice.

Windows and macOS come standard with a screen reader, and for Windows only, there is an OSS screen reader called [NVDA](https://www.nvda.jp/en/).

There are also tools[^4] available for automated testing, although only for certain assistive technologies.

## Conclusion

There are many other accessibility considerations beyond those discussed in this article. It is not something that should only be considered in front-end development.

However, there is no such thing as perfect accessibility support, so it is important to fix mistakes as you find them.

Want to venture further into the realm of Vue.js development and accessibility?

![Screenshot: Accessible Vue official site](../images/accessible-vue.png)

Mr. Marcus, a Web accessibility specialist, has made the content [Accessible Vue](https://accessible-vue.com/) available free of charge.

For those who wish to explore more, please also see this page.

We hope that this presentation will lead to more Vue.js development with accessibility in mind and the creation of accessible sites and applications.

That concludes my presentation. Thank you for your attention.

## Reference

- [WebAIM: The WebAIM Million - The 2022 report on the accessibility of the top 1,000,000 home pages](https://webaim.org/projects/million/)
- [Accessible Rich Internet Applications (WAI-ARIA) 1.1](https://www.w3.org/TR/wai-aria-1.1/)
- [Accessibility Object Model | aom](https://wicg.github.io/aom/explainer.html)
- [WAI-ARIA勉強会 - YouTube](https://www.youtube.com/watch?v=ZLL0_W5w1vo)

[^1]: [https://webaim.org/projects/million/2021](https://webaim.org/projects/million/2021), [https://webaim.org/projects/million/2020](https://webaim.org/projects/million/2020)
[^2]: [Read Me First | APG | WAI | W3C](https://www.w3.org/WAI/ARIA/apg/practices/read-me-first/)
[^3]: [https://router.vuejs.org/api/interfaces/routerlinkprops.html#ariacurrentvalue](https://router.vuejs.org/api/interfaces/routerlinkprops.html#ariacurrentvalue)
[^4]: [Guidepup](https://github.com/guidepup/guidepup), [web-test-runner-voiceover](https://github.com/blueprintui/web-test-runner-voiceover), [VoiceOver.js](https://github.com/AccessLint/screenreaders)
