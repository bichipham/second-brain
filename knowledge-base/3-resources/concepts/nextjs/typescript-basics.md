---
title: TypeScript Cơ Bản
type: concept
tags: [typescript, nextjs, javascript, static-typing, beginner, api, payload]
updated: 2026-05-26
sources: [user-request-2026-05-26]
---

# TypeScript Cơ Bản

TypeScript là superset của JavaScript, thêm **static typing** vào JS. Mọi code JS hợp lệ đều là TS hợp lệ — TS chỉ thêm lớp kiểu trên cùng.

---

## 1. Tại sao dùng TypeScript?

- Phát hiện lỗi **lúc viết code** thay vì lúc chạy.
- IDE tự động gợi ý (autocomplete) chính xác hơn.
- Code tự document — đọc type là biết function nhận/trả gì.
- Bắt buộc trong hầu hết dự án Next.js production hiện nay.

---

## 2. Kiểu Nguyên Thuỷ (Primitive Types)

```typescript
let name: string = "Bichi";
let age: number = 25;
let isLoggedIn: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Đặc biệt: any (tắt type checking — tránh dùng)
let anything: any = "hello";
anything = 42; // OK nhưng không an toàn

// unknown (an toàn hơn any — phải kiểm tra type trước khi dùng)
let value: unknown = "hello";
if (typeof value === "string") {
  console.log(value.toUpperCase()); // OK
}
```

**Ví dụ thực tế:** Form đăng ký user
```typescript
let username: string = "";
let age: number = 0;
let acceptTerms: boolean = false;
```

---

## 3. Arrays & Tuples

```typescript
// Array
let fruits: string[] = ["apple", "banana", "orange"];
let scores: number[] = [90, 85, 78];

// Array với Generic syntax (tương đương)
let items: Array<string> = ["a", "b", "c"];

// Tuple — array với số lượng và kiểu cố định
let coordinate: [number, number] = [10.5, 106.7]; // [lat, lng]
let userInfo: [string, number, boolean] = ["Bichi", 25, true];
```

**Ví dụ thực tế:** API trả về danh sách sản phẩm
```typescript
let productNames: string[] = ["iPhone 15", "MacBook Pro", "AirPods"];
let prices: number[] = [999, 1999, 249];

// Tuple cho toạ độ store location
let hanoiStore: [number, number] = [21.0285, 105.8542];
```

---

## 4. Object Types

```typescript
// Inline object type
let user: { name: string; age: number; email: string } = {
  name: "Bichi",
  age: 25,
  email: "bichi@example.com",
};

// Optional property với ?
let product: { name: string; price: number; discount?: number } = {
  name: "iPhone",
  price: 999,
  // discount không bắt buộc
};
```

---

## 5. Type Alias

Đặt tên cho một kiểu để tái sử dụng.

```typescript
type User = {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user" | "guest"; // Union type
};

type Product = {
  id: number;
  name: string;
  price: number;
  discount?: number;
  inStock: boolean;
};

// Sử dụng
const currentUser: User = {
  id: 1,
  name: "Bichi",
  email: "ptbich308@gmail.com",
  role: "admin",
};
```

**Ví dụ thực tế:** E-commerce app
```typescript
type CartItem = {
  productId: number;
  name: string;
  price: number;
  quantity: number;
};

type Cart = {
  userId: number;
  items: CartItem[];
  totalPrice: number;
};

const myCart: Cart = {
  userId: 1,
  items: [
    { productId: 101, name: "iPhone 15", price: 999, quantity: 1 },
    { productId: 202, name: "AirPods", price: 249, quantity: 2 },
  ],
  totalPrice: 1497,
};
```

---

## 6. Interface

Giống `type` nhưng dành riêng cho **object shapes** và có thể **extend**.

```typescript
interface Animal {
  name: string;
  sound(): string;
}

interface Dog extends Animal {
  breed: string;
}

const myDog: Dog = {
  name: "Rex",
  breed: "Labrador",
  sound: () => "Woof!",
};
```

### Type vs Interface — khi nào dùng cái nào?

| Tình huống | Dùng |
|---|---|
| Object shape, có thể extend sau | `interface` |
| Union types, Intersection types | `type` |
| Function type | `type` |
| Primitive alias | `type` |
| Thư viện / public API | `interface` (extensible) |

