# Next.js Guide

This document provides a comprehensive guide to using Next.js, covering project setup, routing, styling, data fetching, database integration, testing, internationalization, authentication, and more.

## Project Setup

To create a new Next.js project, you can use one of the following commands:

```bash
npm create next-app@latest <project-name>
npx create-next-app@latest
pnpm create next-app <project-name>
```

## Routing

### Pages Directory (`/pages`)

- The `/pages` directory is used for routing in Next.js (similar to SvelteKit).
- Avoid using hyphens (`-`) in route names.

#### Example: Static Routes
- A file named `/pages/MiRuta/index.jsx` creates a route at `/MiRuta`.
- The `index.jsx` file in a folder defines the content for that route.

```tsx
export default function Index() {
  return (
    <>
      Hello
    </>
  );
}
```

### App Directory (`/app`)

- The `/app` directory is used for the App Router in Next.js.
- The `page.tsx` (or `.jsx`) file in `/app` is the root view of the project.
- Each folder with a `page.tsx` or `page.jsx` file creates a new route.
- The component in `page.tsx` is typically named `Page`, though this is not mandatory.

```tsx
export default function Page() {
  return (
    <>
      <h2>Mi ruta1</h2>
    </>
  );
}
```

### Layouts (`/app/layout.tsx`)

- The `layout.tsx` file defines a layout component that wraps its children based on the folder structure.

```tsx
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      Mi Layout
      {children}
      End Mi Layout
    </>
  );
}
```

### Dynamic Routes

- Dynamic routes are created using square brackets for parameters, e.g., `/pages/ruta/[parametro].jsx`.
- Parameters are accessed via the `params` prop in the `Page` component.

```tsx
export default function Page({ params }: { params: { productId: string } }) {
  return (
    <>
      <p>El id es: {params.productId}</p>
    </>
  );
}
```

### Route Grouping

- Folders with names in parentheses, e.g., `/app/(agrupada)/ruta1`, are ignored in the URL but their children routes are not.
- Example structure:
  ```
  /app/(agrupada)/ruta1
  /app/(agrupada)/ruta2
  /app/ruta3
  ```
- Resulting URLs:
  - `/ruta1`
  - `/ruta2`
  - `/ruta3`

### Hiding Routes

- Prefix a folder name with an underscore (`_`) to hide it from routing, e.g., `/app/_oculto`.

### Parallel Routes (`@folder`)

- Parallel routes allow rendering multiple components within the same route.
- Example structure:
  ```
  /ruta/layout.tsx
  /ruta/@chat/page.tsx
  /ruta/@video/page.tsx
  /ruta/page.tsx
  ```
- The `layout.tsx` combines the parallel routes:

```tsx
export default function Layout({
  children,
  chat,
  video,
}: {
  children: React.ReactNode;
  chat: React.ReactNode;
  video: React.ReactNode;
}) {
  return (
    <>
      Mi Layout Complejo
      {chat}
      -----------
      {children}
      ---------
      {video}
      Mi Layout Complejo
    </>
  );
}
```

- `/ruta/@chat/page.tsx`:

```tsx
export default function Page() {
  return (
    <>
      <p>El chat2</p>
    </>
  );
}
```

- `/ruta/@video/page.tsx`:

```tsx
export default function Page() {
  return (
    <>
      <p>333El video</p>
    </>
  );
}
```

- `/ruta/page.tsx`:

```tsx
export default function Page() {
  return <>colmplejo</>;
}
```

## Static Assets

- Place static files (e.g., images) in the `/public` directory.
- The `/public` folder is considered the root, so it’s not included in paths.

### Using `next/link`

```tsx
import Link from 'next/link';

<Link href="/ruta/mivista">Algo</Link>
```

### Using `next/image`

```tsx
import Image from 'next/image';

<Image src="/image.jpg" width={600} height={600} alt="descripcion" />
```

## Styling

### CSS Modules

- Styles are defined in `/styles/miestilos.css`.
- Use class names, not HTML tags, for styling.

```css
.container {
  max-width: 36rem;
  font-size: large;
  color: blue;
}
```

