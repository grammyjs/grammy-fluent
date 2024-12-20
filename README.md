
# @grammyjs/fluent

<!-- NPM Badge -->
<a href="https://badge.fury.io/js/@grammyjs%2Ffluent">
  <img src="https://badge.fury.io/js/@grammyjs%2Ffluent.svg" alt="npm version" height="18">
</a>

<!-- MIT License Badge -->
<a href="https://opensource.org/licenses/MIT">
  <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License: MIT" height="20">
</a>

[Fluent][fluent-website] localization system integration
for [grammY][grammy-website] Telegram bot framework.


## Why Fluent?

I've studied several i18n standards and message formats and
have found that Fluent has all the required features and
at the same time provides a very user-friendly message format,
which is very important for non-dev people (i.e. translators).

It is also supported by **Mozilla Foundation**, a well-respected
leader in world of OpenSource and the Web standards.

### Message format example

Consider the following message format example and see for yourself:

```fluent
-project-name = Super Project

welcome = Welcome, {$name}, to the {-project-name}!
  .balance =
    Your balance is: {
      NUMBER($value, maximumFractionDigits: 2)
    }
  .apples-count =
    You have { NUMBER($applesCount) ->
      [0] no apples
      [one] {$applesCount} apple
      *[other] {$applesCount} apples
    }
```

### Fluent features

- Variable substitution (aka placeables),

- Built-in and custom formatters that could be applied
  to the values of the rendered variables,

- Conditional substitution (selection) based on variable value,

- Powerful pluralization with built-in rules for every locale.


## Library features

- **Built on top** of [@moebius/fluent][moebius-fluent] library,
  that on itself simplifies Fluent integration,

- **Adds helper functions** to grammY bots context to
  simplify message translation,

- Automatically uses translation locale based on the language
  selected by the user in their Telegram settings,

- Uses an **automatic language negotiation**, so the best
  possible language will be automatically picked for each user,

- Gives you **full access to `Fluent` instance**, so you can
  [configure it][moebius-fluent] yourself as you see fit,

- Uses **peer dependencies** so that you can use a wide
  combination of library versions in your project,

- **All LTS Node.js versions are supported**
  (starting from Node 12),