---

## 7. Functions

```typescript
// Khai báo đầy đủ
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const multiply = (a: number, b: number): number => a * b;

// Optional parameter
function greet(name: string, greeting?: string): string {
  return `${greeting ?? "Hello"}, ${name}!`;
}

// Default parameter
function createUser(name: string, role: string = "user"): User {
  return { id: Date.now(), name, email: "", role: role as User["role"] };
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15
```

**Ví dụ thực tế:** Tính tổng tiền giỏ hàng
```typescript
type CartItem = { price: number; quantity: number };

function calculateTotal(items: CartItem[], discountPercent: number = 0): number {
  const subtotal = items.reduce((acc, item) => acc + item.price * item.quantity, 0);
  return subtotal * (1 - discountPercent / 100);
}

const total = calculateTotal(
  [{ price: 999, quantity: 1 }, { price: 249, quantity: 2 }],
  10 // 10% discount
);
console.log(total); // 1347.3
```

---

## 8. Union & Intersection Types

```typescript
// Union — một trong các kiểu
type Status = "pending" | "active" | "inactive";
type ID = string | number;

function printId(id: ID) {
  if (typeof id === "string") {
    console.log(id.toUpperCase());
  } else {
    console.log(id.toFixed(0));
  }
}

// Intersection — kết hợp các kiểu
type Timestamps = {
  createdAt: Date;
  updatedAt: Date;
};

type BaseUser = {
  id: number;
  name: string;
};

type UserWithTimestamps = BaseUser & Timestamps;
```

**Ví dụ thực tế:** API response status
```typescript
type ApiResponse<T> =
  | { status: "success"; data: T }
  | { status: "error"; message: string };

function handleResponse(res: ApiResponse<User>) {
  if (res.status === "success") {
    console.log("User:", res.data.name);
  } else {
    console.error("Error:", res.message);
  }
}
```

---

## 9. Enums

```typescript
// Numeric enum (mặc định)
enum Direction {
  Up,    // 0
  Down,  // 1
  Left,  // 2
  Right, // 3
}

// String enum (khuyến nghị — dễ debug hơn)
enum OrderStatus {
  Pending = "PENDING",
  Processing = "PROCESSING",
  Shipped = "SHIPPED",
  Delivered = "DELIVERED",
  Cancelled = "CANCELLED",
}

// Sử dụng
let order = { id: 1, status: OrderStatus.Pending };
if (order.status === OrderStatus.Pending) {
  console.log("Đơn hàng đang chờ xử lý");
}
```

---

## 10. Type Assertions & Type Guards

```typescript
// Type Assertion — nói với TS "tin tao, đây là kiểu này"
const input = document.getElementById("username") as HTMLInputElement;
console.log(input.value); // TS biết đây là HTMLInputElement

// Type Guard — kiểm tra runtime
function isString(value: unknown): value is string {
  return typeof value === "string";
}

// Narrowing với instanceof
function formatDate(date: string | Date): string {
  if (date instanceof Date) {
    return date.toLocaleDateString("vi-VN");
  }
  return new Date(date).toLocaleDateString("vi-VN");
}
```

---

## 11. Generics Cơ Bản

Viết code linh hoạt mà vẫn type-safe.

```typescript
// Function generic
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello"); // trả về string
identity<number>(42);      // trả về number

// Generic với Array
function getFirstItem<T>(arr: T[]): T | undefined {
  return arr[0];
}

const firstUser = getFirstItem<User>([{ id: 1, name: "Bichi", email: "", role: "user" }]);

// Generic Interface
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

type UsersResponse = ApiResponse<User[]>;
type ProductResponse = ApiResponse<Product>;
```

**Ví dụ thực tế:** Hàm fetch dùng chung
```typescript
async function fetchData<T>(url: string): Promise<ApiResponse<T>> {
  const res = await fetch(url);
  const data = await res.json();
  return { data, status: res.status, message: "OK" };
}

// Sử dụng
const users = await fetchData<User[]>("/api/users");
const product = await fetchData<Product>("/api/products/1");
// TS tự biết users.data là User[], product.data là Product
```

---

---

## 12. Define API Payload — Request & Response

### Pattern 1 — Đơn giản (cho từng endpoint)