- In the component:

```tsx
import styles from '../styles/Layout.module.css';

<div className={styles.container}>Content</div>
```

### Global Styles

- Import global styles in a layout:

```tsx
import './globals.css';
```

### Google Fonts

- Define fonts in `/app/ui/fonts.tsx`:

```tsx
import { Inter, Montserrat } from 'next/font/google';

export const montserrat = Montserrat({ subsets: ['latin'] });
```

- Use in a component:

```tsx
import { montserrat } from './ui/fonts';

<h1 className={`${montserrat.className} antialiased`}>Hola</h1>
```

## Data Fetching

### `getStaticProps`

- Used for static site generation, runs at build time, and does not compile to the frontend.

```tsx
export default function Index({ data }) {
  return (
    <>
      Hello
      {data.map(({ id, title, body }, index) => (
        <div key={id}>
          <h3>
            {id} - {title}
          </h3>
          <p>{body}</p>
        </div>
      ))}
    </>
  );
}

export async function getStaticProps() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts');
    const data = await response.json();
    return {
      props: { data },
      revalidate: 60, // 1 minute
    };
  } catch (error) {
    console.error(error);
  }
}
```

### `getServerSideProps`

- Used for server-side rendering, runs on each request.
- Must be exported as a standalone function in the `/pages` directory.

```tsx
export async function getServerSideProps() {
  // Fetch data
  return {
    props: {}, // Pass data to the page
  };
}
```

## Client-Side Rendering

- Use the `'use client'` directive to render components on the client.

```tsx
'use client';

import { usePathname } from 'next/navigation';

export default function Page() {
  const pathname = usePathname();
  return (
    <>
      <h2>Mi ruta2 {pathname}</h2>
    </>
  );
}
```

## Loading States

### Loading Component

- Create a `loading.tsx` file at the same level as `page.tsx` to display while the page loads.

```tsx
export default function Loading() {
  return <>Cargando...</>;
}
```

### Suspense

- Use React’s `Suspense` for components that load slowly, providing a fallback UI.

```tsx
import ComponenteLento from '@/app/lib/components/component1';
import { Suspense } from 'react';

export default function Page() {
  return (
    <>
      Lento2
      <Suspense fallback={<>Espere...</>}>
        <ComponenteLento />
      </Suspense>
      Lento2
    </>
  );
}
```

- For dynamic updates, add a `key` to `Suspense`:

```tsx
import { Suspense } from 'react';
import Search from '../lib/components/search';
import ComponenteLento from '../lib/components/component1';

export default function Page({
  searchParams,
}: {
  searchParams?: { query?: string; page?: string };
}) {
  const currentPage = Number(searchParams?.page) || 1;
  const query = searchParams?.query || '';
  return (
    <>
      Busqueda: <Search placeholder="termino" />
      <p>
        <span>Page:</span>
        {currentPage}
      </p>
      <p>
        <span>Query:</span>
        {query}
      </p>
      <Suspense key={query + currentPage} fallback={<>Espere...</>}>
        <ComponenteLento />
      </Suspense>
    </>
  );
}
```

## Server Actions

- Define server actions in `/app/lib/actions.ts` with the `'use server'` directive.

```tsx
'use server';

export async function createInvoice(formData: FormData) {
  console.log(formData);
}
```

- Use in a form component:

```tsx
import { createInvoice } from '../actions';

export default function FormExample() {
  return (
    <div className="form" style={{ color: 'red' }}>
      <form action={createInvoice}>
        <label>Name:</label>
        <input type="text" name="name" />
        <label>Age:</label>
        <input type="number" name="age" />
        <button type="submit">Submit</button>
      </form>
    </div>
  );
}
```

## Validation with Zod

- Install Zod:

```bash
pnpm install zod
```

- Define schemas:

```tsx
'use server';

import { z } from 'zod';

const CreateInvoiceSchema = z.object({
  id: z.string(),
  invoice_number: z.coerce.number(),
  customer_name: z.coerce.string(),
});

const CreateInvoiceFormSchema = CreateInvoiceSchema.omit({
  id: true,
});

type TypeSchemaForm = z.infer<typeof CreateInvoiceFormSchema>;
```

