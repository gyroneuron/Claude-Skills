---
name: front-end-design
description: Use when someone asks to create UI components, implement front-end designs, build responsive layouts, style elements with Tailwind CSS, or work with the design system tokens.
argument-hint: 'component name or design specification'
---

## What This Skill Does

Guides the creation of front-end components following the project's design system and Tailwind CSS conventions. Uses semantic design tokens for consistency across the application.

## Context

Before starting any design work:

1. Read the design tokens reference: `.claude/skills/front-end-design/design-tokens.md`
2. Check existing components for patterns and conventions
3. Review the tailwind.config.js for custom token definitions

### Existing Component Conventions

This project uses custom wrapper components from `src/shared/` instead of standard HTML elements:

**Core HTML Element Replacements:**

- **`<Div>`** instead of `<div>` - Custom div wrapper
- **`<Span>`** instead of `<span>` - Custom span wrapper
- **`<Link>`** instead of `<a>` - Custom link component with routing
- **`<Image>`** instead of `<img>` - Custom image component with optimization
- **`<Button>`** instead of `<button>` - Custom button with built-in styling
- **`<Form>`** instead of `<form>` - Custom form wrapper
- **`<Input>`** instead of `<input>` - Custom input with validation
- **`<Label>`** instead of `<label>` - Custom label component
- **`<Paragraph>`** instead of `<p>` - Custom paragraph component
- **`<Heading>`** instead of `<h1>`, `<h2>`, etc. - Custom heading component
- **`<Main>`** instead of `<main>` - Custom main content wrapper
- **`<Header>`** instead of `<header>` - Custom header component
- **`<Footer>`** instead of `<footer>` - Custom footer component
- **`<Container>`** instead of generic containers - Layout container
- **`<Table>`** instead of `<table>` - Custom table component
- **`<Textarea>`** instead of `<textarea>` - Custom textarea with features

**Additional Custom Components in src/shared/:**

- `<AppBar>` - Application bar component
- `<CategoryCard>` - Category display card
- `<Divider>` - Visual divider component
- `<FixedWrapper>` - Fixed positioning wrapper
- `<GeneralSuccessScreen>` - Success state display
- `<LinkButton>` - Button-styled link
- `<MultiCheckbox>` - Multiple checkbox group
- `<OtpInput>` - OTP input component
- `<Picture>` - Picture element wrapper
- `<RadioWithTextArea>` - Radio with text input
- `<TitleBanner>` - Title banner component
- `<Video>` - Video player component
- `<YoutubePlayer>` - YouTube embed component

**Import all components from `src/shared/`:**

```tsx
import { Div, Button, Link, Image, Span } from '@/shared';
```

### Network Requests

**Every network request must use the `useTriggerRequest` hook** — never use `fetch`, `axios`, or other async patterns directly in components.

**Import:**

```tsx
import useTriggerRequest from '@/hooks/useTriggerRequest';
```

**Signature:**

```typescript
const useTriggerRequest = ({ fetch, triggerOnMount = false }: UseTriggerRequestProps)
```

| Prop             | Type                               | Required              | Description                          |
| ---------------- | ---------------------------------- | --------------------- | ------------------------------------ |
| `fetch`          | `(...args: any[]) => Promise<any>` | Yes                   | The async action/API call to execute |
| `triggerOnMount` | `boolean`                          | No (default: `false`) | Auto-trigger on component mount      |

**Return values:**

```typescript
{
  isLoading: boolean,           // true while request is in-flight
  isError: boolean,             // true if last request failed
  isSuccess: boolean,           // true if last request succeeded
  data: any | null,             // response data on success
  error: any | null,            // error details on failure
  trigger: (...args) => void,              // fire-and-forget trigger
  promisifyTrigger: (...args) => Promise,  // awaitable trigger
  reset: () => void,            // reset state to initial values
}
```

**Choosing between `trigger` and `promisifyTrigger`:**

- Use `trigger` for simple side-effect calls where you only need to react to `isLoading`/`isSuccess`/`isError` via the returned state.
- Use `promisifyTrigger` when you need to `await` the result inline (e.g., chained requests, conditional logic after a response).

