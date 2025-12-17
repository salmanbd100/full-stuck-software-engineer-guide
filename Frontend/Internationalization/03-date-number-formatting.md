# Date & Number Formatting

## Overview

Formatting dates, numbers, currency, and time zones correctly for different locales is critical for a truly internationalized application. While it might seem simple, different countries have vastly different conventions: dates (MM/DD/YYYY vs DD/MM/YYYY), numbers (1,000.50 vs 1.000,50), and currency symbols vary significantly. JavaScript's Intl API provides powerful native tools for handling these formatting challenges correctly and efficiently.

## Table of Contents
- [Intl API Overview](#intl-api-overview)
- [Intl.DateTimeFormat](#intldatetimeformat)
- [Intl.NumberFormat](#intlnumberformat)
- [Currency Formatting](#currency-formatting)
- [Intl.RelativeTimeFormat](#intlrelativetimeformat)
- [Intl.ListFormat](#intllistformat)
- [Time Zone Handling](#time-zone-handling)
- [Date Libraries with i18n](#date-libraries-with-i18n)
- [Common Pitfalls](#common-pitfalls)
- [Interview Questions](#interview-questions)

---

## Intl API Overview

### What is the Intl API?

The **Intl (Internationalization)** API is a native browser API that provides language-sensitive formatting and comparison. It's part of the ECMAScript standard and requires no dependencies.

```javascript
// Core Intl objects
const intlObjects = {
  'Intl.DateTimeFormat': 'Format dates and times',
  'Intl.NumberFormat': 'Format numbers and currency',
  'Intl.RelativeTimeFormat': 'Format relative times ("2 days ago")',
  'Intl.ListFormat': 'Format lists ("a, b, and c")',
  'Intl.Collator': 'String comparison with locale rules',
  'Intl.Segmenter': 'Break text into words, graphemes, sentences',
  'Intl.PluralRules': 'Determine plural categories (covered earlier)',
  'Intl.DisplayNames': 'Get localized display names for languages, regions'
};

// Browser support
const support = {
  'Intl.DateTimeFormat': 'All modern browsers',
  'Intl.NumberFormat': 'All modern browsers',
  'Intl.RelativeTimeFormat': 'Modern browsers (IE not supported)',
  'Intl.ListFormat': 'Modern browsers (Edge 79+)',
  'Intl.Segmenter': 'Newer browsers (Chrome 111+, Firefox 125+)'
};

// Check if API is supported
function hasIntlSupport(api) {
  try {
    new Intl[api]('en-US');
    return true;
  } catch (e) {
    return false;
  }
}
```

### Locale Identifier Structure

```javascript
// Locale identifiers follow BCP 47 format
// [language]-[script]-[region]-[variant]

const locales = {
  'en': 'English (any region)',
  'en-US': 'English (United States)',
  'en-GB': 'English (Great Britain)',
  'zh': 'Chinese (any script/region)',
  'zh-Hans-CN': 'Chinese (Simplified script, China)',
  'zh-Hant-TW': 'Chinese (Traditional script, Taiwan)',
  'pt-BR': 'Portuguese (Brazil)',
  'pt-PT': 'Portuguese (Portugal)',
  'ar-SA': 'Arabic (Saudi Arabia)',
  'es-ES': 'Spanish (Spain)',
  'es-MX': 'Spanish (Mexico)'
};

// The browser matches to nearest available
new Intl.DateTimeFormat('fr-CH'); // French in Switzerland
// May fall back to 'fr' if 'fr-CH' not available
```

---

## Intl.DateTimeFormat

### Basic Date Formatting

```javascript
// Create formatters
const enFormatter = new Intl.DateTimeFormat('en-US');
const deFormatter = new Intl.DateTimeFormat('de-DE');
const arFormatter = new Intl.DateTimeFormat('ar-SA');

const date = new Date('2025-12-18');

// Format the same date in different locales
enFormatter.format(date);  // "12/18/2025"
deFormatter.format(date);  // "18.12.2025"
arFormatter.format(date);  // "ah/ab/b`be"

// Different date: December 25, 2025
const xmas = new Date(2025, 11, 25);
enFormatter.format(xmas);  // "12/25/2025"
deFormatter.format(xmas);  // "25.12.2025"
```

### Date Format Options

```javascript
// Comprehensive options for date formatting
const options = {
  // Date components
  year: 'numeric' | '2-digit',
  month: 'numeric' | '2-digit' | 'long' | 'short' | 'narrow',
  day: 'numeric' | '2-digit',

  // Time components
  hour: 'numeric' | '2-digit',
  minute: 'numeric' | '2-digit',
  second: 'numeric' | '2-digit',
  hour12: true | false, // 12-hour vs 24-hour

  // Time zone and daylight saving
  timeZone: 'America/New_York', // IANA timezone
  timeZoneName: 'short' | 'long' | 'shortOffset' | 'longOffset' | 'shortGeneric' | 'longGeneric',

  // Week information
  weekday: 'long' | 'short' | 'narrow',

  // Era (BC/AD)
  era: 'long' | 'short' | 'narrow'
};

// Examples of different formats
const date = new Date('2025-12-18T14:30:00');

// Short date
new Intl.DateTimeFormat('en-US', {
  year: 'numeric',
  month: '2-digit',
  day: '2-digit'
}).format(date);  // "12/18/2025"

// Long date
new Intl.DateTimeFormat('en-US', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric'
}).format(date);  // "Thursday, December 18, 2025"

// Time only
new Intl.DateTimeFormat('en-US', {
  hour: '2-digit',
  minute: '2-digit',
  second: '2-digit',
  hour12: true
}).format(date);  // "02:30:00 PM"

// Date and time
new Intl.DateTimeFormat('de-DE', {
  year: 'numeric',
  month: '2-digit',
  day: '2-digit',
  hour: '2-digit',
  minute: '2-digit'
}).format(date);  // "18.12.2025, 14:30"

// With time zone
new Intl.DateTimeFormat('en-US', {
  year: 'numeric',
  month: 'long',
  day: 'numeric',
  timeZone: 'America/New_York',
  timeZoneName: 'short'
}).format(date);  // "December 18, 2025, EST"

// Month and day only
new Intl.DateTimeFormat('es-ES', {
  month: 'long',
  day: 'numeric'
}).format(date);  // "18 de diciembre"
```

### Predefined Format Options

```javascript
// Some locales support predefined format options (if implementation supports dateStyle/timeStyle)
new Intl.DateTimeFormat('en-US', {
  dateStyle: 'full',
  timeStyle: 'long'
}).format(date);  // "Thursday, December 18, 2025 at 2:30:00 PM EST"

new Intl.DateTimeFormat('en-US', {
  dateStyle: 'short',
  timeStyle: 'short'
}).format(date);  // "12/18/25, 2:30 PM"

new Intl.DateTimeFormat('de-DE', {
  dateStyle: 'long'
}).format(date);  // "18. Dezember 2025"
```

### Working with Date Formatters in React

```javascript
import React, { useMemo } from 'react';

// Create a custom hook for date formatting
function useDateFormatter(locale, options = {}) {
  return useMemo(() => {
    return new Intl.DateTimeFormat(locale, options);
  }, [locale, options]);
}

// Component using the hook
function DateDisplay({ date, locale = 'en-US' }) {
  const formatter = useDateFormatter(locale, {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });

  return <time dateTime={date.toISOString()}>{formatter.format(date)}</time>;
}

// Advanced: Format multiple dates efficiently
function DateList({ dates, locale = 'en-US' }) {
  const formatter = useDateFormatter(locale, {
    year: '2-digit',
    month: '2-digit',
    day: '2-digit'
  });

  return (
    <ul>
      {dates.map((date, i) => (
        <li key={i}>{formatter.format(date)}</li>
      ))}
    </ul>
  );
}
```

---

## Intl.NumberFormat

### Basic Number Formatting

```javascript
// Create formatters for different locales
const enFormatter = new Intl.NumberFormat('en-US');
const deFormatter = new Intl.NumberFormat('de-DE');
const frFormatter = new Intl.NumberFormat('fr-FR');

const number = 1234.56;

// Format in different locales
enFormatter.format(number);  // "1,234.56"
deFormatter.format(number);  // "1.234,56"
frFormatter.format(number);  // "1 234,56"

// Key difference: thousands separator and decimal separator
// en-US: comma for thousands, period for decimal
// de-DE: period for thousands, comma for decimal
// fr-FR: space for thousands, comma for decimal
```

### Number Format Options

```javascript
// Comprehensive options
const options = {
  // Style of the number
  style: 'decimal' | 'percent' | 'currency' | 'unit',

  // For currency style
  currency: 'USD' | 'EUR' | 'JPY' | ..., // ISO 4217 code
  currencySign: 'standard' | 'accounting', // accounting uses parentheses for negatives

  // For unit style
  unit: 'meter' | 'kilogram' | 'second' | ..., // CLDR simple units
  unitDisplay: 'long' | 'short' | 'narrow',

  // Decimal places
  minimumFractionDigits: 0,
  maximumFractionDigits: 3,

  // Significant digits (alternative to fraction digits)
  minimumSignificantDigits: 1,
  maximumSignificantDigits: 3,

  // Integer formatting
  minimumIntegerDigits: 1,

  // Grouping (thousands separator)
  useGrouping: true | false,

  // Sign display
  signDisplay: 'auto' | 'never' | 'always' | 'exceptZero'
};

// Examples
const amount = 1234.56;
const number = 0.1234;

// Decimal number
new Intl.NumberFormat('en-US', { style: 'decimal' }).format(amount);
// "1,234.56"

// Percentage
new Intl.NumberFormat('en-US', { style: 'percent' }).format(number);
// "12%"

// Percentage with decimals
new Intl.NumberFormat('de-DE', {
  style: 'percent',
  minimumFractionDigits: 2
}).format(0.12345);
// "12,35%" (rounded)

// Currency (covered in detail below)
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(amount);
// "$1,234.56"

// Scientific notation
new Intl.NumberFormat('en-US', {
  notation: 'scientific'
}).format(1234.56);
// "1.235E3"

// Compact notation (for display)
new Intl.NumberFormat('en-US', {
  notation: 'compact',
  compactDisplay: 'short'
}).format(1000000);
// "1M"

new Intl.NumberFormat('en-US', {
  notation: 'compact',
  compactDisplay: 'long'
}).format(1000000);
// "1 million"

// Unit formatting
new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilometer-per-hour',
  unitDisplay: 'short'
}).format(100);
// "100 km/h"

// Sign display
new Intl.NumberFormat('en-US', {
  signDisplay: 'always'
}).format(5);
// "+5"

// Accounting format for negative numbers
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencySign: 'accounting'
}).format(-1234.56);
// "($1,234.56)"
```

---

## Currency Formatting

### Basic Currency Formatting

```javascript
// Format numbers as currency
const amount = 1234.56;

// US Dollar
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(amount);
// "$1,234.56"

// Euro (Germany)
new Intl.NumberFormat('de-DE', {
  style: 'currency',
  currency: 'EUR'
}).format(amount);
// "1.234,56 ¬"

// Euro (France)
new Intl.NumberFormat('fr-FR', {
  style: 'currency',
  currency: 'EUR'
}).format(amount);
// "1 234,56 ¬"

// British Pound
new Intl.NumberFormat('en-GB', {
  style: 'currency',
  currency: 'GBP'
}).format(amount);
// "£1,234.56"

// Japanese Yen (no decimal places)
new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(1234.56);
// "¥1,235" (rounded to nearest yen)

// Indian Rupee
new Intl.NumberFormat('en-IN', {
  style: 'currency',
  currency: 'INR'
}).format(amount);
// "¹1,234.56"

// Saudi Riyal
new Intl.NumberFormat('ar-SA', {
  style: 'currency',
  currency: 'SAR'
}).format(amount);
// "ü 1,234.56"
```

### Converting Between Currencies

```javascript
// Note: Intl.NumberFormat does NOT convert currencies
// It only formats numbers with the appropriate currency symbol and locale rules

// To convert: multiply by exchange rate, then format
async function formatCurrency(amount, fromCurrency, toLocale, toCurrency) {
  // Get exchange rate from API
  const rate = await getExchangeRate(fromCurrency, toCurrency);
  const convertedAmount = amount * rate;

  return new Intl.NumberFormat(toLocale, {
    style: 'currency',
    currency: toCurrency
  }).format(convertedAmount);
}

// Example: Convert $100 USD to EUR and format for Germany
const eur = await formatCurrency(100, 'USD', 'de-DE', 'EUR');
// "92,50 ¬" (assuming 1 USD = 0.925 EUR)
```

### Currency Display Options

```javascript
// Control how currency symbol is displayed
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'symbol' // "$"
}).format(1234.56);
// "$1,234.56"

new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'code' // "USD"
}).format(1234.56);
// "USD 1,234.56"

new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'name' // "US dollars"
}).format(1234.56);
// "1,234.56 US dollars"

