# Design Systems

A design system is a comprehensive set of standards, documentation, reusable components, and design tokens that ensures consistency across digital products. Design systems bridge the gap between design and development, providing a single source of truth for building user interfaces.

## Table of Contents
- [Design System Fundamentals](#design-system-fundamentals)
- [Core Components](#core-components)
- [Building a Design System](#building-a-design-system)
- [Design Tokens](#design-tokens)
- [Popular Design Systems](#popular-design-systems)
- [Maintaining Design Systems](#maintaining-design-systems)
- [Governance and Contribution](#governance-and-contribution)
- [Implementation Examples](#implementation-examples)
- [Accessibility Considerations](#accessibility-considerations)
- [Interview Questions](#interview-questions)

## Design System Fundamentals

### What is a Design System?

Design systems are comprehensive collections of reusable components, guidelines, and tokens that ensure consistency across products. They function as living documentation that evolves with organizational needs, serving as the single source of truth for both designers and developers.

A design system is a living product that consists of:

```javascript
// Core elements of a design system
const designSystem = {
  // 1. Design Tokens
  tokens: {
    colors: { primary: '#3b82f6', danger: '#ef4444' },
    spacing: { xs: '0.25rem', sm: '0.5rem', md: '1rem' },
    typography: { fontFamily: 'Inter', fontSize: ['sm', 'base', 'lg'] },
    shadows: { sm: '0 1px 2px', md: '0 4px 6px' },
  },

  // 2. Component Library
  components: {
    Button: { variations: ['primary', 'secondary', 'danger'] },
    Card: { structure: ['header', 'body', 'footer'] },
    Input: { states: ['default', 'focused', 'error', 'disabled'] },
    Modal: { features: ['draggable', 'closeable', 'responsive'] },
  },

  // 3. Documentation
  documentation: {
    guidelines: 'Brand voice, usage patterns',
    patterns: 'Common UI patterns, best practices',
    accessibility: 'WCAG compliance, inclusive design',
    examples: 'Real-world usage examples',
  },

  // 4. Tools and Infrastructure
  tools: {
    storybook: 'Interactive component library',
    figma: 'Visual design tool',
    codegeneration: 'Automated code from design',
  },
};
```

### Why Design Systems Matter

Organizations adopt design systems to solve scaling challenges around consistency, efficiency, and cross-team collaboration. The investment in a design system pays dividends through accelerated development, reduced technical debt, and improved user experience quality.

```
1. CONSISTENCY
   - Same experience across all products
   - Unified brand identity
   - Predictable component behavior

2. EFFICIENCY
   - Faster feature development
   - No reinventing the wheel
   - Reduced design-to-code time

3. SCALABILITY
   - Support multiple teams
   - Enable rapid product growth
   - Maintain quality at scale

4. MAINTAINABILITY
   - Centralized updates
   - Easier bug fixes
   - Version management

5. ACCESSIBILITY
   - WCAG compliance built-in
   - Keyboard navigation
   - Screen reader support
```

## Core Components

### 1. Design Tokens

Design tokens abstract design decisions into platform-agnostic variables representing colors, spacing, typography, and other visual properties. This abstraction enables cross-platform consistency and allows updating the entire design system by changing token definitions centrally.

Design tokens are the smallest, most granular decisions of a design system.

```javascript
// Design tokens hierarchy
const tokens = {
  // Primitive tokens (no semantic meaning)
  primitives: {
    colors: {
      blue: {
        50: '#eff6ff',
        100: '#dbeafe',
        500: '#3b82f6',
        900: '#1e3a8a',
      },
    },
    spacing: {
      4: '1rem',
      8: '2rem',
      16: '4rem',
    },
  },

  // Semantic tokens (context-aware, with meaning)
  semantic: {
    colors: {
      primary: '{primitives.colors.blue.500}',
      primaryHover: '{primitives.colors.blue.600}',
      danger: '{primitives.colors.red.500}',
      dangerHover: '{primitives.colors.red.600}',
    },
    spacing: {
      small: '{primitives.spacing.4}',
      medium: '{primitives.spacing.8}',
      large: '{primitives.spacing.16}',
    },
  },

  // Component tokens (component-specific)
  component: {
    button: {
      padding: '{semantic.spacing.medium}',
      backgroundColor: '{semantic.colors.primary}',
      backgroundColorHover: '{semantic.colors.primaryHover}',
    },
  },
};
```

### 2. Component Library

Component libraries provide pre-built, reusable UI elements that implement design system guidelines. Each component encapsulates structure, behavior, and styling while exposing props for customization, ensuring consistent implementation across applications.

Components are the building blocks of user interfaces.

```javascript
// Component anatomy
const Button = {
  // Structure
  structure: {
    element: 'button',
    children: 'text content',
    attributes: ['disabled', 'type', 'aria-label'],
  },

  // Variants (visual options)
  variants: {
    type: ['primary', 'secondary', 'danger'],
    size: ['small', 'medium', 'large'],
    state: ['default', 'hover', 'focus', 'disabled'],
  },

  // Props (API)
  props: {
    onClick: 'Function',
    disabled: 'Boolean',
    loading: 'Boolean',
    icon: 'ReactNode',
    children: 'ReactNode',
  },

  // Examples
  examples: [
    '<Button variant="primary">Click me</Button>',
    '<Button variant="primary" disabled>Disabled</Button>',
    '<Button variant="primary" loading>Loading...</Button>',
  ],
};
```

### 3. Documentation

Comprehensive documentation ensures proper usage and understanding.

```markdown
# Button Component Documentation

## Overview
The Button component is used to trigger actions or navigate to new pages.

## Usage
```jsx
import { Button } from '@company/design-system';

<Button variant="primary" onClick={() => handleClick()}>
  Click me
</Button>
```

## Props
| Prop | Type | Default | Description |
|------|------|---------|-------------|
| variant | 'primary' | 'secondary' | 'danger' | 'primary' | Visual style |
| size | 'small' | 'medium' | 'large' | 'medium' | Button size |
| disabled | boolean | false | Disable interaction |
| loading | boolean | false | Show loading state |
| onClick | function | - | Click handler |

## Accessibility
- Keyboard accessible (Enter/Space to activate)
- Screen reader friendly (aria-label support)
- Focus management
- Proper button semantics

## Examples
- Primary action: Save, Submit
- Secondary action: Cancel, Back
- Dangerous action: Delete, Remove
```

### 4. Guidelines and Patterns

```markdown
# Design Guidelines

## Spacing
Use the 8px spacing scale:
- 4px: Tight spacing (rarely used)
- 8px: Minimum spacing between elements
- 16px: Standard spacing
- 24px: Large spacing
- 32px: Extra large spacing

## Typography
- Headlines: Bold, 2rem
- Body text: Regular, 1rem
- Small text: Regular, 0.875rem
- Line height: 1.5 (body), 1.2 (headings)

## Color Usage
- Primary: Main CTA buttons, links
- Secondary: Alternative actions
- Danger: Destructive actions
- Info: Informational content
- Success: Success confirmations
- Warning: Warning messages

## Motion
- Transitions: 200ms for hover states
- Animations: 400ms for larger changes
- Avoid: Flashing, auto-playing media
- Respect: prefers-reduced-motion
```

## Building a Design System

### Phase 1: Foundation

```javascript
// Step 1: Define design tokens
// tokens/colors.json
{
  "color": {
    "primary": { "value": "#3b82f6" },
    "secondary": { "value": "#8b5cf6" },
    "gray": {
      "50": { "value": "#f9fafb" },
      "500": { "value": "#6b7280" },
      "900": { "value": "#111827" }
    }
  }
}

// Step 2: Create primitive components
// Button.tsx, Card.tsx, Input.tsx, etc.
export const Button = ({ variant, children, ...props }) => {
  const baseStyles = 'px-4 py-2 rounded font-semibold';
  const variantStyles = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600',
    secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
  };

  return (
    <button className={`${baseStyles} ${variantStyles[variant]}`} {...props}>
      {children}
    </button>
  );
};

// Step 3: Create composite components
// Card with Button composition
export const Card = ({ title, actions, children }) => {
  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      <h2 className="text-xl font-bold">{title}</h2>
      <div>{children}</div>
      <div className="flex gap-2">
        {actions.map(action => (
          <Button key={action.id} variant={action.variant}>
            {action.label}
          </Button>
        ))}
      </div>
    </div>
  );
};
```

### Phase 2: Organization and Documentation

```bash
# Project structure
design-system/
 src/
    tokens/
       colors.ts
       spacing.ts
       typography.ts
       index.ts
    components/
       Button/
          Button.tsx
          Button.test.tsx
          Button.stories.tsx
          Button.module.css
       Card/
       Input/
       index.ts
    hooks/
       useMediaQuery.ts
       useFocus.ts
       index.ts
    index.ts
 .storybook/
    main.ts
    preview.ts
    manager.ts
 package.json
 README.md
```

### Phase 3: Storybook Setup

```javascript
// .storybook/main.ts
export default {
  stories: ['../src/**/*.stories.tsx'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',
    '@storybook/addon-interactions',
  ],
  framework: '@storybook/react',
};

// .storybook/preview.ts
export const decorators = [
  (story) => (
    <div style={{ padding: '2rem' }}>
      {story()}
    </div>
  ),
];

export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
  controls: {
    matchers: {
      color: /(background|color)$/i,
      date: /Date$/,
    },
  },
};
```

### Phase 4: Component Documentation

```javascript
// Button.stories.tsx
import { Button } from './Button';
import type { Meta, StoryObj } from '@storybook/react';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
  },
  tags: ['autodocs'],
};

export default meta;
type Story = StoryObj<typeof meta>;

// Basic variants
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

export const Secondary: Story = {
  args: {
    variant: 'secondary',
    children: 'Secondary Button',
  },
};

// States
export const Disabled: Story = {
  args: {
    variant: 'primary',
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Loading: Story = {
  args: {
    variant: 'primary',
    loading: true,
    children: 'Loading...',
  },
};

// Sizes
export const Small: Story = {
  args: {
    variant: 'primary',
    size: 'small',
    children: 'Small Button',
  },
};

export const Large: Story = {
  args: {
    variant: 'primary',
    size: 'large',
    children: 'Large Button',
  },
};

// Accessibility
export const AccessibleButton: Story = {
  args: {
    variant: 'primary',
    'aria-label': 'Submit form',
    children: 'Submit',
  },
};
```

## Design Tokens

### Token Categories

```javascript
// Tokens structured by category
export const tokens = {
  // Color tokens
  colors: {
    // Primitive colors
    gray: {
      50: '#f9fafb',
      100: '#f3f4f6',
      500: '#6b7280',
      900: '#111827',
    },
    // Semantic colors
    primary: '#3b82f6',
    secondary: '#8b5cf6',
    success: '#22c55e',
    error: '#ef4444',
    warning: '#f59e0b',
    info: '#0ea5e9',
  },

  // Spacing tokens (8px scale)
  spacing: {
    0: '0',
    1: '0.25rem',   // 4px
    2: '0.5rem',    // 8px
    3: '0.75rem',   // 12px
    4: '1rem',      // 16px
    6: '1.5rem',    // 24px
    8: '2rem',      // 32px
    12: '3rem',     // 48px
    16: '4rem',     // 64px
  },

  // Typography tokens
  typography: {
    fontFamily: {
      sans: 'Inter, system-ui, sans-serif',
      serif: 'Merriweather, serif',
      mono: 'Menlo, monospace',
    },
    fontSize: {
      xs: '0.75rem',    // 12px
      sm: '0.875rem',   // 14px
      base: '1rem',     // 16px
      lg: '1.125rem',   // 18px
      xl: '1.25rem',    // 20px
      '2xl': '1.5rem',  // 24px
      '3xl': '1.875rem', // 30px
      '4xl': '2.25rem',  // 36px
    },
    fontWeight: {
      light: 300,
      normal: 400,
      medium: 500,
      semibold: 600,
      bold: 700,
    },
    lineHeight: {
      tight: 1.2,
      normal: 1.5,
      relaxed: 1.75,
    },
  },

  // Shadow tokens
  shadows: {
    none: 'none',
    sm: '0 1px 2px 0 rgba(0, 0, 0, 0.05)',
    md: '0 4px 6px -1px rgba(0, 0, 0, 0.1)',
    lg: '0 10px 15px -3px rgba(0, 0, 0, 0.1)',
    xl: '0 20px 25px -5px rgba(0, 0, 0, 0.1)',
  },

  // Border radius tokens
  borderRadius: {
    none: '0',
    sm: '0.25rem',
    md: '0.375rem',
    lg: '0.5rem',
    xl: '0.75rem',
    full: '9999px',
  },

  // Animation tokens
  animation: {
    duration: {
      fast: '100ms',
      base: '200ms',
      slow: '300ms',
    },
    easing: {
      easeInOut: 'cubic-bezier(0.4, 0, 0.2, 1)',
      easeOut: 'cubic-bezier(0, 0, 0.2, 1)',
      easeIn: 'cubic-bezier(0.4, 0, 1, 1)',
    },
  },
};
```

### Style Dictionary Integration

```javascript
// config.json
{
  "source": [
    "tokens/**/*.json"
  ],
  "platforms": {
    "web": {
      "transformGroup": "web",
      "buildPath": "src/generated/",
      "files": [
        {
          "destination": "tokens.css",
          "format": "css/variables",
          "options": {
            "outputReferences": true
          }
        },
        {
          "destination": "tokens.ts",
          "format": "typescript/module-declarations"
        }
      ]
    },
    "ios": {
      "transformGroup": "ios",
      "buildPath": "ios/DesignTokens/",
      "files": [
        {
          "destination": "Colors.swift",
          "format": "ios/colors"
        },
        {
          "destination": "Spacing.swift",
          "format": "ios/spacing"
        }
      ]
    }
  }
}

// tokens/colors.json
{
  "color": {
    "primary": {
      "value": "#3b82f6",
      "description": "Primary brand color"
    },
    "gray": {
      "50": { "value": "#f9fafb" },
      "500": { "value": "#6b7280" }
    }
  }
}

// Generate with: style-dictionary build
```

## Popular Design Systems

### Material-UI (MUI)

```javascript
// Material Design implementation for React
import { createTheme, ThemeProvider, Button, Card } from '@mui/material';

const theme = createTheme({
  palette: {
    primary: {
      main: '#1976d2',
    },
    secondary: {
      main: '#dc004e',
    },
  },
  typography: {
    fontFamily: 'Roboto',
    h1: {
      fontSize: '2.5rem',
      fontWeight: 700,
    },
  },
});

export default function App() {
  return (
    <ThemeProvider theme={theme}>
      <Button variant="contained" color="primary">
        Click me
      </Button>
      <Card>Content</Card>
    </ThemeProvider>
  );
}
```

**Characteristics:**
- Material Design principles
- Comprehensive component library (50+ components)
- Built-in theming system
- Excellent documentation
- Large community
- TypeScript support
- CSS-in-JS (Emotion)

### Ant Design

```javascript
// Enterprise UI library
import { Button, Card, Layout, theme } from 'antd';

const { Header, Sider, Content } = Layout;
const { useToken } = theme;

export default function App() {
  const { token } = useToken();

  return (
    <Layout>
      <Sider>Sidebar</Sider>
      <Layout>
        <Header>Header</Header>
        <Content>
          <Card title="Card Title">
            <Button type="primary">Submit</Button>
          </Card>
        </Content>
      </Layout>
    </Layout>
  );
}
```

**Characteristics:**
- Enterprise-focused
- Rich component library (60+ components)
- Beautiful default design
- Excellent for admin panels/dashboards
- Chinese company (internationalization support)
- Theme customization
- CSS-in-JS

### Chakra UI

```javascript
// Accessible component library built on styled-system
import { Button, Box, VStack, ChakraProvider, extendTheme } from '@chakra-ui/react';

const theme = extendTheme({
  colors: {
    brand: {
      50: '#f5f7ff',
      500: '#3b82f6',
      900: '#1e3a8a',
    },
  },
  fonts: {
    heading: 'Georgia, serif',
    body: 'Open Sans, sans-serif',
  },
});

export default function App() {
  return (
    <ChakraProvider theme={theme}>
      <VStack spacing={4}>
        <Button colorScheme="brand" size="lg">
          Click me
        </Button>
      </VStack>
    </ChakraProvider>
  );
}
```

**Characteristics:**
- Accessibility-first (WCAG 2.1 Level AA)
- Simple, intuitive API
- Styled-system based
- Great keyboard navigation
- Good documentation
- Smaller bundle size
- React Hooks-based

### Radix UI

```javascript
// Unstyled, accessible components
import * as Dialog from '@radix-ui/react-dialog';
import * as Button from '@radix-ui/react-primitive';

export default function MyDialog() {
  return (
    <Dialog.Root>
      <Dialog.Trigger asChild>
        <Button.default>Open Dialog</Button.default>
      </Dialog.Trigger>
      <Dialog.Portal>
        <Dialog.Overlay />
        <Dialog.Content>
          <Dialog.Title>Dialog Title</Dialog.Title>
          <Dialog.Description>
            Dialog description
          </Dialog.Description>
          <Dialog.Close asChild>
            <Button.default>Close</Button.default>
          </Dialog.Close>
        </Dialog.Content>
      </Dialog.Portal>
    </Dialog.Root>
  );
}
```

**Characteristics:**
- Unstyled, headless components
- Focus on accessibility and behavior
- Bring your own styles
- Composable primitives
- No design opinions
- Flexible and extensible
- Perfect for custom design systems

### Comparison

| System | Use Case | Bundle Size | Learning Curve | Customization | Accessibility |
|--------|----------|------------|-----------------|---------------|----------------|
| Material-UI | General, modern | Large | Medium | Good | Good |
| Ant Design | Enterprise, dashboard | Large | Medium | Medium | Good |
| Chakra UI | Accessible, simple | Small | Low | Excellent | Excellent |
| Radix UI | Custom, headless | Extra small | High | Excellent | Excellent |

## Maintaining Design Systems

### Versioning Strategy

```javascript
// Semantic versioning for design systems
// MAJOR.MINOR.PATCH

// MAJOR: Breaking changes
// 2.0.0 - Button color changed from blue to purple
// Old: <Button>Click</Button> still works
// But visual appearance different

// MINOR: New features, non-breaking changes
// 1.1.0 - Added new size variant "extra-large"
// Old code works, new capability available

// PATCH: Bug fixes, non-breaking changes
// 1.0.1 - Fixed Button hover state bug
// No API changes, just fixes

const package_json = {
  "name": "@company/design-system",
  "version": "2.3.1",
  "description": "Official design system",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "scripts": {
    "build": "tsc && rollup -c",
    "publish": "npm publish",
    "changelog": "auto changelog",
  },
};
```

### Breaking Changes Management

```javascript
// Track and communicate breaking changes
const CHANGELOG = `
## 2.0.0 - 2024-01-15

### Breaking Changes
- Button: Removed 'flat' variant (use 'secondary' instead)
- Button: Changed 'sm' size to 'small'
- Card: Removed card-title, use card-header instead

### Migration Guide
\`\`\`
// Old
<Button variant="flat">Click</Button>

// New
<Button variant="secondary">Click</Button>
\`\`\`

### New Features
- Added Button 'outline' variant
- Added new 'neutral' color token
- Added focus-visible support for keyboard nav

### Deprecations
- Button 'type' prop deprecated, use 'variant'
- Will be removed in v3.0.0
`;
```

### Governance Process

```javascript
// Design system governance
const governanceProcess = {
  // 1. Proposal phase
  proposal: {
    where: 'GitHub Issues',
    who: 'Any team member',
    description: 'Document the need for a new component/token',
    template: 'Issue template with motivation, use cases, examples',
  },

  // 2. Review phase
  review: {
    where: 'Design system team',
    who: 'Design lead + 2+ reviewers',
    criteria: [
      'Aligns with design principles',
      'Solves real problem',
      'Not duplicate existing component',
      'Accessible implementation',
      'Proper documentation',
    ],
  },

  // 3. Implementation phase
  implementation: {
    where: 'Design system repo',
    who: 'Assigned contributor',
    requirements: [
      'Component code',
      'Unit tests (90%+ coverage)',
      'Storybook stories',
      'Documentation',
      'Accessibility audit',
    ],
  },

  // 4. Approval and Release
  approval: {
    where: 'Pull request',
    who: 'CODEOWNERS review',
    requirements: [
      'All tests pass',
      'No console errors/warnings',
      'Documentation complete',
      'Accessibility verified',
    ],
  },

  // 5. Release
  release: {
    version: 'Semantic versioning',
    changelog: 'Auto-generated from commits',
    npm: 'Publish to @company/design-system',
  },
};
```

## Implementation Examples

### Building a Button Component

```typescript
// Button.tsx
import React from 'react';
import styles from './Button.module.css';

export interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
  onClick?: (e: React.MouseEvent<HTMLButtonElement>) => void;
  children: React.ReactNode;
  type?: 'button' | 'submit' | 'reset';
  fullWidth?: boolean;
  icon?: React.ReactNode;
  ariaLabel?: string;
}

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      variant = 'primary',
      size = 'medium',
      disabled = false,
      loading = false,
      onClick,
      children,
      type = 'button',
      fullWidth = false,
      icon,
      ariaLabel,
    },
    ref
  ) => {
    const className = [
      styles.button,
      styles[`button--${variant}`],
      styles[`button--${size}`],
      fullWidth && styles['button--full-width'],
      (disabled || loading) && styles['button--disabled'],
    ]
      .filter(Boolean)
      .join(' ');

    return (
      <button
        ref={ref}
        className={className}
        type={type}
        disabled={disabled || loading}
        onClick={onClick}
        aria-label={ariaLabel}
        aria-busy={loading}
      >
        {loading && <span className={styles.loader} />}
        {icon && <span className={styles.icon}>{icon}</span>}
        <span className={styles.text}>{children}</span>
      </button>
    );
  }
);

Button.displayName = 'Button';
```

```css
/* Button.module.css */
.button {
  display: inline-flex;
  align-items: center;
  gap: var(--spacing-2);
  font-family: var(--font-family-sans);
  font-weight: var(--font-weight-semibold);
  border: none;
  border-radius: var(--border-radius-md);
  cursor: pointer;
  transition: all var(--animation-duration-base) var(--animation-easing-easeOut);
  white-space: nowrap;
  user-select: none;
}

.button--primary {
  background-color: var(--color-primary);
  color: white;
}

.button--primary:hover:not(:disabled) {
  background-color: var(--color-primary-hover);
  box-shadow: var(--shadow-md);
  transform: translateY(-1px);
}

.button--primary:focus-visible {
  outline: 2px solid var(--color-primary-focus);
  outline-offset: 2px;
}

.button--secondary {
  background-color: var(--color-gray-200);
  color: var(--color-gray-900);
}

.button--danger {
  background-color: var(--color-error);
  color: white;
}

.button--small {
  padding: var(--spacing-1) var(--spacing-2);
  font-size: var(--font-size-sm);
}

.button--medium {
  padding: var(--spacing-2) var(--spacing-4);
  font-size: var(--font-size-base);
}

.button--large {
  padding: var(--spacing-3) var(--spacing-6);
  font-size: var(--font-size-lg);
}

.button--full-width {
  width: 100%;
}

.button--disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.loader {
  display: inline-block;
  width: 1em;
  height: 1em;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.icon {
  display: inline-flex;
  align-items: center;
}

.text {
  display: inline;
}

/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  .button {
    transition: none;
  }

  .button--primary:hover {
    transform: none;
  }

  .loader {
    animation: none;
    border: 2px solid currentColor;
    opacity: 0.6;
  }
}

/* Dark mode support */
@media (prefers-color-scheme: dark) {
  .button--secondary {
    background-color: var(--color-gray-700);
    color: var(--color-gray-100);
  }
}
```

### Storybook Documentation

```typescript
// Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';
import { StarIcon } from '@radix-ui/react-icons';

const meta: Meta<typeof Button> = {
  title: 'Components/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'A versatile button component for user interactions.',
      },
    },
  },
  tags: ['autodocs'],
  argTypes: {
    variant: {
      control: { type: 'radio' },
      options: ['primary', 'secondary', 'danger'],
      description: 'Visual style of the button',
    },
    size: {
      control: { type: 'radio' },
      options: ['small', 'medium', 'large'],
      description: 'Size variant',
    },
    disabled: {
      control: 'boolean',
      description: 'Disable user interaction',
    },
    loading: {
      control: 'boolean',
      description: 'Show loading state',
    },
    fullWidth: {
      control: 'boolean',
      description: 'Stretch to full width',
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

// Primary button
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Primary Button',
  },
};

// All variants
export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem' }}>
      <Button variant="primary">Primary</Button>
      <Button variant="secondary">Secondary</Button>
      <Button variant="danger">Danger</Button>
    </div>
  ),
};

// All sizes
export const AllSizes: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: '1rem', alignItems: 'center' }}>
      <Button size="small">Small</Button>
      <Button size="medium">Medium</Button>
      <Button size="large">Large</Button>
    </div>
  ),
};