- **Written completely in TypeScript** from scratch in
  a very strict manner with 100% type coverage
  (and no *any*'s), ensuring that the library code is
  correct (type safe) by itself and also **provides high
  quality typing declarations** to make sure that your
  code is also correct and type safe,

- **Enables amazing type completion** for your IDE
  (even if you are not using TypeScript) thanks to the
  provided typing declarations,

- **Minimal possible dependencies**
  (all are high quality ones) updated to the latest versions,

- **Source maps** for the library is generated and provided
  to you for easier debugging.


## Install

Install the libraries:

```shell
npm install --save @grammyjs/fluent @moebius/fluent
```


## Prior knowledge

> It is highly advisable to read [@moebius/fluent][moebius-fluent]
> as well as [fluent.js][fluent-js] libraries documentation
> before using this library.


## Usage

```typescript
import { Bot, Context } from 'grammy';
import { Fluent } from '@moebius/fluent';
import { useFluent, FluentContextFlavor } from '@grammyjs/fluent';

// Extend your application context type with the provided
// flavor interface
export type MyAppContext = (
  & Context
  & FluentContextFlavor
);

// Create grammY bot as usual,
// specify the extended context
const bot = new Bot<MyAppContext>();

// Create an instance of @moebius/fluent and configure it
const fluent = new Fluent();

// Add translations that you need
await fluent.addTranslation({
  locales: 'en',
  source: `
-brand-name = Super Project

welcome =
  Welcome, {$name}, to the {-brand-name}!
  Your balance is: {
    NUMBER($value, maximumFractionDigits: 2)
  }
  You have { NUMBER($applesCount) ->
    [0] no apples
    [one] {$applesCount} apple
    *[other] {$applesCount} apples
  }
  `,

  // All the aspects of Fluent are highly configurable
  bundleOptions: {
    // Use this option to avoid invisible characters
    // around placeables
    useIsolating: false,
  },
});

// You can also load translations from files
await fluent.addTranslation({
  locales: 'ru',
  filePath: [
    `${__dirname}/feature-1/translation.ru.ftl`,
    `${__dirname}/feature-2/translation.ru.ftl`
  ],
});

// Add fluent middleware to the bot
bot.use(useFluent({
  fluent,
}));

bot.command('i18n_test', async context => {

  // Call the "translate" or "t" helper to render the
  // message by specifying it's ID and
  // additional parameters:
  await context.reply(context.t('welcome', {
    name: context.from.first_name,
    value: 123.456,
    applesCount: 1,
  }));

  // The locale to use will be detected automatically

});
```


## API

### useFluent

```typescript
function useFluent(options: GrammyFluentOptions): Middleware;
```

Call this function to add Fluent middleware to your bot, e.g:

```typescript
bot.use(useFluent({
  fluent,
}));
```

The following options are supported:

| Name             | Type             | Description                                                  |
|------------------| ---------------- | ------------------------------------------------------------ |
| fluent *         | Fluent           | A pre-configured instance of [Fluent][moebius-fluent] to use. |
| defaultLocale    | LocaleId         | A locale ID to use by default. This is used when locale negotiator returns an empty result. The default value is: "*en*". |
| localeNegotiator | LocaleNegotiator | An optional function that determines a locale to use. Check the [locale negotiation](#locale-negotiation) section below for more details. |

> Please, see [@moebius/fluent documentation][moebius-fluent]
  for all the Fluent configuration instructions.

### Context helpers

The following helpers are added to the bots' context by the
middleware:

| Name                       | Type                                                         | Description                                                  |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| fluent                     | Object                                                       | Fluent context namespace object, see the individual properties below. |
| fluent.instance            | Fluent                                                       | An instance of [Fluent][moebius-fluent].                     |
| fluent.renegotiateLocale() | () => Promise<void>                                          | You can manually trigger additional locale negotiation by calling this method. This could be useful if locale negotiation conditions has changed and new locale must be applied (e.g. user has changed the language and you need to display an answer in new locale). |
| fluent.useLocale()         | (localeId: string) => void                                   | Sets the specified locale to be used for future translations. Effect lasts only for the duration of current update and is not preserved. Could be used to change the translation locale in the middle of update processing (e.g. when user changes the language). |
| translate \| t             | (**messageId**: string, **context?**: TranslationContext) => string | Translation function bound to the current locale. Shorthand alias "t" is also available. |

Make sure to use `FluentContextFlavor` to extend your
application context in order for typings to work correctly:

```typescript
import { Context } from 'grammy';
import { FluentContextFlavor } from '@moebius/fluent';

export type MyAppContext = (
  & Context
  & FluentContextFlavor
);

const bot = new Bot<MyAppContext>();
```

### Locale negotiation

You can use a `localeNegotiator` property to define a
custom locale negotiation function that will be called for
each Telegram update and must return a locale ID to use for
message translation.

The default negotiator will detect locale based on users
Telegram language setting.

> Locale negotiation normally happens only once during
> Telegram update processing. However, you can call
> `await context.fluent.renegotiateLocale()` to call the
> negotiator again and determine the new locale. This is
> useful if the locale changes during single update
> processing.

#### API

```typescript
type LocaleNegotiator<ContextType> = (
  (context: ContextType) => (LocaleId | PromiseLike<LocaleId>)
);
```

#### Example

The example below will try to use locale ID stored
in users session:

```typescript
async function myLocaleNegotiator(context: Context) {
  return (
    (await context.session).languageId ||
    context.from.language_code ||
    'en'
  );
}

bot.use(useFluent({
  fluent,

  // Telling middleware to use our custom negotiator
  localeNegotiator: myLocaleNegotiator,

  defaultLocale: 'en', // this is the default
}));
```


## Cookbook

### i18n plugin replacement

If you were using the [official i18n plugin][i18n-plugin]
with session storage, you can easily replace it using
the following code:

```typescript
bot.use(useFluent({
  fluent,
  localeNegotiator: async context => (
    (await context.session).__language_code
  ),
}));

async function handleLocaleChange(context: MyAppContext) {

  // Getting locale from button's callback data
  const newLocale = context.callbackQuery;

  // Saving new locale to the session
  (await context.session).__language_code = newLocale;

  // Making sure that callback query answer will be
  // in new locale
  await context.fluent.useLocale(locale);

  // Sending an answer to the callback query
  await context.answerCallbackQuery(
    context.t('settings_language-changed', {
      language: context.t(`locale_${locale}`)
    })
  );

}
```

It will use the locale that is already stored in the user
session by the i18n plugin.


## Contributors

- [Slava Fomin II](https://github.com/slavafomin) (author)
- [dcdunkan](https://github.com/dcdunkan)

## License (MIT)

Copyright © 2021—2024 Slava Fomin II

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.



  [grammy-website]: https://grammy.dev/
  [fluent-website]: https://projectfluent.org/
  [fluent-js]: https://github.com/projectfluent/fluent.js/
  [moebius-fluent]: https://github.com/the-moebius/fluent
  [i18n-plugin]: https://github.com/grammyjs/i18n