// Narrow variant (if available)
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'EUR',
  currencyDisplay: 'narrowSymbol'
}).format(1234.56);
// "¬1,234.56" (instead of "EUR 1,234.56")
```

---

## Intl.RelativeTimeFormat

### Formatting Relative Times

```javascript
// Format relative times like "2 days ago" or "in 3 hours"
// Requires: relative numeric value and unit

// Create formatter
const formatter = new Intl.RelativeTimeFormat('en-US');
const formatterDE = new Intl.RelativeTimeFormat('de-DE');

// Format relative times
formatter.format(-1, 'day');     // "1 day ago"
formatter.format(2, 'hour');     // "in 2 hours"
formatter.format(-30, 'minute'); // "30 minutes ago"

formatterDE.format(-1, 'day');   // "vor 1 Tag"
formatterDE.format(2, 'hour');   // "in 2 Stunden"
```

### Relative Time Options

```javascript
// Options for controlling format style
const options = {
  // Style of the output
  style: 'long' | 'short' | 'narrow',

  // Numeric formatting
  numeric: 'always' | 'auto' // 'auto' uses "tomorrow" instead of "in 1 day"
};

// Long style (default)
new Intl.RelativeTimeFormat('en-US', { style: 'long' }).format(-1, 'day');
// "1 day ago"

