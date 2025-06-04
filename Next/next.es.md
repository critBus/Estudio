# Guía de Next.js en Español

Esta guía ofrece una visión completa sobre el uso de Next.js, abordando la configuración del proyecto, enrutamiento, estilos, obtención de datos, integración con bases de datos, pruebas, internacionalización, autenticación y más.

## Configuración del Proyecto

Para iniciar un nuevo proyecto de Next.js, utiliza uno de los siguientes comandos:

```bash
npm create next-app@latest <nombre-proyecto>
npx create-next-app@latest
pnpm create next-app <nombre-proyecto>
```

## Enrutamiento

### Directorio de Páginas (`/pages`)

- El directorio `/pages` se utiliza para el enrutamiento en Next.js.
- Evita usar guiones (`-`) en los nombres de las rutas.

#### Ejemplo: Rutas Estáticas

- Un archivo en `/pages/MiRuta/index.jsx` crea una ruta en `/MiRuta`.
- El archivo `index.jsx` define el contenido de esa ruta.

```tsx
export default function Index() {
  return (
    <>
      Hello
    </>
  );
}
```

### Directorio de Aplicación (`/app`)

- El directorio `/app` se usa para el enrutador de aplicaciones (App Router) en Next.js.
- El archivo `page.tsx` (o `.jsx`) en `/app` es la vista raíz del proyecto.
- Cada carpeta con un archivo `page.tsx` o `page.jsx` crea una nueva ruta.
- El componente en `page.tsx` suele llamarse `Page`, aunque no es obligatorio.

```tsx
export default function Page() {
  return (
    <>
      <h2>Mi ruta1</h2>
    </>
  );
}
```

### Diseños (`/app/layout.tsx`)

- El archivo `layout.tsx` define un componente de diseño que envuelve a sus hijos según la estructura de carpetas.

```tsx
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <>
      Mi Layout
      {children}
      Fin Mi Layout
    </>
  );
}
```

### Rutas Dinámicas

- Las rutas dinámicas se crean usando corchetes para parámetros, por ejemplo, `/pages/ruta/[parametro].jsx`.
- Los parámetros se acceden mediante la prop `params` en el componente `Page`.

```tsx
export default function Page({ params }: { params: { productId: string } }) {
  return (
    <>
      <p>El id es: {params.productId}</p>
    </>
  );
}
```

### Agrupación de Rutas

- Las carpetas con nombres entre paréntesis, como `/app/(agrupada)/ruta1`, se ignoran en la URL, pero sus rutas hijas no.

- Ejemplo de estructura:
  
  ```
  /app/(agrupada)/ruta1
  /app/(agrupada)/ruta2
  /app/ruta3
  ```

- URLs resultantes:
  
  - `/ruta1`
  - `/ruta2`
  - `/ruta3`

### Ocultar Rutas

- Prefija el nombre de una carpeta con un guion bajo (`_`) para ocultarla del enrutamiento, por ejemplo, `/app/_oculta`.

### Rutas Paralelas (`@carpeta`)

- Las rutas paralelas permiten renderizar múltiples componentes en la misma ruta.

- Ejemplo de estructura:
  
  ```
  /ruta/layout.tsx
  /ruta/@chat/page.tsx
  /ruta/@video/page.tsx
  /ruta/page.tsx
  ```

- El archivo `layout.tsx` combina las rutas paralelas:

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
      Fin Mi Layout Complejo
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
  return <>complejo</>;
}
```

## Archivos Estáticos

- Coloca los archivos estáticos (como imágenes) en el directorio `/public`.
- La carpeta `/public` se considera la raíz, por lo que no se incluye en las rutas.

### Uso de `next/link`

```tsx
import Link from 'next/link';

<Link href="/ruta/mivista">Algo</Link>
```

### Uso de `next/image`

```tsx
import Image from 'next/image';

<Image src="/image.jpg" width={600} height={600} alt="descripción" />
```

## Estilos

### Módulos CSS

- Define los estilos en `/styles/miestilos.css`.
- Usa nombres de clases, no etiquetas HTML, para los estilos.

```css
.container {
  max-width: 36rem;
  font-size: large;
  color: blue;
}
```

- En el componente:

```tsx
import styles from '../styles/Layout.module.css';