// States
export const Disabled: Story = {
  args: {
    disabled: true,
    children: 'Disabled Button',
  },
};

export const Loading: Story = {
  args: {
    loading: true,
    children: 'Loading...',
  },
};

// With icon
export const WithIcon: Story = {
  args: {
    icon: <StarIcon />,
    children: 'Favorite',
  },
};

// Full width
export const FullWidth: Story = {
  args: {
    fullWidth: true,
    children: 'Full Width Button',
  },
  decorators: [
    (Story) => (
      <div style={{ width: '300px' }}>
        <Story />
      </div>
    ),
  ],
};

// Accessibility
export const Accessible: Story = {
  args: {
    ariaLabel: 'Submit form',
    children: 'Submit',
  },
};
```

## Accessibility Considerations

### WCAG 2.1 Compliance

```javascript
// Accessibility checklist for design system components

const accessibilityRequirements = {
  // Color and contrast
  contrast: {
    requirement: 'WCAG AA: 4.5:1 for normal text, 3:1 for large text',
    implementation: 'Use contrast checker tools during development',
    testing: 'Automated testing with axe-core',
  },

  // Keyboard navigation
  keyboard: {
    requirement: 'All functionality available via keyboard',
    implementation: 'Support Tab, Enter, Space, Escape keys',
    testing: 'Manual keyboard testing',
  },

  // Focus management
  focus: {
    requirement: 'Visible focus indicator',
    implementation: 'Focus outline min 2px',
    testing: 'Visual testing',
  },

  // Screen readers
  screenReader: {
    requirement: 'Proper ARIA labels and roles',
    implementation: 'aria-label, aria-describedby, role attributes',
    testing: 'Screen reader testing (NVDA, JAWS, VoiceOver)',
  },

  // Motion
  motion: {
    requirement: 'Respect prefers-reduced-motion',
    implementation: '@media (prefers-reduced-motion: reduce)',
    testing: 'Browser accessibility settings',
  },

  // Forms
  forms: {
    requirement: 'Proper labels, error messages, validation',
    implementation: 'label elements, aria-required, aria-invalid',
    testing: 'Form submission testing',
  },
};