// Short style
new Intl.RelativeTimeFormat('en-US', { style: 'short' }).format(-1, 'day');
// "1 day ago"

// Narrow style
new Intl.RelativeTimeFormat('en-US', { style: 'narrow' }).format(-1, 'day');
// "1 day ago"

// With numeric: 'auto'
new Intl.RelativeTimeFormat('en-US', { numeric: 'auto' }).format(0, 'day');
// "today" (instead of "in 0 days")

new Intl.RelativeTimeFormat('en-US', { numeric: 'auto' }).format(1, 'day');
// "tomorrow" (instead of "in 1 day")

new Intl.RelativeTimeFormat('en-US', { numeric: 'auto' }).format(-1, 'day');
// "yesterday" (instead of "1 day ago")

// German example
new Intl.RelativeTimeFormat('de-DE', { style: 'long', numeric: 'auto' }).format(-1, 'day');
// "gestern"
```

### Calculating and Formatting Relative Times

```javascript
// Helper function to calculate relative time
function formatTimeAgo(date, locale = 'en-US') {
  const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
  const now = new Date();
  const diffMs = date.getTime() - now.getTime();
  const diffSeconds = Math.round(diffMs / 1000);

  // Calculate largest applicable unit
  if (Math.abs(diffSeconds) < 60) {
    return rtf.format(diffSeconds, 'second');
  }
  if (Math.abs(diffSeconds) < 3600) {
    return rtf.format(Math.round(diffSeconds / 60), 'minute');
  }
  if (Math.abs(diffSeconds) < 86400) {
    return rtf.format(Math.round(diffSeconds / 3600), 'hour');
  }
  if (Math.abs(diffSeconds) < 604800) {
    return rtf.format(Math.round(diffSeconds / 86400), 'day');
  }
  if (Math.abs(diffSeconds) < 2592000) {
    return rtf.format(Math.round(diffSeconds / 604800), 'week');
  }
  if (Math.abs(diffSeconds) < 31536000) {
    return rtf.format(Math.round(diffSeconds / 2592000), 'month');
  }
  return rtf.format(Math.round(diffSeconds / 31536000), 'year');
}