## Database Integration

### Postgres with Vercel

- Install:

```bash
pnpm install @vercel/postgres
```

### Prisma

- Install Prisma and related dependencies:

```bash
pnpm install prisma -D
pnpm install @prisma/client
pnpm install tsx --save-dev
```

- Initialize Prisma:

```bash
npx prisma init
# For SQLite
npx prisma init --datasource-provider sqlite
```

- Configure `.env`:

```env
DATABASE_URL="file:./dev.db"
```

- Define models in `/prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = env("DATABASE_URL")
}

model Note {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

- Run migrations:

```bash
npx prisma migrate dev --name init
```

- Generate client if needed:

```bash
pnpm prisma generate
```

- Seed the database:

```bash
npx prisma db seed
```

- Example seed script (`/prisma/seed.ts`):

```tsx
import prisma from '@/libs/prisma';

export async function main() {
  const categories = await Promise.all([
    prisma.category.create({
      data: {
        name: 'Electrodomésticos',
        image: '/assets/categories/img/electrodomesticos.png',
      },
    }),
    // Additional categories...
  ]);

  console.log('Seeding completado exitosamente!');
  console.log('Categorías creadas:', categories.length);
}

main()
  .catch((e) => {
    console.error('Error durante el seeding:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

- Configure Prisma client in `/src/libs/prisma.ts`:

```tsx
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma = global.prisma || new PrismaClient();

if (process.env.NODE_ENV !== 'production') {
  global.prisma = prisma;
}
```

- Example API route with Prisma:

```tsx
import { prisma } from '@/libs/prisma';

export async function GET() {
  const notes = await prisma.note.findMany();
  return NextResponse.json(notes);
}
```

### MongoDB with Mongoose

- Install:

```bash
pnpm install mongodb mongoose
```

- Configure `.env`:

```env
MONGODB_URI="mongodb://localhost:27017"
```

- Set up connection in `/src/lib/mongoose.ts`:

```tsx
import mongoose from 'mongoose';

const MONGODB_URI = process.env.MONGODB_URI;

const connect = async () => {
  const connectionState = mongoose.connection.readyState;
  if (connectionState === 1) {
    console.log('already connected');
    return;
  }
  if (connectionState === 2) {
    console.log('Connecting...');
    return;
  }
  try {
    await mongoose.connect(MONGODB_URI!, {
      dbName: 'testusers',
      bufferCommands: true,
    });
    console.log('connected');
  } catch (err) {
    console.log('error');
    console.log(err);
    throw new Error('Error', err);
  }
};

export default connect;
```

- Define a schema:

```tsx
import { Schema, model, models } from 'mongoose';

const UserSchema = new Schema(
  {
    email: { type: String, required: true, unique: true },
    username: { type: String, required: true, unique: true },
    password: { type: String, required: true },
  },
  { timestamps: true }
);

const User = models.User || model('User', UserSchema);
export default User;
```

## Testing

### Setup Testing Libraries

- Install dependencies:

```bash
pnpm install --save-dev @testing-library/react @testing-library/dom @types/react @types/react-dom
pnpm install -D jest jest-environment-jsdom @testing-library/jest-dom ts-node @types/jest
pnpm install ts-jest
pnpm install eslint-plugin-jest-dom eslint-plugin-testing-library
```

- Configure ESLint in `eslint.config.mjs`:

```javascript
import { dirname } from 'path';
import { fileURLToPath } from 'url';
import { FlatCompat } from '@eslint/eslintrc';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const compat = new FlatCompat({
  baseDirectory: __dirname,
});

const eslintConfig = [
  ...compat.extends(
    'next/core-web-vitals',
    'next/typescript',
    'plugin:testing-library/react',
    'plugin:jest-dom/recommended'
  ),
];

export default eslintConfig;
```

- Configure Jest in `jest.config.ts`:

```typescript
import type { Config } from 'jest';
import nextJest from 'next/jest.js';

const createJestConfig = nextJest({
  dir: './',
});

const config: Config = {
  coverageProvider: 'v8',
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  preset: 'ts-jest',
};

export default createJestConfig(config);
```

- Create `jest.setup.ts`:

```typescript
import '@testing-library/jest-dom';
```

- Add scripts to `package.json`:

```json
{
  "scripts": {
    "test": "jest --passWithNoTests",
    "test:watch": "jest --watchAll"
  }
}
```

- Run tests:

```bash
pnpm test
```

### Writing Tests

- Place tests in `src/__tests__` or alongside components as `ComponentName.test.tsx`.
- Example test for a homepage:

```tsx
import { render, screen } from '@testing-library/react';
import HomePage from '@/app/page';

describe('Home page', () => {
  it('El texto', () => {
    render(<HomePage />);
    expect(screen.getByText('El texto')).toBeInTheDocument();
  });
});
```

- Test with `beforeEach`:

```tsx
import { render, screen } from '@testing-library/react';
import HomePage from '@/app/page';

describe('Home page', () => {
  beforeEach(() => {
    render(<HomePage />);
  });
  it('El texto', () => {
    expect(screen.getByText('El texto')).toBeInTheDocument();
  });
});
```

- Test by role:

```tsx
expect(
  screen.getByRole('heading', {
    name: /Titulo/i,
  })
).toBeInTheDocument();
```

- Test with `data-testid`:

```tsx
<span data-testid="idtexto">El texto</span>

it('Titulo id test', () => {
  render(<HomePage />);
  expect(screen.getByTestId('idtexto')).toHaveTextContent(/texto/);
});
```

- Test input changes:

```tsx
import { fireEvent, render, screen } from '@testing-library/react';

it('escribir texto abilita el boton', () => {
  render(<HomePage />);
  const inputText = screen.getByPlaceholderText('escriba algo');
  fireEvent.change(inputText, { target: { value: 'new text' } });
  const boton = screen.getByRole('button');
  expect(boton).toBeEnabled();
});
```

- Test with `within`:

```tsx
import { fireEvent, render, screen, within } from '@testing-library/react';

it('sin notas al principio', () => {
  render(<HomePage />);
  const notesContainer = screen.getByTestId('idtestnotes');
  const { queryAllByRole } = within(notesContainer);
  const listItems = queryAllByRole('listitem');
  expect(listItems).toHaveLength(0);
});
```

### Mocking

- Mock a function with `jest.fn()`:

```tsx
const mock = jest.fn();
mock('hola', { id: 1 });
expect(mock.calls).toEqual([['hola', { id: 1 }]]);
```

- Mock `useRouter`:

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { useRouter } from 'next/navigation';
import RedirectPage from '@/app/redirect/page';

jest.mock('next/navigation', () => ({ useRouter: jest.fn() }));

describe('Redirect page', () => {
  it('probar redirect', async () => {
    const mockPush = jest.fn();
    useRouter.mockReturnValue({ push: mockPush });
    render(<RedirectPage />);
    const boton = screen.getByRole('button');
    fireEvent.click(boton);
    expect(mockPush).toHaveBeenCalledWith('other');
  });
});
```

## Internationalization (i18n)

### Using `next-i18n-router`

- Install:

```bash
pnpm install next-i18n-router i18next --save
```

- Configure `i18nConfig.ts`:

```typescript
const i18nConfig = {
  locales: ['en', 'es'],
  defaultLocale: 'es',
};

export default i18nConfig;
```

- Set up middleware in `src/middleware.ts`:

```typescript
import { i18nRouter } from 'next-i18n-router';
import i18nConfig from '../i18nConfig';
import { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  return i18nRouter(request, i18nConfig);
}

export const config = {
  matcher: '/((?!api|static|.*\\..*|_next).*)',
};
```

- Move routes to `src/app/[locale]`:

```
└── app
    └── [locale]
        ├── layout.tsx
        └── page.tsx
```

### Using `next-intl`

- Install:

```bash
pnpm install next-intl
```

- Project structure:

```
├── messages
│   ├── en.json
│   └── es.json
├── next.config.ts
└── src
    ├── i18n
    │   ├── routing.ts
    │   ├── navigation.ts
    │   └── request.ts
    ├── middleware.ts
    └── app
        └── [locale]
            ├── layout.tsx
            └── page.tsx
```

- Configure `next.config.ts`:

```typescript
import type { NextConfig } from 'next';
import createNextIntlPlugin from 'next-intl/plugin';

const nextConfig: NextConfig = {};
const withNextIntl = createNextIntlPlugin();
export default withNextIntl(nextConfig);
```

- Define translations in `messages/en.json`:

```json
{
  "HomePage": {
    "title": "Hello world!",
    "about": "Go to the about page"
  }
}
```

- Configure routing in `src/i18n/routing.ts`:

```typescript
import { defineRouting } from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['en', 'es'],
  defaultLocale: 'es',
});
```

- Set up navigation in `src/i18n/navigation.ts`:

```typescript
import { createNavigation } from 'next-intl/navigation';
import { routing } from './routing';

export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing);
```

- Configure request in `src/i18n/request.ts`:

```typescript
import { getRequestConfig } from 'next-intl/server';
import { hasLocale } from 'next-intl';
import { routing } from './routing';

