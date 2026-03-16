### SSR

Server-Side Rendering (SSR) is the process of rendering React components on the server instead of in the browser. The server generates the complete HTML for a page and sends it to the client, where it becomes interactive (hydration).

- Without ssr - on the 1st req the browser gets a empty html page, where the react send a req and gets the js file
- With ssr - the browser gets the full contents inside the html page
- Way to use - `renderToString` - which is not a industry practise instead use other frameworks(nextjs,remix)

### SWR

**SWR** (Stale-While-Revalidate) is a React Hooks library for data fetching created by Vercel.

- **Return cached (stale) data first** - Show something immediately
- **Send a fetch request** - Get fresh data in the background
- **Update with fresh data** - When the request completes
- If using tanstack no need to use this swr lib

### HydrateRoot

- using react send empty html to the browser - problem here is the user see a blank page
- ssr gives us the HTML - the content is visible immediately but the js still not arrived hence the button or event listeners wont work
- use `hydrateRoot` to make it interactive
- this is used when using SSR in react

```js
hydrateRoot(
  document.getElementById("root"), // Find the existing HTML
  <App />, // Attach React logic to it
);
// can use any component of our choice
```

- using this raw format is not industry practise instead use frameworks(nextjs..)
- Without it: Fast initial paint, but nothing works
  With it: Fast initial paint AND everything works
- if the app is large the perf decreases, instead use `lazy` & `Suspense`

Under the hood - `hydrateRoot` ,`lazy` & `Suspense` are solving the same problem

- `lazy loading` - instead of bundling the whole app into one file, instead we can lazy load large component on when they are used
- `Suspense` - suspense just populate the UI with loading until the content arrives

### Server Components

**Server Components** are a new type of React component that **run and render EXCLUSIVELY on the server** - they never run on the client, never hydrate, and their code never gets sent to the browser.

- React + vite does not have server component
- Server component is a feature in nextjs

```jsx
// Traditional Thinking (CSR + SSR):
// Everything runs everywhere (server AND client)

// Server Components Thinking:
// Components have a PERMANENT home - either server or client
```

- Example

```jsx
// app/products/[id]/page.js - Server Component by DEFAULT!
import { notFound } from "next/navigation";

// This is a Server Component (no 'use client' needed)
export default async function ProductPage({ params }) {
  // Direct database access (using an ORM)
  const product = await prisma.product.findUnique({
    where: { id: params.id },
  });

  if (!product) notFound();

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <Price price={product.price} />
      <AddToCart productId={product.id} /> {/* Client component */}
    </div>
  );
}

// Price.client.jsx - Client Component
("use client");
function Price({ price }) {
  // Can use hooks, state, effects
  const [currency, setCurrency] = useState("USD");
  return <div>{formatPrice(price, currency)}</div>;
}
```

### 1. **SSR - Server-Side Rendering**

_(Dynamic - Fresh on every request)_

#### SSR Example:

```jsx
// app/orders/[id]/page.js
export default async function OrderPage({ params }) {
  // This runs on EVERY request - perfect for:
  // - Order status (changes frequently)
  // - User-specific data
  // - Real-time information

  const order = await db.order.findUnique({
    where: { id: params.id },
    include: { user: true },
  });

  if (!order) {
    notFound(); // Shows 404 page
  }

  return (
    <div>
      <h1>Order #{order.id}</h1>
      <p>Status: {order.status}</p>
      <p>Last updated: {order.updatedAt}</p>

      {/* Client component for real-time updates */}
      <OrderStatusTracker orderId={order.id} />
    </div>
  );
}

// app/orders/[id]/OrderStatusTracker.js - Client Component
("use client");
import { useState, useEffect } from "react";

export function OrderStatusTracker({ orderId }) {
  const [status, setStatus] = useState(null);

  useEffect(() => {
    // Real-time updates on client
    const ws = new WebSocket(`ws://api.com/orders/${orderId}`);
    ws.onmessage = (e) => setStatus(JSON.parse(e.data));

    return () => ws.close();
  }, [orderId]);

  return <div>Live status: {status}</div>;
}
```

### 2. **SSG - Static Site Generation**

_(Static - Built once, served everywhere)_

### SSG with Fallback:

```jsx
// app/products/[id]/page.js
export async function generateStaticParams() {
  // Only generate for popular products at build time
  const popularProducts = await fetch(
    'https://api.example.com/products/popular'
  ).then(r => r.json());

  return popularProducts.map((product) => ({
    id: product.id.toString()
  }));
}

export default async function ProductPage({ params }) {
  const product = await fetch(
    `https://api.example.com/products/${params.id}`
  ).then(r => r.json());

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}

// app/products/[id]/not-found.js - Custom 404 for non-generated paths
export default function NotFound() {
  return <div>Product not found</div>;
}
```

### 3. **ISR - Incremental Static Regeneration**

_(Hybrid - Static but updates periodically)_

#### App Router Implementation:

```jsx
// app/products/page.js
// ISR with time-based revalidation

export default async function ProductsPage() {
  // This runs at build time AND every hour after
  const products = await fetch("https://api.example.com/products", {
    // ⏰ ISR: Revalidate every hour
    next: { revalidate: 3600 },
  }).then((r) => r.json());

  return (
    <div>
      <h1>Products</h1>
      <div className="grid">
        {products.map((product) => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}

// Configure ISR for the route
export const revalidate = 3600; // Revalidate every hour
```