// Usage
const pastDate = new Date(Date.now() - 5 * 60 * 1000); // 5 minutes ago
formatTimeAgo(pastDate, 'en-US');  // "5 minutes ago"
formatTimeAgo(pastDate, 'de-DE');  // "vor 5 Minuten"
```

### React Hook for Relative Time

```javascript
import { useState, useEffect } from 'react';

function useRelativeTime(date, locale = 'en-US', updateInterval = 60000) {
  const [relativeTime, setRelativeTime] = useState('');

  useEffect(() => {
    const update = () => {
      setRelativeTime(formatTimeAgo(date, locale));
    };

    update(); // Set initial value
    const interval = setInterval(update, updateInterval);

    return () => clearInterval(interval);
  }, [date, locale, updateInterval]);

  return relativeTime;
}

// Usage in component
function PostTime({ timestamp }) {
  const relativeTime = useRelativeTime(new Date(timestamp));
  return <time>{relativeTime}</time>;
}
```

---

## Intl.ListFormat

### Formatting Lists

```javascript
// Format lists with proper conjunctions and separators

// Create formatter
const formatter = new Intl.ListFormat('en-US');
const formatterDE = new Intl.ListFormat('de-DE');

// Format lists
formatter.format(['Alice', 'Bob']);  // "Alice and Bob"
formatter.format(['Alice', 'Bob', 'Charlie']);  // "Alice, Bob, and Charlie"