**Pattern — basic trigger with state-driven UI:**

```tsx
const { isLoading, isError, isSuccess, data, trigger } = useTriggerRequest({
  fetch: fetchProductList,
});

// Fire on button click
<Button onClick={() => trigger(payload)} disabled={isLoading}>
  Load Products
</Button>;

{
  isLoading && <Span>Loading...</Span>;
}
{
  isError && <Span className="text-cs-error">Something went wrong</Span>;
}
```

**Pattern — awaitable request with chaining:**

```tsx
const productList = useTriggerRequest({ fetch: fetchProductList });
const couponValidate = useTriggerRequest({ fetch: validateCoupon });

const handleSubmit = async () => {
  const response = await productList.promisifyTrigger(payload);
  if (response?.data) {
    await couponValidate.promisifyTrigger(couponCode);
  }
};
```

**Pattern — trigger on mount:**

```tsx
const { isLoading, data } = useTriggerRequest({
  fetch: fetchUserProfile,
  triggerOnMount: true,
});
```

**Pattern — multiple independent requests in one component:**

```tsx
const couponList = useTriggerRequest({ fetch: fetchCouponsData });
const couponValidateTrigger = useTriggerRequest({ fetch: couponValidate });
const couponApplyTrigger = useTriggerRequest({ fetch: couponApply });
```

**Pattern — reset after use:**

```tsx
const { isSuccess, trigger, reset } = useTriggerRequest({ fetch: submitForm });

useEffect(() => {
  if (isSuccess) {
    // Do something, then reset for next use
    reset();
  }
}, [isSuccess]);
```

### Rendering Lists

**Never use `array.map()` directly in JSX.** Always use the `<Map>` component from `src/components/Map` — it handles null/empty array guards, validates the render function, and falls back gracefully.

**Import:**

```tsx
import Map from '@/components/Map';
```

**Props:**
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `list` | `any[]` | Yes | The array to iterate over |
| `render` | `(item, index) => ReactNode` | Yes | Render function for each item |
| `wrapper` | `ElementType` | No | Wrapping element (e.g. `'ul'`, `Div`) |
| `className` | `string` | No | Applied to the wrapper element (also auto-creates a `div` wrapper when set) |
| `id` | `string` | No | `id` attribute on the wrapper |
| `emptyMessage` | `string` | No | Text shown when `list` is empty or null |
| `style` | `any` | No | Inline style on the wrapper |

**Why it's required:** The component internally checks `is.not.empty.array(list)` and `is.function(render)` before iterating — skipping those checks in every component leads to runtime errors and inconsistent empty states.

**Pattern — basic list:**

```tsx
<Map list={products} render={(product, index) => <ProductCard key={product.id} {...product} />} />
```

**Pattern — with wrapper and className:**

```tsx
<Map
  list={categories}
  className="grid md:grid-flow-col"
  render={(category, index) => <CategoryCard key={category.id} {...category} />}
/>
```

**Pattern — with custom wrapper element:**

```tsx
<Map
  list={navLinks}
  wrapper="ul"
  className="flex gap-s5"
  render={(link, index) => (
    <li key={link.href}>
      <Link href={link.href}>{link.label}</Link>
    </li>
  )}
/>
```

**Pattern — empty state message:**

```tsx
<Map
  list={orders}
  emptyMessage="No orders found"
  render={(order) => <OrderCard key={order.id} {...order} />}
/>
```

**DON'T:**

```tsx
// ❌ Never do this
{
  items.map((item) => <Card key={item.id} {...item} />);
}

// ❌ Never do this either
{
  items?.map((item) => <Card key={item.id} {...item} />);
}
```

## Steps

### 1. Analyze Design Requirements

When given a design task or component request:

- Identify the component type (layout, form, card, navigation, etc.)
- Determine responsive breakpoints needed
- List required interactive states (hover, focus, disabled, loading)
- Note any animations or transitions

### 2. Component Structure

Create the component following these conventions:

- Use semantic HTML elements
- Apply responsive-first approach (mobile → desktop)
- Structure with proper accessibility attributes
- Use TypeScript for type safety

### 3. Apply Design Tokens

**Always use design tokens, never arbitrary values:**