// Implementation example
export const AccessibleInput = React.forwardRef<
  HTMLInputElement,
  {
    label: string;
    error?: string;
    required?: boolean;
    id: string;
  }
>(({ label, error, required, id }, ref) => (
  <div>
    <label htmlFor={id}>
      {label}
      {required && <span aria-label="required">*</span>}
    </label>
    <input
      ref={ref}
      id={id}
      type="text"
      required={required}
      aria-invalid={!!error}
      aria-describedby={error ? `${id}-error` : undefined}
    />
    {error && (
      <div id={`${id}-error`} role="alert">
        {error}
      </div>
    )}
  </div>
));
```

## Interview Questions

### Question 1: What is a design system and why do companies build them?

**Answer:**

A design system is a comprehensive set of standards, reusable components, design tokens, and documentation that ensures consistency across digital products. It's the single source of truth for design and development.

**Key Components:**
1. Design tokens (colors, spacing, typography, shadows)
2. Component library (buttons, cards, inputs, etc.)
3. Documentation and guidelines
4. Tools (Storybook, Figma, code generation)

**Why Companies Build Design Systems:**

```
1. CONSISTENCY
   - Unified user experience across products
   - Brand consistency
   - Predictable component behavior

2. EFFICIENCY
   - Faster development (reusable components)
   - Reduced design-to-code time
   - No reinventing the wheel