export default getRequestConfig(async ({ requestLocale }) => {
  const requested = await requestLocale;
  const locale = hasLocale(routing.locales, requested)
    ? requested
    : routing.defaultLocale;

  return {
    locale,
    messages: (await import(`../../messages/${locale}.json`)).default,
  };
});
```

- Update middleware in `src/middleware.ts`:

```typescript
import createMiddleware from 'next-intl/middleware';
import { routing } from './i18n/routing';

export default createMiddleware(routing);

export const config = {
  matcher: '/((?!api|trpc|_next|_vercel|.*\\..*).*)',
};
```

- Create the main layout in `src/app/[locale]/layout.tsx`:

```tsx
import { NextIntlClientProvider, hasLocale } from 'next-intl';
import { notFound } from 'next/navigation';
import { routing } from '@/i18n/routing';
import '../globals.css';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};

type Props = {
  children: React.ReactNode;
  params: Promise<{ locale: string }>;
};

export default async function RootLayout({
  children,
  params,
}: Readonly<Props>) {
  const { locale } = await params;
  if (!hasLocale(routing.locales, locale)) {
    notFound();
  }
  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider>{children}</NextIntlClientProvider>
      </body>
    </html>
  );
}
```

- Create a page in `src/app/[locale]/page.tsx`:

```tsx
import React from 'react';
import { useTranslations } from 'next-intl';
import { Link } from '@/i18n/navigation';