formatterDE.format(['Alice', 'Bob']);  // "Alice und Bob"
formatterDE.format(['Alice', 'Bob', 'Charlie']);  // "Alice, Bob und Charlie"
```

### List Format Options

```javascript
// Control how lists are formatted
const options = {
  // Type of list
  type: 'conjunction' | 'disjunction' | 'unit',

  // Style
  style: 'long' | 'short' | 'narrow'
};

// Conjunction (default, for "and")
new Intl.ListFormat('en-US', { type: 'conjunction' }).format(['tea', 'coffee', 'milk']);
// "tea, coffee, and milk"

// Disjunction (for "or")
new Intl.ListFormat('en-US', { type: 'disjunction' }).format(['tea', 'coffee']);
// "tea or coffee"

// Unit (for measurements)
new Intl.ListFormat('en-US', { type: 'unit', style: 'narrow' }).format(['5 feet', '8 inches']);
// "5 feet, 8 inches"

// Short style
new Intl.ListFormat('en-US', { style: 'short' }).format(['Alice', 'Bob', 'Charlie']);
// "Alice, Bob, Charlie" (no "and")

// Narrow style
new Intl.ListFormat('en-US', { style: 'narrow' }).format(['Alice', 'Bob', 'Charlie']);
// "Alice Bob Charlie" (no punctuation)

// German disjunction
new Intl.ListFormat('de-DE', { type: 'disjunction' }).format(['Tee', 'Kaffee', 'Milch']);
// "Tee, Kaffee oder Milch"
```

---

## Time Zone Handling

### Working with Time Zones

```javascript
// IMPORTANT: Time zones are tricky with Intl API

// Get current date in specific time zone
function getDateInTimeZone(date, timeZone) {
  const formatter = new Intl.DateTimeFormat('en-US', {
    timeZone,
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false
  });

  const parts = formatter.formatToParts(date);
  const obj = {};

  for (const { type, value } of parts) {
    obj[type] = value;
  }

  return new Date(
    `${obj.year}-${obj.month}-${obj.day}T${obj.hour}:${obj.minute}:${obj.second}`
  );
}

// Example
const utcDate = new Date('2025-12-18T12:00:00Z');
getDateInTimeZone(utcDate, 'America/New_York'); // Local time in NY
// 2025-12-18T07:00:00 (12:00 UTC = 07:00 EST)
```

### Common IANA Time Zones

```javascript
// Common time zones (IANA format)
const timeZones = {
  // US
  'America/New_York': 'Eastern Time',
  'America/Chicago': 'Central Time',
  'America/Denver': 'Mountain Time',
  'America/Los_Angeles': 'Pacific Time',

  // Europe
  'Europe/London': 'London',
  'Europe/Paris': 'Central European Time',
  'Europe/Berlin': 'Berlin/Central European Time',
  'Europe/Moscow': 'Moscow Standard Time',

  // Asia
  'Asia/Tokyo': 'Japan Standard Time',
  'Asia/Shanghai': 'China Standard Time',
  'Asia/Singapore': 'Singapore Standard Time',
  'Asia/Kolkata': 'India Standard Time',

  // Australia
  'Australia/Sydney': 'Sydney',
  'Australia/Melbourne': 'Melbourne',

  // UTC
  'UTC': 'Coordinated Universal Time'
};