3. SCALABILITY
   - Support multiple teams
   - Maintain quality at scale
   - Easier to onboard new members

4. MAINTAINABILITY
   - Centralized updates
   - Single place to fix bugs
   - Version management

5. ACCESSIBILITY
   - WCAG compliance built-in
   - Consistent keyboard navigation
   - Screen reader support
```

**Real-world Examples:**
- Google Material Design
- Shopify Polaris
- Atlassian Design System
- IBM Carbon

---

### Question 2: Describe the core components of a design system.

**Answer:**

A mature design system has four core pillars:

**1. Design Tokens:**
- Smallest, most granular design decisions
- Structured hierarchy (primitive � semantic � component)
- Examples: colors, spacing, typography, shadows

```javascript
// Token hierarchy
const tokens = {
  primitives: { colors: { blue: { 50: '#eff6ff' } } },
  semantic: { colors: { primary: '{primitives.colors.blue.500}' } },
  component: { button: { backgroundColor: '{semantic.colors.primary}' } },
};
```

**2. Component Library:**
- Reusable UI building blocks
- Documented props and variants
- Accessibility built-in
- Examples: Button, Card, Input, Modal

**3. Documentation:**
- How to use each component
- Design guidelines and patterns
- Best practices
- Real-world examples

**4. Tools and Infrastructure:**
- Storybook (component library showcase)
- Figma (design tool integration)
- Code generation
- CI/CD integration

**Integration:**
These components work together. Design tokens define the styling palette, components consume tokens, documentation explains usage, and tools manage everything.

---

### Question 3: How do you implement design tokens across multiple platforms (web, iOS, Android)?

**Answer:**

Use a centralized token management system like Style Dictionary to generate platform-specific code from a single source.

**Approach:**

```
1. Define tokens in JSON
    tokens/colors.json, tokens/spacing.json, etc.