<div className={styles.container}>Contenido</div>
```

### Estilos Globales

- Importa estilos globales en un diseño:

```tsx
import './globals.css';
```

### Fuentes de Google

- Define fuentes en `/app/ui/fonts.tsx`:

```tsx
import { Inter, Montserrat } from 'next/font/google';

export const montserrat = Montserrat({ subsets: ['latin'] });
```

- Úsalas en un componente:

```tsx
import { montserrat } from './ui/fonts';

<h1 className={`${montserrat.className} antialiased`}>Hola</h1>
```

## Obtención de Datos

### `getStaticProps`

- Se usa para generación de sitios estáticos, se ejecuta en tiempo de compilación y no se incluye en el frontend.

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
      revalidate: 60, // 1 minuto
    };
  } catch (error) {
    console.error(error);
  }
}
```

### `getServerSideProps`

- Se usa para renderizado del lado del servidor, se ejecuta en cada solicitud.
- Debe exportarse como una función independiente en el directorio `/pages`.

```tsx
export async function getServerSideProps() {
  // Obtener datos
  return {
    props: {}, // Pasar datos a la página
  };
}
```

## Renderizado del Lado del Cliente

- Usa la directiva `'use client'` para renderizar componentes en el cliente.

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

## Estados de Carga

### Componente de Carga

- Crea un archivo `loading.tsx` al mismo nivel que `page.tsx` para mostrar mientras la página carga.

```tsx
export default function Loading() {
  return <>Cargando...</>;
}
```

### Suspense

- Usa `Suspense` de React para componentes que cargan lentamente, proporcionando una interfaz de respaldo.

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

- Para actualizaciones dinámicas, agrega una `key` a `Suspense`:

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
      Búsqueda: <Search placeholder="término" />
      <p>
        <span>Página:</span>
        {currentPage}
      </p>
      <p>
        <span>Consulta:</span>
        {query}
      </p>
      <Suspense key={query + currentPage} fallback={<>Espere...</>}>
        <ComponenteLento />
      </Suspense>
    </>
  );
}
```

## Acciones del Servidor

- Define acciones del servidor en `/app/lib/actions.ts` con la directiva `'use server'`.

```tsx
'use server';

export async function createInvoice(formData: FormData) {
  console.log(formData);
}
```

- Úsalas en un formulario:

```tsx
import { createInvoice } from '../actions';

export default function FormExample() {
  return (
    <div className="form" style={{ color: 'red' }}>
      <form action={createInvoice}>
        <label>Nombre:</label>
        <input type="text" name="name" />
        <label>Edad:</label>
        <input type="number" name="age" />
        <button type="submit">Enviar</button>
      </form>
    </div>
  );
}
```

## Validación con Zod

- Instala Zod:

```bash
pnpm install zod
```

- Define esquemas:

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

## Integración con Bases de Datos

### Postgres con Vercel

- Instala:

```bash
pnpm install @vercel/postgres
```

### Prisma

- Instala Prisma y dependencias relacionadas:

```bash
pnpm install prisma -D
pnpm install @prisma/client
pnpm install tsx --save-dev
```

- Inicializa Prisma:

```bash
npx prisma init
# Para SQLite
npx prisma init --datasource-provider sqlite
```

- Configura `.env`:

sqlite

```env
DATABASE_URL="file:./dev.db"
```

postgrest

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/prueba_auth?schema=public"
```

- Define modelos en `/prisma/schema.prisma`:

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

- Ejecuta migraciones:

```bash
npx prisma migrate dev --name init
```

- Genera el cliente si es necesario:

```bash
pnpm prisma generate
```

- Siembra la base de datos:

```bash
npx prisma db seed
```

- Ejemplo de script de siembra (`/prisma/seed.ts`):

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
    // Categorías adicionales...
  ]);

  console.log('Siembra completada exitosamente!');
  console.log('Categorías creadas:', categories.length);
}