const Page = () => {
  const t = useTranslations('HomePage');
  return (
    <div>
      <h1>{t('title')}</h1>
      <Link href="/traslate">{t('about')}</Link>
    </div>
  );
};

export default Page;
```

- Language switcher in `LocaleSwitcher.tsx`:

```tsx
import { useLocale, useTranslations } from 'next-intl';
import { routing } from '@/i18n/routing';
import LocaleSwitcherSelect from './LocaleSwitcherSelect';

export default function LocaleSwitcher() {
  const t = useTranslations('LocaleSwitcher');
  const locale = useLocale();

  return (
    <LocaleSwitcherSelect defaultValue={locale} label={t('label')}>
      {routing.locales.map((cur) => (
        <option key={cur} value={cur}>
          {t(cur)}
        </option>
      ))}
    </LocaleSwitcherSelect>
  );
}
```

- `LocaleSwitcherSelect.tsx`:

```tsx
'use client';

import { useParams } from 'next/navigation';
import { Locale } from 'next-intl';
import { ChangeEvent, ReactNode, useTransition } from 'react';
import { usePathname, useRouter } from '@/i18n/navigation';

type Props = {
  children: ReactNode;
  defaultValue: string;
  label: string;
};

export default function LocaleSwitcherSelect({
  children,
  defaultValue,
  label,
}: Props) {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();
  const pathname = usePathname();
  const params = useParams();

  function onSelectChange(event: ChangeEvent<HTMLSelectElement>) {
    const nextLocale = event.target.value as Locale;
    startTransition(() => {
      router.replace({ pathname, params }, { locale: nextLocale });
    });
  }

  return (
    <label
      className={`relative text-gray-400 ${
        isPending && 'transition-opacity [&:disabled]:opacity-30'
      }`}
    >
      <p className="sr-only">{label}</p>
      <select
        className="inline-flex appearance-none bg-transparent py-3 pl-2 pr-6"
        defaultValue={defaultValue}
        disabled={isPending}
        onChange={onSelectChange}
      >
        {children}
      </select>
      <span className="pointer-events-none absolute right-2 top-[8px]">⌄</span>
    </label>
  );
}
```

## Authentication with NextAuth

### Setup

- Install:

```bash
pnpm add next-auth@beta
npx auth secret
```

- Configure `src/auth.ts`:

```tsx
import NextAuth from 'next-auth';

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [],
});
```

- Set up API route in `src/app/api/auth/[...nextauth]/route.ts`:

```tsx
import { handlers } from '@/auth';