2. Create Style Dictionary config
    Specifies transforms and outputs

3. Generate platform-specific code
    CSS variables (web)
    Swift (iOS)
    Kotlin (Android)
    JavaScript modules

4. Version and distribute
    npm packages, version control
```

**Implementation:**

```javascript
// config.json
{
  "source": ["tokens/**/*.json"],
  "platforms": {
    "web": {
      "transformGroup": "web",
      "buildPath": "dist/web/",
      "files": [{
        "destination": "tokens.css",
        "format": "css/variables"
      }]
    },
    "ios": {
      "transformGroup": "ios",
      "buildPath": "dist/ios/",
      "files": [{
        "destination": "Colors.swift",
        "format": "ios/colors"
      }]
    }
  }
}

// Run: style-dictionary build
// Generates:
// - dist/web/tokens.css (CSS variables)
// - dist/ios/Colors.swift (Swift colors)
```

**Benefits:**
- Single source of truth
- Consistency across platforms
- Automated code generation
- Easy maintenance and updates

---

### Question 4: Compare popular design systems (Material-UI, Ant Design, Chakra UI, Radix UI).

**Answer:**

| Feature | Material-UI | Ant Design | Chakra UI | Radix UI |
|---------|-----------|-----------|----------|----------|
| **Design Philosophy** | Material Design | Enterprise | Accessibility-first | Headless |
| **Bundle Size** | Large (~50KB) | Large (~60KB) | Small (~30KB) | Extra small (~15KB) |
| **Components** | 50+ | 60+ | 40+ | Primitives |
| **Styled System** | Emotion/JSS | Tailwind/CSS-in-JS | Styled-system | Bring your own |
| **Best For** | Modern apps | Admin dashboards | Accessible UIs | Custom systems |
| **Learning Curve** | Medium | Medium | Low | High |
| **Customization** | Good | Medium | Excellent | Maximum |
| **Documentation** | Excellent | Good | Excellent | Good |
| **Accessibility** | Good (WCAG 2.1) | Good | Excellent (AA+) | Excellent |
| **Popularity** | Very high | High | High | Growing |

**When to Use Each:**

- **Material-UI:** Building Material Design UIs quickly, components are feature-rich
- **Ant Design:** Enterprise applications, especially admin panels
- **Chakra UI:** Simple, accessible components, focus on developer experience
- **Radix UI:** Building your own design system, maximum flexibility

---

### Question 5: How do you manage breaking changes in a design system?

**Answer:**

Managing breaking changes requires careful planning and clear communication.

**Strategy:**

1. **Semantic Versioning:**
   - MAJOR: Breaking changes
   - MINOR: New non-breaking features
   - PATCH: Bug fixes

2. **Deprecation Process:**
   ```javascript
   // Step 1: Mark as deprecated
   export const OldComponent = (props) => {
     console.warn(
       'OldComponent is deprecated in v2.0.0. ' +
       'Use NewComponent instead.'
     );
     return <NewComponent {...props} />;
   };

   // Step 2: Provide migration guide
   // Step 3: Remove in major version
   ```

3. **Communication:**
   - Announcement 2-3 versions early
   - Detailed migration guide
   - Code examples
   - Automated codemods for simple changes

4. **Versioning Timeline:**
   ```
   v1.0.0 - Original release
   v1.5.0 - Deprecate OldButton (console warning)
   v2.0.0 - Remove OldButton (breaking change)
   ```

5. **Help Users Migrate:**
   ```bash
   # Automated migration script
   npx @company/design-system@latest codemod Button
   ```

---

### Question 6: How do you ensure accessibility in a design system?

**Answer:**

Accessibility must be built into every component from the start.

**Implementation Steps:**

1. **Design Phase:**
   - Plan keyboard navigation
   - Ensure color contrast (4.5:1 minimum)
   - Design focus indicators

2. **Development:**
   ```javascript
   // Semantic HTML
   <button>Click me</button> // Not <div onClick>

   // ARIA labels
   <button aria-label="Close dialog">�</button>

   // Focus management
   :focus-visible { outline: 2px solid; }

   // Screen reader support
   <div role="alert">{error}</div>
   ```

3. **Testing:**
   - Automated: axe-core, jest-axe
   - Manual: Keyboard testing, screen readers
   - Tools: NVDA, JAWS, VoiceOver

4. **Guidelines:**
   - WCAG 2.1 Level AA minimum
   - Keyboard accessible
   - Visible focus indicators
   - Proper ARIA labels
   - Respect `prefers-reduced-motion`

---

### Question 7: How do you document design system components effectively?

**Answer:**

Good documentation is critical for design system adoption.

**Documentation Elements:**

1. **Component Overview:**
   - What the component does
   - When to use it
   - When NOT to use it

2. **Usage Examples:**
   ```jsx
   <Button variant="primary">Click me</Button>
   ```

3. **Props Table:**
   | Prop | Type | Default | Description |
   |------|------|---------|-------------|

4. **Accessibility:**
   - Keyboard support
   - ARIA labels
   - Screen reader testing

5. **Design Guidelines:**
   - Spacing rules
   - Color usage
   - Typography scales

6. **Interactive Examples (Storybook):**
   - Variants
   - States (hover, disabled, loading)
   - Edge cases
   - Responsive behavior

**Tools:**
- Storybook for interactive docs
- Notion/Confluence for written guides
- Figma for design specs
- Auto-generated API docs from TypeScript

---

### Question 8: What is the difference between a design system and a component library?

**Answer:**

**Component Library:**
- Just the code
- Reusable UI components
- Example: React component package

**Design System:**
- Holistic approach
- Components + design tokens + documentation + guidelines
- Single source of truth for design and code
- Includes processes and governance

```
Component Library          Design System
 Code                    Components
 TypeScript              Design Tokens
 Some docs               Documentation
                           Guidelines
                           Figma library
                           Storybook
                           Processes
                           Governance