// Format with time zone name
function formatWithTimeZone(date, timeZone, locale = 'en-US') {
  return new Intl.DateTimeFormat(locale, {
    timeZone,
    year: 'numeric',
    month: 'long',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    timeZoneName: 'long'
  }).format(date);
}

formatWithTimeZone(new Date(), 'America/New_York');
// "December 18, 2025, 7:30 AM Eastern Standard Time"
```

---

## Date Libraries with i18n

### date-fns with Locales

```javascript
// date-fns library with built-in locale support
import { format, parseISO } from 'date-fns';
import { enUS, de, ar } from 'date-fns/locale';

const date = new Date('2025-12-18');

// English
format(date, 'PPPP', { locale: enUS });  // "Thursday, December 18, 2025"

// German
format(date, 'PPPP', { locale: de });    // "Donnerstag, 18. Dezember 2025"

// Arabic
format(date, 'PPPP', { locale: ar });    // "'D.EJ3 ah /J3E(1 b`be"

// Different format patterns
format(date, 'yyyy-MM-dd');              // "2025-12-18"
format(date, 'dd.MM.yyyy', { locale: de }); // "18.12.2025"
format(date, 'MM/dd/yyyy', { locale: enUS }); // "12/18/2025"

// Relative time
import { formatDistance } from 'date-fns';
formatDistance(new Date('2025-12-10'), new Date('2025-12-18'), { addSuffix: true });
// "in 8 days"
```

### Day.js with i18n

```javascript
// Lightweight alternative to moment.js
import dayjs from 'dayjs';
import 'dayjs/locale/de';
import 'dayjs/locale/ar';
import relativeTime from 'dayjs/plugin/relativeTime';

dayjs.extend(relativeTime);

const date = dayjs('2025-12-18');

// English (default)
date.format('LLLL');  // "Thursday, December 18, 2025 12:00 AM"

// German
dayjs.locale('de');
date.format('LLLL');  // "Donnerstag, 18. Dezember 2025 00:00"

// Arabic
dayjs.locale('ar');
date.format('LLLL');  // "'D.EJ3 ah /J3E(1 b`be ab:``"

// Relative time
const past = dayjs('2025-12-10');
past.from(date);  // "8 days ago"
```

---

## Common Pitfalls

### 1. Forgetting to Handle Zero Decimals for Currencies

```javascript
// WRONG: Japanese Yen with 2 decimal places
new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY',
  minimumFractionDigits: 2
}).format(1234.56);
// "¥1,234.56" (looks wrong)

// RIGHT: Let Intl determine appropriate decimals
new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(1234.56);
// "¥1,235" (rounded to nearest yen)
```

### 2. Mixing UTC and Local Times

```javascript
// WRONG: Assuming local date operations work everywhere
const date = new Date(2025, 11, 18); // Local time
// Different in different time zones!

// RIGHT: Use ISO strings and explicit time zones
const isoDate = '2025-12-18T12:00:00Z'; // Explicit UTC
const date = new Date(isoDate);

// Format with specific time zone
new Intl.DateTimeFormat('en-US', {
  timeZone: 'America/New_York'
}).format(date);
```

### 3. Not Memoizing Formatters

```javascript
// WRONG: Creating formatters on every render
function DateComponent({ date }) {
  const formatted = new Intl.DateTimeFormat('en-US').format(date);
  return <p>{formatted}</p>;
}

// RIGHT: Memoize the formatter
function DateComponent({ date }) {
  const formatter = useMemo(
    () => new Intl.DateTimeFormat('en-US'),
    []
  );
  return <p>{formatter.format(date)}</p>;
}
```

### 4. Ignoring Locale Fallbacks

```javascript
// WRONG: Assuming all locales are supported
try {
  new Intl.DateTimeFormat('xyz-ABC').format(new Date());
} catch (e) {
  console.log('Unsupported locale');
}

