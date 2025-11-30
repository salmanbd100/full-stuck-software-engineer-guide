# Next.js Image Optimization

## Table of Contents
- [Next.js Image Component](#nextjs-image-component)
- [Image Formats and Optimization](#image-formats-and-optimization)
- [Responsive Images](#responsive-images)
- [Performance Optimization](#performance-optimization)
- [Remote Images](#remote-images)
- [Interview Questions](#interview-questions)

---

## Next.js Image Component

The `next/image` component automatically optimizes images for better performance.

### Basic Usage

```jsx
import Image from 'next/image';

export default function Page() {
  return (
    <Image
      src="/profile.jpg"
      alt="Profile picture"
      width={500}
      height={300}
    />
  );
}
```

**Benefits:**
- Automatic lazy loading
- Prevents Cumulative Layout Shift (CLS)
- Resizes images on-demand
- Serves modern formats (WebP, AVIF)
- Optimized loading with priority flag

### Required Props

```jsx
<Image
  src="/photo.jpg"        // Required
  alt="Description"       // Required for accessibility
  width={800}             // Required (unless fill)
  height={600}            // Required (unless fill)
/>
```

---

## Image Formats and Optimization

### Automatic Format Optimization

Next.js automatically serves WebP/AVIF when browser supports it.

```jsx
// Serves as WebP or AVIF (if supported)
<Image src="/photo.jpg" width={800} height={600} alt="Photo" />
```

### Quality Settings

```jsx
// Default quality: 75
<Image src="/photo.jpg" quality={75} width={800} height={600} alt="Photo" />

// High quality: 90
<Image src="/photo.jpg" quality={90} width={800} height={600} alt="Photo" />

// Low quality placeholder: 10
<Image
  src="/photo.jpg"
  quality={10}
  placeholder="blur"
  blurDataURL="/photo-blur.jpg"
  width={800}
  height={600}
  alt="Photo"
/>
```

### Placeholder Blur

```jsx
import myImage from '/public/photo.jpg';

// Static import (automatic blur)
<Image
  src={myImage}
  alt="Photo"
  placeholder="blur"
/>

// Remote image with custom blur
<Image
  src="https://example.com/photo.jpg"
  width={800}
  height={600}
  alt="Photo"
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

**Generate blur data URL:**
```bash
npm install plaiceholder
```

```jsx
import { getPlaiceholder } from 'plaiceholder';

export async function getStaticProps() {
  const { base64 } = await getPlaiceholder('/public/photo.jpg');

  return {
    props: { blurDataURL: base64 }
  };
}
```

---

## Responsive Images

### Fill Container

Use `fill` for responsive images that fill their parent container.

```jsx
<div style={{ position: 'relative', width: '100%', height: '400px' }}>
  <Image
    src="/photo.jpg"
    alt="Photo"
    fill
    style={{ objectFit: 'cover' }}
  />
</div>
```

**objectFit options:**
```jsx
// Cover (crop to fill)
<Image src="/photo.jpg" fill style={{ objectFit: 'cover' }} alt="Photo" />

// Contain (fit inside)
<Image src="/photo.jpg" fill style={{ objectFit: 'contain' }} alt="Photo" />

// Fill (stretch)
<Image src="/photo.jpg" fill style={{ objectFit: 'fill' }} alt="Photo" />
```

### Responsive Sizes

Define different sizes for different breakpoints.

```jsx
<Image
  src="/photo.jpg"
  alt="Photo"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
/>
```

**Sizes explained:**
- `(max-width: 768px) 100vw` → Mobile: full width
- `(max-width: 1200px) 50vw` → Tablet: half width
- `33vw` → Desktop: one-third width

---

## Performance Optimization

### Priority Loading

Load critical images immediately (above the fold).

```jsx
// Hero image - load with priority
<Image
  src="/hero.jpg"
  alt="Hero"
  width={1920}
  height={1080}
  priority
/>

// Below fold - lazy load (default)
<Image
  src="/content.jpg"
  alt="Content"
  width={800}
  height={600}
/>
```

### Lazy Loading

Lazy loading is enabled by default for non-priority images.

```jsx
// Lazy load (default)
<Image src="/photo.jpg" width={800} height={600} alt="Photo" />

// Eager load
<Image src="/photo.jpg" width={800} height={600} loading="eager" alt="Photo" />

// Priority (eager + preload)
<Image src="/hero.jpg" width={1920} height={1080} priority alt="Hero" />
```

### Loading States

```jsx
import Image from 'next/image';
import { useState } from 'react';

export default function LoadingImage() {
  const [isLoading, setIsLoading] = useState(true);

  return (
    <div className={isLoading ? 'blur' : ''}>
      <Image
        src="/photo.jpg"
        width={800}
        height={600}
        alt="Photo"
        onLoadingComplete={() => setIsLoading(false)}
      />
    </div>
  );
}
```

---

## Remote Images

### Configure Remote Domains

```js
// next.config.js
module.exports = {
  images: {
    domains: ['example.com', 'cdn.example.com'],
    // OR use remotePatterns (more flexible)
    remotePatterns: [
      {
        protocol: 'https',
        hostname: '**.example.com',
        port: '',
        pathname: '/images/**',
      },
    ],
  },
};
```

### Remote Image Usage

```jsx
<Image
  src="https://example.com/photo.jpg"
  width={800}
  height={600}
  alt="Remote photo"
/>
```

### Loader Configuration

Custom image loader for CDN.

```js
// next.config.js
module.exports = {
  images: {
    loader: 'custom',
    loaderFile: './my-loader.js',
  },
};
```

```js
// my-loader.js
export default function myImageLoader({ src, width, quality }) {
  return `https://cdn.example.com/${src}?w=${width}&q=${quality || 75}`;
}
```

---

## Interview Questions

### Q1: What are the benefits of using next/image?

**Answer:**

1. **Automatic optimization:**
   - Serves modern formats (WebP, AVIF)
   - Resizes images on-demand
   - Optimizes quality

2. **Performance:**
   - Lazy loading by default
   - Prevents layout shift (CLS)
   - Priority loading for critical images

3. **Responsive:**
   - Automatic srcset generation
   - Size optimization per device

```jsx
<Image
  src="/photo.jpg"
  width={800}
  height={600}
  alt="Photo"
  quality={75}
  placeholder="blur"
/>
```

---

### Q2: How do you prevent Cumulative Layout Shift with images?

**Answer:**

Always provide `width` and `height` or use `fill` with a sized container.

```jsx
// Method 1: width and height
<Image
  src="/photo.jpg"
  width={800}
  height={600}
  alt="Photo"
/>

// Method 2: fill with sized container
<div style={{ position: 'relative', width: '100%', height: '400px' }}>
  <Image src="/photo.jpg" fill alt="Photo" />
</div>
```

---

### Q3: When should you use the `priority` prop?

**Answer:**

Use `priority` for images visible above the fold (LCP images).

```jsx
// Hero image - above the fold
<Image
  src="/hero.jpg"
  width={1920}
  height={1080}
  priority  // Load immediately
  alt="Hero"
/>

// Content image - below the fold
<Image
  src="/content.jpg"
  width={800}
  height={600}
  // Lazy loaded by default
  alt="Content"
/>
```

**Don't overuse:** Only 1-2 priority images per page.

---

### Q4: How do you implement responsive images?

**Answer:**

Use the `sizes` prop to specify different widths for different viewports.

```jsx
<Image
  src="/photo.jpg"
  width={1200}
  height={800}
  sizes="(max-width: 768px) 100vw,
         (max-width: 1200px) 50vw,
         33vw"
  alt="Responsive photo"
/>
```

This tells Next.js:
- Mobile (≤768px): Use full viewport width
- Tablet (≤1200px): Use 50% viewport width
- Desktop (>1200px): Use 33% viewport width

---

### Q5: How do you use images from external sources?

**Answer:**

Configure remote patterns in `next.config.js`:

```js
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
        pathname: '/**',
      },
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
        pathname: '/images/**',
      },
    ],
  },
};
```

Then use normally:

```jsx
<Image
  src="https://images.unsplash.com/photo-123"
  width={800}
  height={600}
  alt="Photo"
/>
```

---

## Key Takeaways

1. **Always use next/image** for automatic optimization
2. **Provide dimensions** to prevent layout shift
3. **Use priority** for above-the-fold images only
4. **Configure remote patterns** for external images
5. **Use sizes** for responsive images
6. **Add blur placeholders** for better UX
7. **Optimize quality** based on use case (75 default, 90 for important images)

---

[← Back to Next.js](./README.md)