main()
  .catch((e) => {
    console.error('Error durante la siembra:', e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

- Configura el cliente de Prisma en `/src/libs/prisma.ts`:

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

- Ejemplo de ruta API con Prisma:

```tsx
import { prisma } from '@/libs/prisma';

export async function GET() {
  const notes = await prisma.note.findMany();
  return NextResponse.json(notes);
}
```

### MongoDB con Mongoose

- Instala:

```bash
pnpm install mongodb mongoose
```

- Configura `.env`:

```env
MONGODB_URI="mongodb://localhost:27017"
```

- Configura la conexión en `/src/lib/mongoose.ts`:

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

- Define un esquema:

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

## Pruebas

### Jest: Configuración de Bibliotecas de Pruebas

- Instala dependencias:

```bash
pnpm install --save-dev @testing-library/react @testing-library/dom @types/react @types/react-dom
pnpm install -D jest jest-environment-jsdom @testing-library/jest-dom ts-node @types/jest
pnpm install ts-jest
pnpm install eslint-plugin-jest-dom eslint-plugin-testing-library
```

- Configura ESLint en `eslint.config.mjs`:

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

- Configura Jest en `jest.config.ts`:

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

- Crea `jest.setup.ts`:

```typescript
import '@testing-library/jest-dom';
```

- Agrega scripts a `package.json`:

```json
{
  "scripts": {
    "test": "jest --passWithNoTests",
    "test:watch": "jest --watchAll"
  }
}
```



- Ejecuta pruebas:

```bash
pnpm test
```

## Secuencialmente

Si estas usando prisma lo mejor es que sea en secuencia y con algun metodo que limpie los datos usando el argumento `--runInBand`

```bash
npx jest --runInBand
npx jest NombreTest.test.tsx
```

O agergandolo en el script 

```json
{
  "scripts": {
    "test": "jest --passWithNoTests --runInBand",
    "test:watch": "jest --watchAll --runInBand"
  }
}
```

### Escribiendo Pruebas

- Coloca las pruebas en `src/__tests__` o junto a los componentes como `ComponentName.test.tsx`.
- Ejemplo de prueba para una página de inicio:

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

- Prueba con `beforeEach`:

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

- Prueba por rol:

```tsx
expect(
  screen.getByRole('heading', {
    name: /Título/i,
  })
).toBeInTheDocument();
```

- Prueba con `data-testid`:

```tsx
<span data-testid="idtexto">El texto</span>

it('Título id test', () => {
  render(<HomePage />);
  expect(screen.getByTestId('idtexto')).toHaveTextContent(/texto/);
});
```

- Prueba cambios en un input:

```tsx
import { fireEvent, render, screen } from '@testing-library/react';

it('escribir texto habilita el botón', () => {
  render(<HomePage />);
  const inputText = screen.getByPlaceholderText('escriba algo');
  fireEvent.change(inputText, { target: { value: 'nuevo texto' } });
  const boton = screen.getByRole('button');
  expect(boton).toBeEnabled();
});
```

- Prueba con `within`:

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

### Simulaciones (Mocking)

- Simula una función con `jest.fn()`:

```tsx
const mock = jest.fn();
mock('hola', { id: 1 });
expect(mock.calls).toEqual([['hola', { id: 1 }]]);
```

- Simula `useRouter`:

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { useRouter } from 'next/navigation';
import RedirectPage from '@/app/redirect/page';

jest.mock('next/navigation', () => ({ useRouter: jest.fn() }));

describe('Redirect page', () => {
  it('probar redirección', async () => {
    const mockPush = jest.fn();
    useRouter.mockReturnValue({ push: mockPush });
    render(<RedirectPage />);
    const boton = screen.getByRole('button');
    fireEvent.click(boton);
    expect(mockPush).toHaveBeenCalledWith('other');
  });
});
```

## Internacionalización (i18n)

### Usando `next-i18n-router`

- Instala:

```bash
pnpm install next-i18n-router i18next --save
```

- Configura `i18nConfig.ts`:

```typescript
const i18nConfig = {
  locales: ['en', 'es'],
  defaultLocale: 'es',
};

export default i18nConfig;
```

- Configura el middleware en `src/middleware.ts`:

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

- Mueve las rutas a `src/app/[locale]`:

```
└── app
    └── [locale]
        ├── layout.tsx
        └── page.tsx
```

### Usando `next-intl`

- Instala:

```bash
pnpm install next-intl
```

- Estructura del proyecto:

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

- Configura `next.config.ts`:

```typescript
import type { NextConfig } from 'next';
import createNextIntlPlugin from 'next-intl/plugin';

const nextConfig: NextConfig = {};
const withNextIntl = createNextIntlPlugin();
export default withNextIntl(nextConfig);
```

- Define traducciones en `messages/es.json`:

```json
{
  "HomePage": {
    "title": "¡Hola mundo!",
    "about": "Ir a la página de acerca"
  }
}
```

- Configura el enrutamiento en `src/i18n/routing.ts`:

```typescript
import { defineRouting } from 'next-intl/routing';

export const routing = defineRouting({
  locales: ['en', 'es'],
  defaultLocale: 'es',
});
```

- Configura la navegación en `src/i18n/navigation.ts`:

```typescript
import { createNavigation } from 'next-intl/navigation';
import { routing } from './routing';

export const { Link, redirect, usePathname, useRouter, getPathname } =
  createNavigation(routing);
```

- Configura la solicitud en `src/i18n/request.ts`:

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

- Actualiza el middleware en `src/middleware.ts`:

```typescript
import createMiddleware from 'next-intl/middleware';
import { routing } from './i18n/routing';

export default createMiddleware(routing);

export const config = {
  matcher: '/((?!api|trpc|_next|_vercel|.*\\..*).*)',
};
```

- Crea el diseño principal en `src/app/[locale]/layout.tsx`:

```tsx
import { NextIntlClientProvider, hasLocale } from 'next-intl';
import { notFound } from 'next/navigation';
import { routing } from '@/i18n/routing';
import '../globals.css';
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Crear Aplicación Next',
  description: 'Generado por create next app',
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

- Crea una página en `src/app/[locale]/page.tsx`:

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

- Selector de idioma en `LocaleSwitcher.tsx`:

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

## Autenticación con NextAuth

### Configuración

- Instala:

```bash
pnpm add next-auth@beta
npx auth secret
```

- Configura `src/auth.ts`:

```tsx
import NextAuth from 'next-auth';

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [],
});
```

- Configura la ruta API en `src/app/api/auth/[...nextauth]/route.ts`:

```tsx
import { handlers } from '@/auth';

export const { GET, POST } = handlers;
```

- Configura el middleware en `src/middleware.ts`:

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
    console.log('error en middleware');
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

### Proveedores

- Separa los proveedores en `src/auth.config.ts`:

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
      console.log('error en credenciales');
      console.log(error);
    }
    return null;
  },
});