export const { GET, POST } = handlers;
```

- Configure middleware in `src/middleware.ts`:

```tsx
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { auth } from '@/auth';
import {
  AUTH_ROUTES,
  DEFAULT_LOGIN_REDIRECT,
  PROTECTED_ROUTES,
  LOGIN_URL,
} from '@/lib/authRoutes';

const { auth } = NextAuth(authConfig);

export default async function middleware(request: NextRequest) {
  try {
    const session = await auth();
    const { pathname } = request.nextUrl;
    const isProtected = PROTECTED_ROUTES.some((route) =>
      pathname.startsWith(route)
    );
    const isLoggedIn = !!session;
    if (isProtected && !session) {
      return NextResponse.redirect(new URL(LOGIN_URL, request.url));
    }
    const isAuthRoute = AUTH_ROUTES.some((route) => pathname.startsWith(route));
    if (isAuthRoute && isLoggedIn) {
      return NextResponse.redirect(new URL(DEFAULT_LOGIN_REDIRECT, request.url));
    }
  } catch (error) {
    console.log('middleware error');
    console.log(error);
  }
  return NextResponse.next();
}

export const config = {
  matcher: [
    '/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)',
    '/(api|trpc)(.*)',
  ],
};
```

### Providers

- Separate providers in `src/auth.config.ts`:

```tsx
import GitHub from 'next-auth/providers/github';
import Credentials from 'next-auth/providers/credentials';
import type { NextAuthConfig } from 'next-auth';
import { LoginSchema } from './schemas';
import { getUserByEmail } from './data/user';
import bcrypt from 'bcryptjs';

const customAuthProvider = Credentials({
  async authorize(credentials) {
    try {
      const validatedFields = LoginSchema.safeParse(credentials);
      if (validatedFields.success) {
        const { email, password } = validatedFields.data;
        const user = await getUserByEmail(email);
        if (!user || !user.password) {
          return null;
        }
        const passwordMatch = await bcrypt.compare(password, user.password);
        if (passwordMatch) {
          return user;
        }
      }
    } catch (error) {
      console.log('error en credentials');
      console.log(error);
    }
    return null;
  },
});

export default {
  providers: [GitHub, customAuthProvider],
} satisfies NextAuthConfig;
```

- Update `src/auth.ts`:

```tsx
import NextAuth from 'next-auth';
import { PrismaAdapter } from '@auth/prisma-adapter';
import { prisma } from '@/lib/db';
import authConfig from '@/auth.config';

export const { handlers, signIn, signOut, auth } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: 'jwt' },
  ...authConfig,
});
```

### Login and Register Forms

- Register form (`register-form.tsx`):

```tsx
'use client';

import React, { startTransition, useState } from 'react';
import CardWrapper from './card-wrapper';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import * as z from 'zod';
import { RegisterSchema } from '@/schemas';
import { Input } from '../ui/input';
import { Button } from '../ui/button';
import FormError from './form-error';
import FormSuccess from './form-success';
import { register } from '@/actions/register';

type TypeSchemaForm = z.infer<typeof RegisterSchema>;