```

**Analogy:**
- Component library = IKEA instructions for one chair
- Design system = IKEA entire product ecosystem with guidelines for everything

---

### Question 9: How do you handle theming (dark mode, multiple color schemes) in a design system?

**Answer:**

Theming requires flexible design tokens and component implementation.

**Approach 1: CSS Variables:**
```css
/* Light theme (default) */
:root {
  --color-bg: #ffffff;
  --color-text: #000000;
  --color-primary: #3b82f6;
}

/* Dark theme */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #1a1a1a;
    --color-text: #ffffff;
    --color-primary: #60a5fa;
  }
}

/* Explicit theme override */
[data-theme="dark"] {
  --color-bg: #1a1a1a;
  --color-text: #ffffff;
}
```

**Approach 2: Design Tokens:**
```javascript
const light = {
  colors: {
    bg: '#ffffff',
    text: '#000000',
    primary: '#3b82f6',
  },
};

const dark = {
  colors: {
    bg: '#1a1a1a',
    text: '#ffffff',
    primary: '#60a5fa',
  },
};
```

**Approach 3: Theme Provider (React):**
```jsx
const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <div data-theme={theme}>
        {children}
      </div>
    </ThemeContext.Provider>
  );
}
```

**Key Considerations:**
- Accessibility (sufficient contrast in all themes)
- Performance (avoid flash of unstyled theme)
- User preference (respect system preference)
- Manual override option

---

### Question 10: What metrics do you use to measure the success of a design system?

**Answer:**

Measuring design system impact requires quantitative and qualitative metrics.

**Developer Metrics:**
```
1. Adoption rate
   - % of components built using system
   - # of projects using system

