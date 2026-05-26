---
title: TypeScript Nâng Cao
type: concept
tags: [typescript, nextjs, advanced, utility-types, generics, react]
updated: 2026-05-26
sources: [user-request-2026-05-26]
---

# TypeScript Nâng Cao

Tài liệu này bao gồm các kỹ thuật TypeScript nâng cao, đặc biệt hữu ích khi làm việc với **Next.js** và **React**.

---

## 1. Utility Types

TypeScript có sẵn các kiểu tiện ích để biến đổi types hiện có.

### `Partial<T>` — tất cả fields thành optional
```typescript
type User = {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user";
};

type UserUpdate = Partial<User>;
// Tương đương: { id?: number; name?: string; email?: string; role?: ... }

// Thực tế: PATCH endpoint — chỉ update những field được gửi lên
async function updateUser(id: number, updates: Partial<User>): Promise<User> {
  const res = await fetch(`/api/users/${id}`, {
    method: "PATCH",
    body: JSON.stringify(updates),
  });
  return res.json();
}

await updateUser(1, { name: "Bichi Updated" }); // Chỉ update name, không cần gửi cả object
```

### `Required<T>` — tất cả fields thành required
```typescript
type Config = {
  apiUrl?: string;
  timeout?: number;
  retries?: number;
};

type StrictConfig = Required<Config>;
// { apiUrl: string; timeout: number; retries: number }
```

### `Readonly<T>` — không cho phép thay đổi sau khi tạo
```typescript
type ImmutableUser = Readonly<User>;

const user: ImmutableUser = { id: 1, name: "Bichi", email: "b@b.com", role: "user" };
// user.name = "other"; // ❌ Error: Cannot assign to 'name' — readonly
```

### `Pick<T, Keys>` — chọn một số fields
```typescript
type UserPreview = Pick<User, "id" | "name">;
// { id: number; name: string }

// Thực tế: Hiển thị list user — không cần email, role
function UserCard({ user }: { user: UserPreview }) {
  return <div>{user.name}</div>;
}
```

### `Omit<T, Keys>` — bỏ một số fields
```typescript
type CreateUserDto = Omit<User, "id">;
// { name: string; email: string; role: "admin" | "user" }

// Thực tế: Tạo user mới — id do server tự sinh
async function createUser(data: CreateUserDto): Promise<User> {
  const res = await fetch("/api/users", {
    method: "POST",
    body: JSON.stringify(data),
  });
  return res.json();
}
```

### `Record<Keys, Type>` — object với keys và value types cố định
```typescript
type Role = "admin" | "user" | "guest";
type Permissions = Record<Role, string[]>;

const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};

// Thực tế: Map lỗi validation theo field
type ValidationErrors = Record<string, string>;
const errors: ValidationErrors = {
  email: "Email không hợp lệ",
  password: "Mật khẩu phải có ít nhất 8 ký tự",
};
```

### `Exclude<T, U>` và `Extract<T, U>`
```typescript
type Status = "pending" | "active" | "inactive" | "deleted";

type ActiveStatuses = Exclude<Status, "deleted" | "inactive">;
// "pending" | "active"

type DangerousStatuses = Extract<Status, "deleted" | "inactive">;
// "deleted" | "inactive"
```

### `NonNullable<T>` — loại bỏ null và undefined
```typescript
type MaybeUser = User | null | undefined;
type DefiniteUser = NonNullable<MaybeUser>; // User
```

### `ReturnType<T>` — lấy kiểu trả về của function
```typescript
function getUser() {
  return { id: 1, name: "Bichi", email: "b@b.com" };
}

type UserFromFn = ReturnType<typeof getUser>;
// { id: number; name: string; email: string }
```

---

## 2. Conditional Types

Type thay đổi tuỳ theo điều kiện — như ternary nhưng cho types.

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>; // "yes"
type B = IsString<number>; // "no"

// Thực tế: Unwrap Promise
type Awaited<T> = T extends Promise<infer U> ? U : T;

type UserData = Awaited<Promise<User>>; // User
type StringData = Awaited<string>;      // string (không phải Promise, giữ nguyên)
```

### `infer` — suy luận kiểu trong conditional types
```typescript
// Lấy element type từ array
type ElementType<T> = T extends (infer U)[] ? U : never;