// RIGHT: Have fallback logic
function safeFormat(date, locale = 'en-US') {
  try {
    const resolved = new Intl.DateTimeFormat(locale).resolvedOptions();
    return new Intl.DateTimeFormat(locale).format(date);
  } catch (e) {
    return new Intl.DateTimeFormat('en-US').format(date);
  }
}
```

### 5. Not Handling Missing Data for Relative Time

```javascript
// WRONG: Trying to format without calculating difference
formatTimeAgo(null); // Error!

// RIGHT: Add validation
function formatTimeAgo(date, locale = 'en-US') {
  if (!date || !(date instanceof Date)) {
    return 'Invalid date';
  }
  // ... rest of logic
}
```

---

## Interview Questions

### 1. What's the difference between Intl.DateTimeFormat and date-fns?

**Answer:**
- **Intl.DateTimeFormat**: Native browser API, no dependencies, follows Unicode standards
- **date-fns**: Library with more features, better date math, easier formatting patterns

Use Intl for simple locale-aware formatting; use date-fns for complex date manipulation.

### 2. How do you format a number as currency in multiple currencies?

**Answer:**
```javascript
function formatCurrency(amount, locale, currency) {
  return new Intl.NumberFormat(locale, {
    style: 'currency',
    currency
  }).format(amount);
}

formatCurrency(1234.56, 'en-US', 'USD'); // "$1,234.56"
formatCurrency(1234.56, 'de-DE', 'EUR'); // "1.234,56 ¬"
```

### 3. Why is time zone handling tricky in JavaScript?

**Answer:**
JavaScript's Date object stores times in UTC internally, but Intl.DateTimeFormat displays in local time zone. You must explicitly specify the time zone if you want a specific region, otherwise you get the browser's local time zone.

### 4. How do you calculate "time ago" correctly with i18n?

**Answer:**
Use `Intl.RelativeTimeFormat` with calculated time differences:
```javascript
const diff = (date - now) / 1000; // seconds
const rtf = new Intl.RelativeTimeFormat(locale, { numeric: 'auto' });
return rtf.format(Math.round(diff / 60), 'minute');
```

### 5. What's a performance consideration when using Intl formatters?

**Answer:**
Create formatters once and reuse them. Creating a new Intl.DateTimeFormat for every render is wasteful. Use React's useMemo hook or module-level constants.

### 6. How do you handle currency conversion with Intl?

**Answer:**
Intl.NumberFormat does NOT convert currencies. You must:
1. Get exchange rate from an API
2. Multiply amount by rate
3. Format the result with Intl.NumberFormat

### 7. What are the decimal place conventions for different currencies?

**Answer:**
- **Most currencies**: 2 decimal places (USD, EUR, GBP)
- **JPY, KRW**: 0 decimal places
- **Some currencies**: 3 decimal places (KWD, TND)

Intl.NumberFormat handles this automatically if you don't override minimumFractionDigits.

### 8. How do you format a list of items with locale-aware conjunctions?

**Answer:**
Use `Intl.ListFormat`:
```javascript
new Intl.ListFormat('en-US').format(['tea', 'coffee']);
// "tea and coffee"
```

### 9. How do you display a relative time that updates as time passes?

**Answer:**
Use a React hook that updates the display at intervals:
```javascript
function useRelativeTime(date) {
  const [time, setTime] = useState('');
  useEffect(() => {
    const interval = setInterval(
      () => setTime(formatTimeAgo(date)),
      60000 // Update every minute
    );
    return () => clearInterval(interval);
  }, [date]);
  return time;
}
```

### 10. What's the best library for comprehensive date/time handling with i18n?

**Answer:**
- **Simple needs**: Intl API (native, no dependencies)
- **Complex date math**: date-fns (modern, tree-shakeable, locale support)
- **Legacy projects**: moment.js (comprehensive but heavy)
- **Lightweight**: Day.js (smaller bundle)

---

## Navigation

**Continue Reading:**
- [04 - RTL Support](./04-rtl-support.md) - Support right-to-left languages

**Related Topics:**
- [01 - i18n Fundamentals](./01-i18n-fundamentals.md) - Core i18n concepts
- [02 - Pluralization](./02-pluralization.md) - Handle plural forms
- Frontend/React - Component patterns
- Frontend/JavaScript - Date and time handling

---

**Last Updated:** December 2025
**Difficulty:** Intermediate
**Estimated Time:** 3-4 hours