```typescript
// Request body khi tạo user
type CreateUserRequest = {
  name: string;
  email: string;
  password: string;
  role?: "admin" | "user";
};

// Response trả về
type CreateUserResponse = {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user";
  createdAt: string;
};

// Dùng với fetch
async function createUser(body: CreateUserRequest): Promise<CreateUserResponse> {
  const res = await fetch("/api/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(body),
  });
  return res.json();
}
```

### Pattern 2 — Generic wrapper (khuyến nghị cho production)

API thường trả về cùng 1 cấu trúc, chỉ khác phần `data`:

```typescript
// Wrapper chung cho mọi response
type ApiResponse<T> = {
  success: boolean;
  data: T;
  message: string;
};

// Wrapper khi có lỗi
type ApiError = {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, string>; // validation errors theo field
  };
};

// Kết hợp thành union
type ApiResult<T> = ApiResponse<T> | ApiError;

// Generic fetch function — dùng lại cho mọi endpoint
async function apiFetch<T>(url: string, options?: RequestInit): Promise<ApiResult<T>> {
  const res = await fetch(url, options);
  return res.json();
}

// Gọi — TS tự biết data là kiểu tương ứng
const result = await apiFetch<User>("/api/users/1");

if (result.success) {
  console.log(result.data.name); // TS biết đây là User
} else {
  console.error(result.error.message);
}
```

### Pattern 3 — Tách Request/Response theo domain (dự án lớn)

```typescript
// types/api/user.ts

// --- Entity ---
export type User = {
  id: number;
  name: string;
  email: string;
  role: "admin" | "user";
  createdAt: string;
};

// --- Requests ---
export type CreateUserRequest = Omit<User, "id" | "createdAt"> & {
  password: string;
};

export type UpdateUserRequest = Partial<Pick<User, "name" | "email" | "role">>;

export type GetUsersRequest = {
  page?: number;
  pageSize?: number;
  search?: string;
  role?: User["role"];
};

// --- Responses ---
export type GetUsersResponse = {
  data: User[];
  total: number;
  page: number;
  pageSize: number;
};
```

Dùng trong Next.js Route Handler:

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";
import type { GetUsersRequest, GetUsersResponse } from "@/types/api/user";

export async function GET(req: NextRequest) {
  const { searchParams } = req.nextUrl;

  const query: GetUsersRequest = {
    page: Number(searchParams.get("page") ?? 1),
    pageSize: Number(searchParams.get("pageSize") ?? 10),
    search: searchParams.get("search") ?? undefined,
  };

  const result: GetUsersResponse = await getUsersFromDB(query);
  return NextResponse.json<ApiResponse<GetUsersResponse>>({
    success: true,
    data: result,
    message: "OK",
  });
}
```

### Khi nào dùng pattern nào?

| Tình huống | Pattern |
|---|---|
| Side project, app nhỏ | Pattern 1 — đơn giản, nhanh |
| Production app | Pattern 2 — generic wrapper |
| Team lớn, nhiều domain | Pattern 3 — tách file theo domain |

---

---

## 13. Checklist: Lỗi Hay Gặp Khi Dùng TypeScript Với API & Next.js

> **Rule quan trọng nhất:** _"TypeScript không validate runtime."_ — Chỉ cần nhớ câu này là tránh được cực nhiều bug API.

### 1. Không trust API response — dùng runtime validation

TypeScript chỉ check lúc compile, không validate runtime.

❌ Sai:
```typescript
const data: User = await res.json() // Nếu backend trả sai shape → vẫn crash runtime
```

✅ Nên dùng [Zod](https://zod.dev) để validate:
```typescript
import { z } from "zod"

const UserSchema = z.object({
  id: z.string(),
  name: z.string(),
})

// parse() → throw error nếu sai
const data = UserSchema.parse(await res.json())

// safeParse() → không throw, kiểm tra thủ công
const result = UserSchema.safeParse(json)
if (!result.success) return null
const user = result.data
```

---

### 2. Tách type Request / Response rõ ràng

❌ Sai — dùng 1 type cho mọi thứ:
```typescript
type User = { id: string; email: string; password: string }
```

✅ Nên tách:
```typescript
type CreateUserRequest = { email: string; password: string }
type UserResponse      = { id: string; email: string }
// request ≠ response, tránh leak password/internal fields ra ngoài
```

---

### 3. Next.js Route Handler — luôn type JSON body

❌ `body` là `any`:
```typescript
export async function POST(req: Request) {
  const body = await req.json() // any
}
```

✅ Tốt hơn — type thủ công:
```typescript
type CreatePostBody = { title: string; content: string }