export default {
  providers: [GitHub, customAuthProvider],
} satisfies NextAuthConfig;
```

- Actualiza `src/auth.ts`:

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

### Formularios de Inicio de Sesión y Registro

- Formulario de registro (`register-form.tsx`):

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
      headerLabel="Crear una cuenta"
      backButtonLabel="¿Ya tienes una cuenta?"
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
                  <FormLabel>Nombre</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="Tu Nombre" />
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
                  <FormLabel>Correo</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="ejemplo@exam.com" type="email" />
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
                  <FormLabel>Contraseña</FormLabel>
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
            Registrarse
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};

export default RegisterForm;
```

- Formulario de inicio de sesión (`login-form.tsx`):

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
      headerLabel="Bienvenido de vuelta"
      backButtonLabel="¿No tienes una cuenta?"
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
                  <FormLabel>Correo</FormLabel>
                  <FormControl>
                    <Input {...field} placeholder="ejemplo@exam.com" type="email" />
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
                  <FormLabel>Contraseña</FormLabel>
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
            Iniciar Sesión
          </Button>
        </form>
      </Form>
    </CardWrapper>
  );
};

export default LoginForm;
```

### Botón de Cierre de Sesión

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
      <Button type="submit">Cerrar Sesión</Button>
    </form>
  );
};

export default SignoutButton;
```

### Encriptación de Contraseñas

- Instala bcrypt:

