# Pluralization

## Overview

Pluralization is one of the most complex aspects of internationalization. While English has two plural forms (singular and plural), many languages have different rules. Arabic has six forms, Slavic languages have three, and others have their own unique patterns. Understanding pluralization rules and how to implement them correctly is essential for building truly international applications.

## Table of Contents
- [Plural Rules Across Languages](#plural-rules-across-languages)
- [CLDR Plural Categories](#cldr-plural-categories)
- [ICU MessageFormat](#icu-messageformat)
- [Implementation with i18next](#implementation-with-i18next)
- [Intl.PluralRules API](#intlpluralsrules-api)
- [Edge Cases and Common Mistakes](#edge-cases-and-common-mistakes)
- [Testing Pluralization](#testing-pluralization)
- [Performance Considerations](#performance-considerations)
- [Interview Questions](#interview-questions)

---

## Plural Rules Across Languages

### English (2 forms)

English has the simplest plural system: singular (1) and other (everything else).

```javascript
// Plural categories: one, other
const englishRules = {
  one: [1],           // singular: 1 item
  other: [0, 2, 3, 4, 5, ...] // plural: 0, 2, 3, ... items
};

// Examples
const examples = [
  { count: 0, form: 'other', text: '0 messages' },
  { count: 1, form: 'one', text: '1 message' },
  { count: 2, form: 'other', text: '2 messages' },
  { count: 21, form: 'other', text: '21 messages' }
];

// Translation file
// en.json
{
  "messages": "You have {{count}} message",
  "messages_plural": "You have {{count}} messages"
}

// Usage
t('messages', { count: 1 })  // "You have 1 message"
t('messages', { count: 2 })  // "You have 2 messages"
```

### Russian (3 forms)

Russian has three plural forms based on the number:

```javascript
// Plural categories: one, few, many, other
// one: ends in 1 (except 11)
// few: ends in 2-4 (except 12-14)
// many: other numbers
// other: (fallback)

const russianRules = {
  one: [1, 21, 31, 41, 51, ...],      // 1, 21, 31, ...
  few: [2, 3, 4, 22, 23, 24, ...],    // 2-4, 22-24, ...
  many: [0, 5, 6, 7, 8, 9, 10, 11, ...], // 0, 5-20, 25-30, ...
};

// Examples
const examples = [
  { count: 0, form: 'many', text: '0 A>>1I5=89' },
  { count: 1, form: 'one', text: '1 A>>1I5=85' },
  { count: 2, form: 'few', text: '2 A>>1I5=8O' },
  { count: 5, form: 'many', text: '5 A>>1I5=89' },
  { count: 21, form: 'one', text: '21 A>>1I5=85' },
  { count: 22, form: 'few', text: '22 A>>1I5=8O' }
];

// Translation file
// ru.json
{
  "messages": "{{count}} A>>1I5=85",
  "messages_few": "{{count}} A>>1I5=8O",
  "messages_many": "{{count}} A>>1I5=89"
}
```

### Arabic (6 forms)

Arabic has the most complex pluralization with six different forms:

```javascript
// Plural categories: zero, one, two, few, many, other
// zero: 0
// one: 1
// two: 2
// few: 3-10
// many: 11-99
// other: 100+

const arabicRules = {
  zero: [0],
  one: [1],
  two: [2],
  few: [3, 4, 5, 6, 7, 8, 9, 10],
  many: [11, 12, 13, ..., 99],
  other: [100, 101, ...] // 100+
};

// Examples
const examples = [
  { count: 0, form: 'zero', text: 'D' *H,/ 13'&D' },      // no messages
  { count: 1, form: 'one', text: '13'D) H'-/)' },         // 1 message
  { count: 2, form: 'two', text: '13'D*'F' },             // 2 messages
  { count: 5, form: 'few', text: '5 13'&D' },             // 3-10 messages
  { count: 50, form: 'many', text: '50 13'D)' },          // 11-99 messages
  { count: 100, form: 'other', text: '100 13'D)' }        // 100+ messages
];

// Translation file
// ar.json
{
  "messages": "D' *H,/ 13'&D",
  "messages_one": "13'D) H'-/)",
  "messages_two": "13'D*'F",
  "messages_few": "{{count}} 13'&D",
  "messages_many": "{{count}} 13'D)",
  "messages_other": "{{count}} 13'D)"
}
```

### German (2 forms, like English)

```javascript
// Plural categories: one, other
const germanRules = {
  one: [1],
  other: [0, 2, 3, 4, ...]
};

// Translation file
// de.json
{
  "messages": "{{count}} Nachricht",
  "messages_plural": "{{count}} Nachrichten"
}

// Usage
t('messages', { count: 1 })  // "1 Nachricht"
t('messages', { count: 2 })  // "2 Nachrichten"
```

### Polish (3 forms)

```javascript
// Plural categories: one, few, many, other
// one: 1
// few: ends in 2-4 (except 12-14)
// many: other cases
// other: (fallback)

const polishRules = {
  one: [1],                           // 1
  few: [2, 3, 4, 22, 23, 24, ...],   // 2-4, 22-24, ...
  many: [0, 5, 6, ..., 21, 25, ...], // 0, 5-21, 25+
};

// Translation file
// pl.json
{
  "messages": "{{count}} wiadomo[",
  "messages_few": "{{count}} wiadomo[ci",
  "messages_many": "{{count}} wiadomo[ci"
}
```

---

## CLDR Plural Categories

### Understanding CLDR

The **Common Locale Data Repository (CLDR)** defines plural rules for every language. It specifies which numbers belong to which plural category for each language.

```javascript
// CLDR defines pluralization for 300+ languages
// Each language has categories it supports:
// Possible categories: zero, one, two, few, many, other

// Example language categories
const cldrCategories = {
  english: ['one', 'other'],           // 2 categories
  russian: ['one', 'few', 'many', 'other'], // 4 categories
  arabic: ['zero', 'one', 'two', 'few', 'many', 'other'], // 6 categories
  french: ['one', 'many', 'other'],    // 3 categories
  chinese: ['other']                   // 1 category (no pluralization)
};

// CLDR rules use mathematical formulas
// Example: Russian rule for 'one' category
// (n % 10 == 1 && n % 100 != 11) -> belongs to 'one'

// This means:
// n=1: (1 % 10 == 1 && 1 % 100 != 11) = true  -> 'one'
// n=11: (11 % 10 == 1 && 11 % 100 != 11) = false -> not 'one'
// n=21: (21 % 10 == 1 && 21 % 100 != 11) = true  -> 'one'
```

### Checking Supported Categories for a Language

```javascript
// Use Intl.PluralRules to check categories
const intlCategories = {
  english: new Intl.PluralRules('en').resolvedOptions().pluralCategories,
  // Result: ['one', 'other']

  russian: new Intl.PluralRules('ru').resolvedOptions().pluralCategories,
  // Result: ['one', 'few', 'many', 'other']

  arabic: new Intl.PluralRules('ar').resolvedOptions().pluralCategories,
  // Result: ['zero', 'one', 'two', 'few', 'many', 'other']
};

// Check which category a number belongs to
function getCategory(number, locale) {
  const pluralRules = new Intl.PluralRules(locale);
  return pluralRules.select(number);
}

getCategory(0, 'en')    // 'other'
getCategory(1, 'en')    // 'one'
getCategory(2, 'en')    // 'other'
getCategory(0, 'ar')    // 'zero'
getCategory(1, 'ar')    // 'one'
getCategory(2, 'ar')    // 'two'
getCategory(5, 'ar')    // 'few'
```

---

## ICU MessageFormat

### Basic Syntax

The **ICU MessageFormat** is an industry standard for handling pluralization and other message formatting.

```javascript
// Basic ICU format for pluralization
// {variable, plural, singular_form {text} plural_form {text}}

// English example
const englishICU = "{count, plural, one {# item} other {# items}}";

// Usage
// count = 1 -> "1 item"
// count = 5 -> "5 items"
// # is replaced with the actual count

// Russian example (3 forms)
const russianICU = "{count, plural, one {# A>>1I5=85} few {# A>>1I5=8O} many {# A>>1I5=89}}";

// Arabic example (6 forms)
const arabicICU = "{count, plural, zero {=5B A>>1I5=89} one {# A>>1I5=85} two {# A>>1I5=8O} few {# A>>1I5=8O} many {# A>>1I5=89} other {# A>>1I5=89}}";
```

### Advanced ICU Features

```javascript
// 1. Nested formatting
"{name} has {count, plural, one {# book} other {# books}}";
// "Alice has 1 book"
// "Bob has 3 books"

// 2. Offset formatting (for natural language)
"{count, plural, offset:1, one {You} one {# other person} other {# other people}}";
// count = 1: "You"
// count = 2: "1 other person"
// count = 3: "2 other people"

// 3. Select with plural
"{gender, select, male {He} female {She} other {They}} have {count, plural, one {# book} other {# books}}";
// gender=male, count=1: "He has 1 book"
// gender=female, count=3: "She has 3 books"

// 4. Complex nesting
"{gender, select, male {He} female {She} other {They}} {action, select, buy {bought} sell {sold} other {have}} {count, plural, one {# book} other {# books}}";
// "She bought 3 books"
```

### ICU in JSON Format

```javascript
// How ICU messages look in translation files

// en.json
{
  "itemCount": "{count, plural, one {# item} other {# items}}",
  "userAction": "{user} {action, select, buy {bought} read {read} other {has}} {count, plural, one {# book} other {# books}}",
  "notificationMessage": "{name} sent you {count, plural, one {# message} other {# messages}}"
}

// Usage with i18n
i18n.t('itemCount', { count: 1 })  // "1 item"
i18n.t('itemCount', { count: 5 })  // "5 items"

// Usage with FormatJS
intl.formatMessage({ id: 'itemCount' }, { count: 5 })  // "5 items"
```

---

## Implementation with i18next

### Setting Up Pluralization

```javascript
// i18n.js
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import enTranslation from './locales/en.json';
import ruTranslation from './locales/ru.json';
import arTranslation from './locales/ar.json';

i18n.use(initReactI18next).init({
  resources: {
    en: { translation: enTranslation },
    ru: { translation: ruTranslation },
    ar: { translation: arTranslation }
  },
  lng: 'en',
  fallbackLng: 'en',

  // Pluralization configuration
  pluralSeparator: '_',  // Default: '_one', '_other', '_plural'

  // Handle pluralization correctly
  interpolation: {
    escapeValue: false,
    formatSeparator: ','
  },

  // These settings work automatically with i18next
  // It uses CLDR rules internally
});

export default i18n;
```

### Translation File Structure

```javascript
// en.json - English (2 plural forms)
{
  "messages": "You have {{count}} message",
  "messages_plural": "You have {{count}} messages",

  "notifications": "You have {{count}} notification",
  "notifications_plural": "You have {{count}} notifications"
}

// ru.json - Russian (3+ plural forms)
{
  "messages": "# 20A {{count}} A>>1I5=85",
  "messages_few": "# 20A {{count}} A>>1I5=8O",
  "messages_many": "# 20A {{count}} A>>1I5=89",

  "followers": "{{count}} ?>4?8AG8:",
  "followers_few": "{{count}} ?>4?8AG8:0",
  "followers_many": "{{count}} ?>4?8AG8:>2"
}

// ar.json - Arabic (6 plural forms)
{
  "messages": "DJ3 D/JC 13'&D",
  "messages_one": "D/JC {{count}} 13'D)",
  "messages_two": "D/JC {{count}} 13'D*'F",
  "messages_few": "D/JC {{count}} 13'&D",
  "messages_many": "D/JC {{count}} 13'D)",
  "messages_other": "D/JC {{count}} 13'D)"
}
```

### Using Pluralization in Components

```javascript
import { useTranslation } from 'react-i18next';

// Basic usage
function MessageCount({ count }) {
  const { t } = useTranslation();

  return <p>{t('messages', { count })}</p>;
  // count=1: "You have 1 message"
  // count=5: "You have 5 messages"
}

// With all languages
function MultiLanguageCount({ count }) {
  const { t } = useTranslation();

  // Same code works for all languages!
  // i18next automatically handles plural forms
  return <p>{t('messages', { count })}</p>;
}

// Dynamic lists
function NotificationList({ notifications }) {
  const { t } = useTranslation();

  return (
    <div>
      <h2>{t('notifications', { count: notifications.length })}</h2>
      <ul>
        {notifications.map(notif => (
          <li key={notif.id}>{notif.message}</li>
        ))}
      </ul>
    </div>
  );
}

// Custom suffix handling
function AdvancedPluralization({ count, namespace = 'common' }) {
  const { t } = useTranslation(namespace);

  // Manually specify plural form (if needed)
  const suffix = getPluralSuffix(count, 'en');
  const key = `items${suffix}`;

  return <p>{t(key, { count })}</p>;
}

function getPluralSuffix(count, lang) {
  if (lang === 'en') {
    return count === 1 ? '' : '_plural';
  }
  if (lang === 'ru') {
    // Complex Russian rules
    if (count % 10 === 1 && count % 100 !== 11) return '';
    if (count % 10 >= 2 && count % 10 <= 4 && !(count % 100 >= 12 && count % 100 <= 14)) return '_few';
    return '_many';
  }
  return '_plural';
}
```

---

## Intl.PluralRules API

### Native Browser API

```javascript
// Intl.PluralRules - native browser API for pluralization
// No dependencies needed!

// Basic usage
const pluralRules = new Intl.PluralRules('en');
pluralRules.select(0)   // 'other'
pluralRules.select(1)   // 'one'
pluralRules.select(2)   // 'other'

// Different locales
const enRules = new Intl.PluralRules('en');
const ruRules = new Intl.PluralRules('ru');
const arRules = new Intl.PluralRules('ar');

ruRules.select(0)   // 'many'
ruRules.select(1)   // 'one'
ruRules.select(2)   // 'few'
ruRules.select(5)   // 'many'

arRules.select(0)   // 'zero'
arRules.select(1)   // 'one'
arRules.select(2)   // 'two'
arRules.select(5)   // 'few'
```

### Building a Custom Pluralization System

```javascript
// Create a custom pluralization function
class PluralFormatter {
  constructor(locale = 'en', translations = {}) {
    this.pluralRules = new Intl.PluralRules(locale);
    this.locale = locale;
    this.translations = translations;
  }

  format(key, count) {
    const category = this.pluralRules.select(count);
    const message = this.translations[key];

    if (!message) {
      console.warn(`Missing translation key: ${key}`);
      return key;
    }

    // Handle both ICU format and simple suffix format
    if (typeof message === 'string') {
      return message.replace('{{count}}', count);
    }

    // For object format
    const form = message[category] || message.other;
    if (!form) {
      console.warn(`Missing plural form for key: ${key}, category: ${category}`);
      return key;
    }

    return form.replace('{{count}}', count);
  }
}

// Usage
const translations = {
  messages: {
    one: 'You have {{count}} message',
    other: 'You have {{count}} messages'
  },
  followers: {
    one: '{{count}} follower',
    few: '{{count}} followers',
    other: '{{count}} followers'
  }
};

const formatter = new PluralFormatter('en', translations);
formatter.format('messages', 1)   // "You have 1 message"
formatter.format('messages', 5)   // "You have 5 messages"
```

### Advanced: ICU MessageFormat Parsing

```javascript
// Parse and format ICU messages
function parseICUMessage(message, values = {}) {
  // Extract plural rule: {variable, plural, ...}
  const pluralMatch = message.match(/\{(\w+),\s*plural,\s*(.+?)\}/);

  if (!pluralMatch) return message;

  const [, variable, forms] = pluralMatch;
  const count = values[variable];

  if (count === undefined) return message;

  // Parse forms: "one {text} other {text}"
  const formPattern = /(\w+)\s*\{([^}]+)\}/g;
  const formMap = {};
  let match;

  while ((match = formPattern.exec(forms)) !== null) {
    formMap[match[1]] = match[2];
  }

  // Get appropriate form using PluralRules
  const pluralRules = new Intl.PluralRules();
  const form = pluralRules.select(count);
  const selectedForm = formMap[form] || formMap.other;

  // Replace # with count and variables
  return selectedForm
    .replace(/#/g, count)
    .replace(/\{\{(\w+)\}\}/g, (match, key) => values[key] || match);
}

// Usage
const message = "{count, plural, one {# item} other {# items}}";
parseICUMessage(message, { count: 1 })   // "1 item"
parseICUMessage(message, { count: 5 })   // "5 items"

const complex = "{name} has {count, plural, one {# book} other {# books}}";
parseICUMessage(complex, { name: 'Alice', count: 3 })
// "Alice has 3 books"
```

---

## Edge Cases and Common Mistakes

### 1. Off-by-One Errors

```javascript
// WRONG: Forgetting that 0 is also plural
const bad = {
  one: 'You have 1 item',
  other: 'You have multiple items'
};
// 0 items -> "You have multiple items"  Actually correct for English!

// Another wrong approach: Using other conditions
if (count === 1) {
  msg = 'item';
} else {
  msg = 'items';
}
// This works for English but NOT for other languages!

// RIGHT: Use built-in plural rules
t('items', { count }) // Works for all languages
```

### 2. Missing Plural Forms

```javascript
// WRONG: Only providing singular form for Russian
// ru.json
{
  "items": "{{count}} B>20@"  // Only singular!
}

// RIGHT: Provide all forms needed by the language
// ru.json
{
  "items": "{{count}} B>20@",
  "items_few": "{{count}} B>20@0",
  "items_many": "{{count}} B>20@>2"
}
```

### 3. Forgetting to Include Count in Message

```javascript
// WRONG: Translation doesn't include count
// en.json
{
  "messages": "Message",
  "messages_plural": "Messages"
}

// RIGHT: Include the count variable
// en.json
{
  "messages": "You have {{count}} message",
  "messages_plural": "You have {{count}} messages"
}
```

### 4. Not Handling Zero

```javascript
// WRONG: Not considering zero case
t('items', { count: 0 })  // "You have 0 items" (grammatically weird)

// RIGHT: Special handling for zero
// en.json
{
  "items": "You have {{count}} item",
  "items_plural": "You have {{count}} items",
  "items_zero": "You have no items"  // Add special handling for 0
}
```

### 5. Context-Dependent Pluralization

```javascript
// WRONG: Same key used in different contexts
{
  "items": "You have {{count}} items"
}

// RIGHT: Different keys for different contexts
{
  "cart_items": "{{count}} item in cart",
  "cart_items_plural": "{{count}} items in cart",
  "list_items": "Showing {{count}} item",
  "list_items_plural": "Showing {{count}} items"
}
```

---

## Testing Pluralization

### Unit Tests

```javascript
// Using Jest + React Testing Library
import { render, screen } from '@testing-library/react';
import { I18nextProvider } from 'react-i18next';
import i18n from '../i18n';
import MessageCount from '../components/MessageCount';

describe('MessageCount Pluralization', () => {
  beforeEach(() => {
    i18n.changeLanguage('en');
  });

  test('shows singular for count=1', () => {
    render(
      <I18nextProvider i18n={i18n}>
        <MessageCount count={1} />
      </I18nextProvider>
    );
    expect(screen.getByText('You have 1 message')).toBeInTheDocument();
  });

  test('shows plural for count=0', () => {
    render(
      <I18nextProvider i18n={i18n}>
        <MessageCount count={0} />
      </I18nextProvider>
    );
    expect(screen.getByText('You have 0 messages')).toBeInTheDocument();
  });

  test('shows plural for count>1', () => {
    render(
      <I18nextProvider i18n={i18n}>
        <MessageCount count={5} />
      </I18nextProvider>
    );
    expect(screen.getByText('You have 5 messages')).toBeInTheDocument();
  });
});

// Test for different languages
describe('MessageCount - Russian Pluralization', () => {
  beforeEach(() => {
    i18n.changeLanguage('ru');
  });

  const testCases = [
    { count: 0, expected: '# 20A 0 A>>1I5=89' },
    { count: 1, expected: '# 20A 1 A>>1I5=85' },
    { count: 2, expected: '# 20A 2 A>>1I5=8O' },
    { count: 5, expected: '# 20A 5 A>>1I5=89' },
    { count: 21, expected: '# 20A 21 A>>1I5=85' }
  ];

  testCases.forEach(({ count, expected }) => {
    test(`count=${count} shows correct form`, () => {
      render(
        <I18nextProvider i18n={i18n}>
          <MessageCount count={count} />
        </I18nextProvider>
      );
      expect(screen.getByText(expected)).toBeInTheDocument();
    });
  });
});

// Test with Intl.PluralRules directly
describe('Intl.PluralRules', () => {
  test('English plural forms', () => {
    const rules = new Intl.PluralRules('en');
    expect(rules.select(0)).toBe('other');
    expect(rules.select(1)).toBe('one');
    expect(rules.select(2)).toBe('other');
  });

  test('Russian plural forms', () => {
    const rules = new Intl.PluralRules('ru');
    expect(rules.select(1)).toBe('one');
    expect(rules.select(2)).toBe('few');
    expect(rules.select(5)).toBe('many');
  });

  test('Arabic plural forms', () => {
    const rules = new Intl.PluralRules('ar');
    expect(rules.select(0)).toBe('zero');
    expect(rules.select(1)).toBe('one');
    expect(rules.select(2)).toBe('two');
    expect(rules.select(5)).toBe('few');
  });
});
```

### Property-Based Testing

```javascript
// Using fast-check for property-based testing
import fc from 'fast-check';

describe('Pluralization Properties', () => {
  test('plural form is always defined for any count', () => {
    fc.assert(
      fc.property(fc.integer({ min: 0, max: 1000000 }), (count) => {
        const rules = new Intl.PluralRules('en');
        const form = rules.select(count);
        return ['one', 'other'].includes(form);
      })
    );
  });

  test('each count maps to exactly one plural category', () => {
    const testLocales = ['en', 'ru', 'ar', 'pl', 'de'];

    testLocales.forEach(locale => {
      fc.assert(
        fc.property(fc.integer({ min: 0, max: 100 }), (count) => {
          const rules = new Intl.PluralRules(locale);
          const form = rules.select(count);
          // Form should be one of the supported categories
          const categories = rules.resolvedOptions().pluralCategories;
          return categories.includes(form);
        })
      );
    });
  });
});
```

---

## Performance Considerations

### 1. Avoiding Excessive Instantiation

```javascript
// WRONG: Creating PluralRules for every call
function getPluralForm(count, locale) {
  const rules = new Intl.PluralRules(locale); // Created every time!
  return rules.select(count);
}

// RIGHT: Memoize instances
const pluralRulesCache = {};

function getPluralForm(count, locale) {
  if (!pluralRulesCache[locale]) {
    pluralRulesCache[locale] = new Intl.PluralRules(locale);
  }
  return pluralRulesCache[locale].select(count);
}

// Or use a class
class PluralHelper {
  constructor() {
    this.cache = new Map();
  }

  getForm(count, locale) {
    if (!this.cache.has(locale)) {
      this.cache.set(locale, new Intl.PluralRules(locale));
    }
    return this.cache.get(locale).select(count);
  }
}
```

### 2. Translation File Optimization

```javascript
// WRONG: Including all forms for all keys
{
  "items": {
    "one": "Item",
    "few": "Items",
    "many": "Items",
    "other": "Items"
  },
  "messages": {
    "one": "Message",
    "few": "Messages",
    "many": "Messages",
    "other": "Messages"
  }
}

// RIGHT: Only include needed forms
{
  "items": "Item",
  "items_plural": "Items",
  "messages": "Message",
  "messages_plural": "Messages"
}

// For languages that need more:
// ru.json
{
  "items": "">20@",
  "items_few": "">20@0",
  "items_many": "">20@>2"
}
```

### 3. Caching Formatted Messages

```javascript
// React component with memoization
import { memo } from 'react';
import { useTranslation } from 'react-i18next';

const MessageCount = memo(function MessageCount({ count }) {
  const { t } = useTranslation();
  return <p>{t('messages', { count })}</p>;
});

// Or use useMemo
function MessageList({ counts }) {
  const { t } = useTranslation();

  const formatted = useMemo(
    () => counts.map(count => t('messages', { count })),
    [counts, t]
  );

  return <ul>{formatted.map((msg, i) => <li key={i}>{msg}</li>)}</ul>;
}
```

---

## Interview Questions

### 1. Why is pluralization language-dependent?

**Answer:**
Different languages have different grammatical rules for plurals. English uses only 2 forms (singular/plural based on count == 1), while Russian has 3 forms based on the last digit and whether it's divisible by 100, and Arabic has 6 forms. Proper internationalization must respect these language-specific rules.

### 2. What are the CLDR plural categories?

**Answer:**
CLDR (Common Locale Data Repository) defines plural categories: zero, one, two, few, many, and other. Not all languages use all categories. English uses 'one' and 'other', Arabic uses all six, and Slavic languages typically use 'one', 'few', 'many', and 'other'.

### 3. How do you implement pluralization in react-i18next?

**Answer:**
Create separate translation keys with suffixes:
```json
{ "items": "Item", "items_plural": "Items" }
```
Then use: `t('items', { count: 5 })` and i18next automatically selects the right form based on the count and language rules.

### 4. What's the difference between ICU MessageFormat and i18next plural syntax?

**Answer:**
- **ICU MessageFormat**: Industry standard format using `{count, plural, one {text} other {text}}` syntax
- **i18next**: Uses separate keys with suffixes like `items` and `items_plural`

Both achieve the same goal, but ICU is more compact while i18next is simpler to implement.

### 5. How do you handle the zero case in pluralization?

**Answer:**
Some languages and contexts require special handling for zero. You can add a `_zero` suffix or create a separate key:
```json
{
  "items": "You have {{count}} item",
  "items_plural": "You have {{count}} items",
  "items_zero": "You have no items"
}
```

### 6. What's Intl.PluralRules and why should you use it?

**Answer:**
It's a native browser API that applies the correct plural rules for any language without external libraries. Performance-efficient and always accurate according to Unicode standards. Use it when you need low-level pluralization logic.

### 7. How do you test pluralization across multiple languages?

**Answer:**
Create test cases for each language's plural forms:
- English: test count 0, 1, 2+
- Russian: test for 'one', 'few', 'many' categories
- Arabic: test all 6 categories

Use parametrized tests to avoid duplication.

### 8. What happens if a translation file is missing plural forms for a language?

**Answer:**
The application will either:
1. Fallback to the default form (error/undefined)
2. Fallback to another language's form
3. Show the key name itself

This is why validation is important - check that all necessary plural forms are provided for each language.

### 9. Can you explain Russian pluralization rules?

**Answer:**
Russian has 4 plural forms selected based on:
- **one**: n % 10 == 1 AND n % 100 != 11 (1, 21, 31...)
- **few**: n % 10 IN 2..4 AND n % 100 NOT IN 12..14 (2-4, 22-24...)
- **many**: Everything else
- **other**: Fallback

### 10. How would you optimize pluralization for performance?

**Answer:**
1. Memoize Intl.PluralRules instances
2. Cache formatted messages
3. Use React.memo for pluralization components
4. Lazy load translation files by language
5. Only include needed plural forms in bundles

---

## Navigation

**Continue Reading:**
- [03 - Date & Number Formatting](./03-date-number-formatting.md) - Format dates and numbers
- [04 - RTL Support](./04-rtl-support.md) - Support right-to-left languages

**Related Topics:**
- [01 - i18n Fundamentals](./01-i18n-fundamentals.md) - Core i18n concepts
- Frontend/React - Component patterns
- Frontend/JavaScript - String handling

---

**Last Updated:** December 2025
**Difficulty:** Intermediate
**Estimated Time:** 3-4 hours