2. Development velocity
   - Time to build new feature (before vs after)
   - Component reuse ratio
   - Faster time-to-market

3. Code quality
   - # of accessibility violations
   - CSS file size reduction
   - Bundle size reduction

4. Maintenance
   - # of bug fixes needed per component
   - Time to fix bugs across system
```

**Design Metrics:**
```
1. Consistency
   - Visual variance between products
   - Brand guideline adherence

2. Quality
   - User satisfaction scores
   - Design system NPS

3. Efficiency
   - Time from design to implementation
   - Designer productivity increase
```

**Business Metrics:**
```
1. Cost savings
   - Development time saved
   - Fewer bugs in production
   - Reduced maintenance overhead

2. User impact
   - Accessibility improvements (a11y score)
   - Performance improvements
   - User satisfaction

3. Adoption
   - # of teams using system
   - # of components built
```

**Example Dashboard:**
```
Total Development Time Saved: 2,000 hours/year
Components Built: 85
Projects Using System: 12
Average Adoption Rate: 78%
Bug Reduction: 30%
Performance Improvement: 25%
Accessibility Score: 95/100
```

---

## Summary

A well-designed design system provides:

**Immediate Benefits:**
- Faster development cycles
- Consistent user experience
- Better accessibility
- Reduced CSS file size
- Team alignment

**Long-term Benefits:**
- Scalable to multiple teams
- Easier maintenance
- Strong brand identity
- Product quality improvements
- Cost savings

**Critical Success Factors:**
- Clear governance process
- Comprehensive documentation
- Active maintenance
- Strong team adoption
- Regular updates and versioning

Design systems are not just codethey're living products that evolve with your organization's needs and your users' expectations.

---

**Next:** [Back to CSSArchitecture](./README.md)

**Previous:** [� Atomic CSS](./04-atomic-css.md)

---

[� Back to CSSArchitecture](./README.md) | [� Back to Frontend](../README.md)