```bash
pnpm install bcryptjs
pnpm install -D @types/bcryptjs
```

- Encripta y compara contraseñas:

```tsx
import bcrypt from 'bcrypt';

const hashedPassword = await bcrypt.hash(password, 10);
const passwordMatch = await bcrypt.compare(password, user.password);
```

## Componentes de Interfaz con Shadcn

- Inicializa Shadcn:

```bash
pnpm dlx shadcn@latest init
```

- Instala componentes:

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

- Función de utilidad en `src/lib/utils.ts`:

```tsx
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

- Usa componentes con `size` (`sm`, `lg`) y `variant` (`default`, `destructive`, `outline`, `secondary`, `ghost`, `link`).

## Envío de Correos con Resend

- Instala:

```bash
pnpm add resend
```

- Envía un correo de verificación:

```tsx
import { Resend } from 'resend';

const resend = new Resend(process.env.AUTH_RESEND_KEY);

export const sendVerificationEmail = async (email: string, token: string) => {
  const confirmLink = `http://localhost:3000/auth/new-verification?token=${token}`;
  return await resend.emails.send({
    from: 'onboarding@resend.dev',
    to: email,
    subject: 'Confirma tu correo',
    html: `<p>Haz clic <a href="${confirmLink}">aquí</a> para confirmar tu correo</p>`,
  });
};
```

## Envio de correos con nodemailer

```bash
pnpm install nodemailer
pnpm install @types/nodemailer
```

```javascript
import nodemailer from "nodemailer";
const transporter = nodemailer.createTransport({
  service: "Gmail",
  host: "smtp.gmail.com",
  port: 587,
  secure: false,
  auth: {
    user: process.env.GMAIL_USER,
    pass: process.env.GMAIL_PASS,
  },
}); 
const sendEmailFree = async ({
  to,
  subject,
  html,
}: {
  to: string;
  subject: string;
  html: string;
}) => {
  console.log({
    to,
    subject,
    html,
  });
  if (SENT_EMAIL) {
    return await transporter.sendMail({
      from: `"Example Team" <${process.env.GMAIL_USER}>`, // sender address
      to: to, // list of receivers
      subject: subject, // Subject line
      html: html, // html body
    });
  }
};
```

## Metadatos

### Metadatos Estáticos

- Define en `app/layout.tsx`:

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Crear Aplicación Next',
  description: 'Generado por create next app',
};
```

### Metadatos Dinámicos

- Usa `generateMetadata` en un diseño o página:

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

### Plantillas para Títulos

- Define una plantilla de título en un diseño padre:

```tsx
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    default: 'Por defecto',
    template: '%s Resto del título',
  },
  description: 'Generado por create next app',
};
```

- Diseño o página hija:

```tsx
export const metadata: Metadata = {
  title: 'Crear Aplicación Next',
  description: 'Generado por create next app',
};
```

- Título resultante: `Crear Aplicación Next Resto del título`.

- Usa `absolute` para anular la plantilla:

```tsx
export const metadata: Metadata = {
  title: {
    absolute: 'Este es todo el título',
  },
  description: 'Generado por create next app',
};
```

## Rutas API

- Crea rutas API en `/app/api/ruta/route.ts`:

```tsx
export async function GET() {
  return new Response('hola mundo');
}
```

- Ejemplo con respuesta JSON:

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

## Misceláneos

### Debouncing

- Instala `use-debounce`:

```bash
pnpm install use-debounce
```

- Úsalo en un componente:

```tsx
import { useDebounceCallback } from 'use-debounce';

const handleSearch = useDebounceCallback((term: string) => {
  // Lógica de búsqueda
}, 300);
```

### Revalidar y Redirigir

- Revalida una ruta:

```tsx
import { revalidatePath } from 'next/cache';

revalidatePath('/rutaform');
```

- Redirige:

```tsx
import { redirect } from 'next/navigation';

redirect('/rutaform');
```

### Página 404 Personalizada

- Crea `app/not-found.tsx`:

```tsx
export default async function NotFound() {
  return <>No lo encuentro</>;
}
```

### useTransition

- Usa `useTransition` para actualizaciones de estado no urgentes:

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