type StrElement = ElementType<string[]>; // string
type NumElement = ElementType<number[]>; // number

// Lấy parameter types của function
type FirstParam<T extends (...args: any) => any> =
  T extends (first: infer P, ...rest: any) => any ? P : never;

function login(email: string, password: string) {}
type EmailType = FirstParam<typeof login>; // string
```

---

## 3. Mapped Types

Biến đổi tất cả properties của một type.

```typescript
// Làm tất cả properties thành nullable
type Nullable<T> = { [K in keyof T]: T[K] | null };

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; ... }

// Làm tất cả properties thành getters
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<{ name: string; age: number }>;
// { getName: () => string; getAge: () => number }
```

**Ví dụ thực tế:** Form validation state
```typescript
type FormErrors<T> = {
  [K in keyof T]?: string; // Mỗi field có thể có error message
};

type LoginForm = {
  email: string;
  password: string;
};

type LoginErrors = FormErrors<LoginForm>;
// { email?: string; password?: string }

const errors: LoginErrors = {
  email: "Email không hợp lệ",
};
```

---

## 4. Template Literal Types

Tạo string types từ template.

```typescript
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

type CSSProperty = "margin" | "padding";
type CSSDirection = "Top" | "Right" | "Bottom" | "Left";
type CSSClass = `${CSSProperty}${CSSDirection}`;
// "marginTop" | "marginRight" | ... | "paddingLeft"

// Thực tế: API endpoint builder
type HttpMethod = "get" | "post" | "put" | "delete";
type ApiAction = `${HttpMethod}${Capitalize<string>}`;

// Strongly typed event emitter
type EventMap = {
  userCreated: { userId: number };
  userDeleted: { userId: number };
  orderPlaced: { orderId: number; total: number };
};

type EventKey = keyof EventMap;
// "userCreated" | "userDeleted" | "orderPlaced"
```

---

## 5. Generics Nâng Cao

### Constraints với `extends`
```typescript
// T phải có property id
function findById<T extends { id: number }>(items: T[], id: number): T | undefined {
  return items.find((item) => item.id === id);
}

findById([{ id: 1, name: "Bichi" }], 1); // OK
findById(["a", "b"], 1); // ❌ Error: string không có id
```

### Multiple Generics
```typescript
function mergeObjects<T extends object, U extends object>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = mergeObjects({ name: "Bichi" }, { age: 25 });
// result: { name: string; age: number }
```

### Generic Defaults
```typescript
interface PaginatedResponse<T = unknown> {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
}

// Không cần chỉ định type — dùng default
type DefaultResponse = PaginatedResponse;         // T = unknown
type UserPage = PaginatedResponse<User>;           // T = User
type ProductPage = PaginatedResponse<Product>;    // T = Product
```

**Ví dụ thực tế:** Generic hook cho data fetching
```typescript
import { useState, useEffect } from "react";

interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((data: T) => setState({ data, loading: false, error: null }))
      .catch((err) => setState({ data: null, loading: false, error: err.message }));
  }, [url]);

  return state;
}

// Sử dụng — tự động type-safe
const { data: users, loading } = useFetch<User[]>("/api/users");
const { data: product } = useFetch<Product>("/api/products/1");
```

---

## 6. TypeScript với React & Next.js

### Props typing
```typescript
// Cách 1: Interface (khuyến nghị cho props)
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary" | "danger";
  disabled?: boolean;
  children?: React.ReactNode;
}

// Cách 2: React.FC (không cần thiết, dùng trực tiếp)
function Button({ label, onClick, variant = "primary", disabled = false }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
    >
      {label}
    </button>
  );
}
```

### Event handlers
```typescript
// Input change
function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
  console.log(e.target.value);
}

// Form submit
function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
  e.preventDefault();
}

// Button click
function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
  console.log("clicked");
}
```

### useState với TypeScript
```typescript
// Infer từ giá trị khởi tạo
const [count, setCount] = useState(0);           // number
const [name, setName] = useState("");             // string