export async function POST(req: Request) {
  const body: CreatePostBody = await req.json()
}
```

✅ Tốt nhất — kết hợp Zod:
```typescript
const BodySchema = z.object({ title: z.string(), content: z.string() })

export async function POST(req: Request) {
  const body = BodySchema.parse(await req.json())
}
```

---

### 4. Define API response format thống nhất

Đừng để mỗi route trả về kiểu khác nhau.

```typescript
type ApiResponse<T> = {
  success: boolean
  data?: T
  error?: string
}

return Response.json<ApiResponse<User>>({ success: true, data: user })
```

---

### 5. Không dùng `any` — dùng `unknown`

```typescript
// ❌
const data: any

// ✅ — sau đó validate trước khi dùng
const data: unknown
```

---

### 6. Fetch wrapper typed — đừng viết fetch raw khắp project

```typescript
async function apiFetch<T>(url: string, options?: RequestInit): Promise<T> {
  const res = await fetch(url, options)
  if (!res.ok) throw new Error("Request failed")
  return res.json()
}

// Dùng
const user = await apiFetch<User>("/api/user")
```

---

### 7. Cẩn thận với optional fields

```typescript
type User = { avatar?: string }

// ❌ crash nếu avatar undefined
user.avatar.toLowerCase()

// ✅
user.avatar?.toLowerCase()
if (user.avatar) { ... }
```

---

### 8. Enum API → ưu tiên union type

```typescript
// ❌ — emit JS dư, kém compatible với JSON
enum Status { ACTIVE = "ACTIVE" }

// ✅ — nhẹ hơn, JSON-friendly, không emit JS
type Status = "ACTIVE" | "INACTIVE"
```

---

### 9. Next.js: phân biệt Server vs Client types

`import fs from "fs"` chỉ dùng được server side. Import nhầm vào Client Component → build fail.

---

### 10. Đừng expose Prisma/database type trực tiếp ra API

```typescript
// ❌ — lộ fields, coupling DB schema ↔ API
type User = Prisma.User

// ✅ — map sang DTO
type UserDto = { id: string; name: string }
```

---

### 11. Route params trong Next.js — type rõ

```typescript
type Params = { params: { id: string } }

export async function GET(req: Request, { params }: Params) {
  const id = params.id
}
```

---

### 12. Query params — luôn parse

`searchParams.get()` luôn trả `string | null`.

```typescript
// ❌
const page = searchParams.get("page") // string | null

// ✅
const page = Number(searchParams.get("page") ?? 1)
```

---

### 13. Đừng over-type khi TS tự infer được

```typescript
// ❌ — thừa
const x: string = "hello"

// ✅ — TS tự infer
const x = "hello"
```

---

### 14. Shared types giữa frontend/backend (monorepo)

```
/packages/types   ← request types, response types, zod schemas
```

Frontend + backend import chung — đảm bảo đồng bộ.

---

### 15. Next.js App Router → ưu tiên Server Actions cho internal mutations

```typescript
// Thay vì: Client → fetch("/api/...")
// Dùng:    Client → server action
// Lợi ích: type-safe hơn, ít boilerplate, không cần serialize API nội bộ
```

---

### Pattern đầy đủ cho Route Handler

```typescript
import { z } from "zod"

const BodySchema = z.object({ title: z.string() })

export async function POST(req: Request) {
  try {
    const body = BodySchema.parse(await req.json())
    return Response.json({ success: true, data: body })
  } catch {
    return Response.json({ success: false, error: "Invalid request" }, { status: 400 })
  }
}
```

---

### Stack recommend cho TypeScript API

- [Next.js](https://nextjs.org) — framework
- [Zod](https://zod.dev) — runtime validation
- [tRPC](https://trpc.io) hoặc REST — API layer
- [TanStack Query](https://tanstack.com/query/latest) — data fetching
- [Prisma](https://www.prisma.io) — ORM

---

## Xem thêm

- [TypeScript Nâng Cao](./typescript-advanced.md)
- [Next.js Overview](./nextjs-overview.md)