#### Spacing

- Use `s1` through `s11` tokens (e.g., `p-s5`, `gap-s6`, `mt-s7`)
- Never use arbitrary values like `p-4` or `mt-16`

#### Colors

- Primary choice: Semantic tokens (`cm-*`, `cu-*`, `cs-*`, `cti-*`, `co-*`)
- Fallback: Raw palette tokens (`neutral-*`, `blue-*`, `green-*`, `orange-*`)
- Never use hex values directly

#### Typography

- Desktop: `desktop-display`, `hd-1` through `hd-5`
- Mobile: `h-1` through `h-4`
- Body text: `body-xs` through `body-2xl`
- Buttons: `button-normal`, `button-small`
- Callouts: `callout-normal`, `callout-small`

#### Border Radius

- Use `rounded-cxxs` through `rounded-cxxl`
- Never use arbitrary radius values

#### Shadows

- Use `shadow-subtle`, `shadow-medium`, `shadow-bold`, `shadow-xbold`, `shadow-above`

### 4. Responsive Design

Apply responsive modifiers systematically:

```jsx
<div className="
  p-s4 md:p-s6 lg:p-s8
  text-h-3 md:text-hd-3
  grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3
">
```

### 5. Interactive States

Add appropriate state classes:

```jsx
<button className="
  bg-cm-brand hover:bg-blue-600
  disabled:bg-cs-disabled disabled:text-cti-disabled
  focus:ring-2 focus:ring-cu-selected
  transition-colors duration-200
">
```

### 6. Component Example Template

```tsx
import React from 'react';
import { Div, Button, Link, Image, Span, Heading, Paragraph } from '@/shared';

interface ${ARGUMENTS}Props {
  // Define props with TypeScript
}

const ${ARGUMENTS}: React.FC<${ARGUMENTS}Props> = ({ /* props */ }) => {
  return (
    <Div className="
      /* Layout */
      flex flex-col gap-s5

      /* Spacing */
      p-s5 md:p-s7

      /* Colors & Backgrounds */
      bg-cs-primary
      border border-co-subtle

      /* Typography */
      text-body-lg text-cti-primary

      /* Border Radius */
      rounded-cm

      /* Shadows */
      shadow-medium

      /* Responsive */
      md:flex-row md:gap-s7

      /* States */
      hover:shadow-bold
      transition-shadow duration-200
    ">
      {/* Component content */}
    </Div>
  );
};

export default ${ARGUMENTS};
```

## Output Format

When creating components:

1. Save TypeScript/TSX files to `src/components/[ComponentName]/`
2. Include index file for cleaner imports
3. Add any component-specific styles as Tailwind classes
4. Document complex styling decisions with comments

## Notes

### DO:

- Always use custom components from `src/shared/` instead of HTML elements
- Always use semantic design tokens from the system
- Follow mobile-first responsive approach
- Include proper TypeScript types
- Add accessibility attributes (aria-\*, role, etc.)
- Import components from `@/shared`
- Test across breakpoints
- Use `<Map>` from `@/components/Map` for every list render

### DON'T:

- Never use arbitrary Tailwind values when tokens exist
- Don't use inline styles unless absolutely necessary
- Avoid creating new CSS files (use Tailwind classes)
- Don't hardcode colors, spacing, or typography values
- Never skip accessibility considerations
- Never use `array.map()` directly in JSX — always use `<Map>` component

### Common Patterns:

**Cards:**

```jsx
<Div className="bg-cs-primary rounded-cm p-s5 shadow-medium hover:shadow-bold transition-shadow">
```

**Buttons:**

```jsx
<Button className="bg-cm-brand text-cti-on-primary px-s6 py-s4 rounded-cs text-button-normal">
```

**Forms:**

```jsx
<Input className="border border-co-bold rounded-cxs px-s4 py-s3 text-body-lg focus:border-cu-selected" />
```

**Responsive Grid:**

```jsx
<Div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-s5 md:gap-s6 lg:gap-s7">
```

### Reference Files:

- Design tokens: `.claude/skills/front-end-design/design-tokens.md`
- Tailwind config: `tailwind.config.js`
- Existing components: `src/components/`
