# Next.js SEO Optimization

## Table of Contents
- [Metadata API](#metadata-api)
- [Dynamic Metadata](#dynamic-metadata)
- [Open Graph and Social](#open-graph-and-social)
- [JSON-LD Structured Data](#json-ld-structured-data)
- [Sitemap and Robots](#sitemap-and-robots)
- [Interview Questions](#interview-questions)

---

## Metadata API

### App Router Metadata

```jsx
// app/layout.js
export const metadata = {
  title: 'My App',
  description: 'Welcome to my app',
  keywords: ['next.js', 'react', 'javascript'],
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

### Page-Level Metadata

```jsx
// app/about/page.js
export const metadata = {
  title: 'About Us',
  description: 'Learn more about our company',
};

export default function AboutPage() {
  return <h1>About Us</h1>;
}
```

### Template Titles

```jsx
// app/layout.js
export const metadata = {
  title: {
    template: '%s | My App',
    default: 'My App',
  },
};

// app/blog/page.js
export const metadata = {
  title: 'Blog', // Becomes "Blog | My App"
};
```

---

## Dynamic Metadata

### generateMetadata Function

```jsx
// app/products/[id]/page.js
export async function generateMetadata({ params }) {
  const product = await fetchProduct(params.id);

  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.image],
    },
  };
}

export default async function Product({ params }) {
  const product = await fetchProduct(params.id);
  return <div>{product.name}</div>;
}
```

### Pages Router - Head Component

```jsx
// pages/products/[id].js
import Head from 'next/head';

export default function Product({ product }) {
  return (
    <>
      <Head>
        <title>{product.name} | My Store</title>
        <meta name="description" content={product.description} />
        <meta property="og:image" content={product.image} />
      </Head>
      <div>{product.name}</div>
    </>
  );
}

export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  return { props: { product } };
}
```

---

## Open Graph and Social

### Complete Open Graph Setup

```jsx
// app/layout.js or page.js
export const metadata = {
  title: 'My App',
  description: 'Best app ever',
  openGraph: {
    title: 'My App',
    description: 'Best app ever',
    url: 'https://example.com',
    siteName: 'My App',
    images: [
      {
        url: 'https://example.com/og-image.jpg',
        width: 1200,
        height: 630,
        alt: 'My App Preview',
      },
    ],
    locale: 'en_US',
    type: 'website',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My App',
    description: 'Best app ever',
    images: ['https://example.com/twitter-image.jpg'],
    creator: '@username',
  },
};
```

### Product Open Graph

```jsx
export const metadata = {
  title: 'Product Name',
  openGraph: {
    type: 'product',
    title: 'Product Name',
    description: 'Product description',
    images: ['https://example.com/product.jpg'],
    url: 'https://example.com/products/123',
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Product Name',
    description: 'Product description',
    images: ['https://example.com/product.jpg'],
  },
};
```

### Article Open Graph

```jsx
export const metadata = {
  title: 'Blog Post Title',
  openGraph: {
    type: 'article',
    title: 'Blog Post Title',
    description: 'Post summary',
    publishedTime: '2024-01-01T00:00:00.000Z',
    authors: ['Author Name'],
    tags: ['nextjs', 'react'],
  },
};
```

---

## JSON-LD Structured Data

### Basic JSON-LD

```jsx
export default function Page() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'My Company',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+1-555-555-5555',
      contactType: 'Customer Service',
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <h1>Home Page</h1>
    </>
  );
}
```

### Product Schema

```jsx
export default function Product({ product }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    description: product.description,
    image: product.image,
    offers: {
      '@type': 'Offer',
      priceCurrency: 'USD',
      price: product.price,
      availability: 'https://schema.org/InStock',
    },
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: product.rating,
      reviewCount: product.reviewCount,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <div>{product.name}</div>
    </>
  );
}
```

### Article Schema

```jsx
export default function BlogPost({ post }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Article',
    headline: post.title,
    description: post.excerpt,
    image: post.image,
    datePublished: post.publishedAt,
    dateModified: post.updatedAt,
    author: {
      '@type': 'Person',
      name: post.author.name,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <article>{post.content}</article>
    </>
  );
}
```

### Breadcrumb Schema

```jsx
export default function Page() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'BreadcrumbList',
    itemListElement: [
      {
        '@type': 'ListItem',
        position: 1,
        name: 'Home',
        item: 'https://example.com',
      },
      {
        '@type': 'ListItem',
        position: 2,
        name: 'Products',
        item: 'https://example.com/products',
      },
      {
        '@type': 'ListItem',
        position: 3,
        name: 'Product Name',
        item: 'https://example.com/products/123',
      },
    ],
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <div>Content</div>
    </>
  );
}
```

---

## Sitemap and Robots

### Static Sitemap

```jsx
// app/sitemap.js
export default function sitemap() {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'yearly',
      priority: 1,
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.5,
    },
  ];
}
```

### Dynamic Sitemap

```jsx
// app/sitemap.js
export default async function sitemap() {
  const posts = await fetchPosts();

  const postUrls = posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.5,
  }));

  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      priority: 1,
    },
    ...postUrls,
  ];
}
```

### Robots.txt

```jsx
// app/robots.js
export default function robots() {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/admin/', '/private/'],
    },
    sitemap: 'https://example.com/sitemap.xml',
  };
}
```

---

## Interview Questions

### Q1: How do you set up dynamic metadata in App Router?

**Answer:**

Use `generateMetadata` function:

```jsx
export async function generateMetadata({ params }) {
  const post = await fetchPost(params.id);

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      images: [post.image],
    },
  };
}
```

---

### Q2: What's the difference between Open Graph and Twitter Cards?

**Answer:**

**Open Graph:**
- Used by Facebook, LinkedIn, WhatsApp
- Standard protocol
- More properties

**Twitter Cards:**
- Twitter-specific
- Simpler format
- Falls back to Open Graph

```jsx
export const metadata = {
  openGraph: {
    title: 'My Page',
    images: ['https://example.com/og.jpg'],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'My Page',
    images: ['https://example.com/twitter.jpg'],
  },
};
```

---

### Q3: How do you implement structured data?

**Answer:**

Use JSON-LD script tags:

```jsx
export default function Product({ product }) {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Product',
    name: product.name,
    offers: {
      '@type': 'Offer',
      price: product.price,
    },
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <div>{product.name}</div>
    </>
  );
}
```

---

### Q4: How do you create a dynamic sitemap?

**Answer:**

```jsx
// app/sitemap.js
export default async function sitemap() {
  const posts = await fetchPosts();

  return posts.map((post) => ({
    url: `https://example.com/blog/${post.slug}`,
    lastModified: post.updatedAt,
    changeFrequency: 'weekly',
    priority: 0.8,
  }));
}
```

---

### Q5: What are the essential meta tags for SEO?

**Answer:**

```jsx
export const metadata = {
  // Basic SEO
  title: 'Page Title',
  description: 'Page description (155 chars)',
  keywords: ['keyword1', 'keyword2'],

  // Open Graph
  openGraph: {
    title: 'OG Title',
    description: 'OG Description',
    images: ['/og-image.jpg'],
  },

  // Twitter
  twitter: {
    card: 'summary_large_image',
    title: 'Twitter Title',
    images: ['/twitter-image.jpg'],
  },

  // Robots
  robots: {
    index: true,
    follow: true,
  },
};
```

---

## Key Takeaways

1. **Metadata API** - Static and dynamic metadata configuration
2. **generateMetadata** - Dynamic metadata for SSG/SSR pages
3. **Open Graph** - Social media previews
4. **JSON-LD** - Structured data for rich snippets
5. **Sitemap** - Help search engines crawl your site
6. **robots.txt** - Control crawler access
7. **Template titles** - Consistent title formatting

---

[← Back to Next.js](./README.md)