// Chỉ định rõ khi type phức tạp
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Product[]>([]);
const [status, setStatus] = useState<"idle" | "loading" | "error">("idle");
```

### Next.js App Router types
```typescript
// Page component
type PageProps = {
  params: { slug: string };
  searchParams: { [key: string]: string | string[] | undefined };
};

export default function ProductPage({ params, searchParams }: PageProps) {
  return <div>Product: {params.slug}</div>;
}

// Server Action
"use server";
async function createOrder(formData: FormData): Promise<{ success: boolean; orderId?: number }> {
  const productId = formData.get("productId") as string;
  // ...
  return { success: true, orderId: 123 };
}

// Route Handler
import { NextRequest, NextResponse } from "next/server";

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await getUser(Number(params.id));
  return NextResponse.json(user);
}
```

---

## 7. Declaration Merging & Module Augmentation

```typescript
// Mở rộng type của thư viện bên thứ 3
// File: types/next-auth.d.ts
import "next-auth";

declare module "next-auth" {
  interface Session {
    user: {
      id: string;
      role: "admin" | "user";
      name?: string | null;
      email?: string | null;
    };
  }
}

// Mở rộng global types
declare global {
  interface Window {
    analytics: {
      track: (event: string, properties?: object) => void;
    };
  }
}
```

---

## 8. Discriminated Unions (Pattern quan trọng)

Khi kết hợp với switch/if, TS tự thu hẹp type chính xác.

```typescript
type LoadingState = { status: "loading" };
type SuccessState<T> = { status: "success"; data: T };
type ErrorState = { status: "error"; message: string };

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// Component render theo state
function UserProfile({ state }: { state: AsyncState<User> }) {
  switch (state.status) {
    case "loading":
      return <Spinner />;
    case "error":
      return <ErrorMessage message={state.message} />; // TS biết có .message
    case "success":
      return <Profile user={state.data} />;          // TS biết có .data
  }
}
```

---

## 9. Decorators (TypeScript 5+)

```typescript
// Decorator cho class
function Singleton<T extends { new (...args: any[]): {} }>(constructor: T) {
  let instance: T;
  return class extends constructor {
    constructor(...args: any[]) {
      if (instance) return instance;
      super(...args);
      instance = this as unknown as T;
    }
  };
}

// Decorator cho method — logging
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with`, args);
    const result = original.apply(this, args);
    console.log(`${key} returned`, result);
    return result;
  };
  return descriptor;
}
```

---

## 10. Strict Mode & tsconfig Best Practices

```json
// tsconfig.json cho Next.js project
{
  "compilerOptions": {
    "strict": true,              // Bật tất cả strict checks
    "noImplicitAny": true,       // Không cho phép type any ngầm định
    "strictNullChecks": true,    // null/undefined phải được xử lý rõ ràng
    "noUnusedLocals": true,      // Cảnh báo biến không dùng
    "noUnusedParameters": true,  // Cảnh báo parameter không dùng
    "exactOptionalPropertyTypes": true, // Optional có nghĩa là absent, không phải undefined
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "module": "esnext",
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "paths": {
      "@/*": ["./*"]             // Alias import
    }
  }
}
```

---

## Cheatsheet Nhanh

| Utility Type | Tác dụng | Ví dụ |
|---|---|---|
| `Partial<T>` | Tất cả optional | Update payload |
| `Required<T>` | Tất cả required | Strict config |
| `Readonly<T>` | Không đổi được | Immutable state |
| `Pick<T, K>` | Chọn fields | Preview card |
| `Omit<T, K>` | Bỏ fields | DTO tạo mới |
| `Record<K, V>` | Object map | Permissions map |
| `Exclude<T, U>` | Bỏ khỏi union | Filter status |
| `Extract<T, U>` | Giữ từ union | Subset types |
| `NonNullable<T>` | Bỏ null/undefined | Safe value |
| `ReturnType<F>` | Kiểu trả về | Infer từ function |

---

## Xem thêm

- [TypeScript Cơ Bản](./typescript-basics.md)
- [Next.js Server vs Client Components](./nextjs-server-vs-client-components.md)
- [Next.js Route Handlers & Server Actions](../playbooks/nextjs/nextjs-route-handlers-and-server-actions.md)