const RegisterForm = () => {
  const [error, setError] = useState<string | undefined>('');
  const [success, setSuccess] = useState<string | undefined>('');
  const form = useForm<TypeSchemaForm>({
    resolver: zodResolver(RegisterSchema),
    defaultValues: {
      email: '',
      password: '',
      name: '',
    },
  });

  const handlerSubmit = (values: TypeSchemaForm) => {
    setError('');
    setSuccess('');
    startTransition(() => {
      register(values).then((data) => {
        setError(data.error);
        setSuccess(data.success);
      });
    });
  };

  return (
    <CardWrapper
      headerLabel="Create an account"
      backButtonLabel="Already have an account?"
      backButtonHref="/auth/login"
      showSocial
    >
      <Form {...form}>
        <form onSubmit={form.handleSubmit(handlerSubmit)} className="space-y-6">
          <div className="space-y-6">
            <FormField
              control={form.control}
              name="name"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Name</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="Your Name" />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <FormField
              control={form.control}
              name="email"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Email</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="example@exam.com" type="email" />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <FormField
              control={form.control}
              name="password"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Password</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="*******" type="password" />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
          </div>
          <FormError message={error ?? ''} />
          <FormSuccess message={success ?? ''} />
          <Button type="submit" className="w-full">
            Register
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};

export default RegisterForm;
```

- Login form (`login-form.tsx`):

```tsx
'use client';

import React, { startTransition, useState } from 'react';
import CardWrapper from './card-wrapper';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import * as z from 'zod';
import { LoginSchema } from '@/schemas';
import { Input } from '../ui/input';
import { Button } from '../ui/button';
import FormError from './form-error';
import FormSuccess from './form-success';
import { login } from '@/actions/login';

type TypeSchemaForm = z.infer<typeof LoginSchema>;

const LoginForm = () => {
  const [error, setError] = useState<string | undefined>('');
  const [success, setSuccess] = useState<string | undefined>('');
  const form = useForm<TypeSchemaForm>({
    resolver: zodResolver(LoginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  const handlerSubmit = (values: TypeSchemaForm) => {
    setError('');
    setSuccess('');
    login(values).then((data) => {
      setError(data.error);
      setSuccess(data.success);
    });
  };

  return (
    <CardWrapper
      headerLabel="Welcome back"
      backButtonLabel="Don't have an account?"
      backButtonHref="/auth/register"
      showSocial
    >
      <Form {...form}>
        <form onSubmit={form.handleSubmit(handlerSubmit)} className="space-y-6">
          <div className="space-y-6">
            <FormField
              control={form.control}
              name="email"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Email</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="example@exam.com" type="email" />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
            <FormField
              control={form.control}
              name="password"
              render={({ field }) => (
                <FormItem>
                  <FormLabel>Password</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="*******" type="password" />
                  </FormControl>
                  <FormMessage />
                </FormItem>
              )}
            />
          </div>
          <FormError message={error ?? ''} />
          <FormSuccess message={success ?? ''} />
          <Button type="submit" className="w-full">
            Login
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};

export default LoginForm;
```

### Signout Button

```tsx
import { signOut } from '@/auth';
import React from 'react';
import { Button } from '../ui/button';

const SignoutButton = () => {
  return (
    <form
      action={async () => {
        'use server';
        await signOut();
      }}
    >
      <Button type="submit">Sign out</Button>
    </form>
  );
};

export default SignoutButton;
```

### Password Encryption

- Install bcrypt:

```bash
pnpm install bcryptjs
pnpm install -D @types/bcryptjs
```

- Hash and compare passwords:

```tsx
import bcrypt from 'bcrypt';

const hashedPassword = await bcrypt.hash(password, 10);
const passwordMatch = await bcrypt.compare(password, user.password);
```

## UI Components with Shadcn

- Initialize Shadcn:

```bash
pnpm dlx shadcn@latest init
```

- Install components:

```bash
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add dropdown-menu
pnpm dlx shadcn@latest add avatar
pnpm dlx shadcn@latest add badge
pnpm dlx shadcn@latest add sonner
pnpm dlx shadcn@latest add switch
pnpm dlx shadcn@latest add select
pnpm dlx shadcn@latest add dialog
```

- Utility function in `src/lib/utils.ts`:

```tsx
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

- Use components with `size` (`sm`, `lg`) and `variant` (`default`, `destructive`, `outline`, `secondary`, `ghost`, `link`).

## Email Sending with Resend

- Install:

```bash
pnpm add resend
```

- Send verification email:

```tsx
import { Resend } from 'resend';

const resend = new Resend(process.env.AUTH_RESEND_KEY);

export const sendVerificationEmail = async (email: string, token: string) => {
  const confirmLink = `http://localhost:3000/auth/new-verification?token=${token}`;
  return await resend.emails.send({
    from: 'onboarding@resend.dev',
    to: email,
    subject: 'Confirm your email',
    html: `<p>Click <a href="${confirmLink}">here</a> to confirm email</p>`,
  });
};
```

## Metadata

### Static Metadata

- Define in `app/layout.tsx`:

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};
```

### Dynamic Metadata

- Use `generateMetadata` in a layout or page:

```tsx
import { Metadata } from 'next';

type Props = {
  params: { productId: string };
};

export const generateMetadata = ({ params }: Props): Metadata => {
  return {
    title: `Producto ${params.productId}`,
  };
};

export default function Page({ params }: Props) {
  return (
    <>
      <p>El id es: {params.productId}</p>
    </>
  );
}
```

### Templates for Titles

- Define a title template in a parent layout:

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'Por defecto',
    template: '%s Resto del titulo',
  },
  description: 'Generated by create next app',
};
```

- Child layout or page:

```tsx
export const metadata: Metadata = {
  title: 'Create Next App',
  description: 'Generated by create next app',
};
```

- Resulting title: `Create Next App Resto del titulo`.

- Use `absolute` to override the template:

```tsx
export const metadata: Metadata = {
  title: {
    absolute: 'Este es todo el titulo',
  },
  description: 'Generated by create next app',
};
```

## API Routes

- Create API routes in `/app/api/ruta/route.ts`:

```tsx
export async function GET() {
  return new Response('hola mundo');
}
```

- Example with JSON response:

```tsx
import { products } from './data';

export async function GET() {
  return Response.json(products);
}

export function DELETE() {
  return NextResponse.json({
    message: 'deleting single note',
  });
}

export function PUT() {
  return NextResponse.json({
    message: 'updating single note',
  });
}

export async function POST(request: Request) {
  const product = await request.json();
  products.push(product);
  return Response.json(products, {
    status: 201,
    headers: {
      'Content-Type': 'application/json',
    },
  });
}
```

## Miscellaneous

### Debouncing

- Install `use-debounce`:

```bash
pnpm install use-debounce
```

- Use in a component:

```tsx
import { useDebounceCallback } from 'use-debounce';

const handleSearch = useDebounceCallback((term: string) => {
  // Search logic
}, 300);
```

### Revalidate and Redirect

- Revalidate a path:

```tsx
import { revalidatePath } from 'next/cache';

revalidatePath('/rutaform');
```

- Redirect:

```tsx
import { redirect } from 'next/navigation';

redirect('/rutaform');
```

### Custom 404 Page

- Create `app/not-found.tsx`:

```tsx
export default async function NotFound() {
  return <>No lo encuentro</>;
}
```

### useTransition

- Use `useTransition` for non-urgent state updates:

```tsx
import { useState, useTransition } from 'react';

export default function SearchPage() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  function handleSearch(e) {
    const newQuery = e.target.value;
    setQuery(newQuery);
    startTransition(() => {
      const filtered = searchData(newQuery);
      setResults(filtered);
    });
  }

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={handleSearch}
        placeholder="Buscar..."
      />
      {isPending ? (
        <div>Cargando...</div>
      ) : (
        <ul>
          {results.map((result) => (
            <li key={result.id}>{result.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
}

function searchData(query) {
  const start = Date.now();
  while (Date.now() - start < 500) {}
  return [{ id: 1, name: `Resultado para "${query}"` }];
}
```