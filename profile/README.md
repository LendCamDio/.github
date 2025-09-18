# 📑 Table of Contents
- [🖥️ Frontend (React + TypeScript)](#️-frontend-react--typescript)
  - [1. Cấu trúc dự án](#1-cấu-trúc-dự-án)
  - [2. Quy tắc đặt tên](#2-quy-tắc-đặt-tên)
  - [3. Code Style](#3-code-style)
  - [4. UI & Logic](#4-ui--logic)
- [📌 Chuẩn viết code RESTful API cho C#](#-chuẩn-viết-code-restful-api-cho-c)
  - [1. Cấu trúc dự án](#1-cấu-trúc-dự-án-1)
  - [2. Quy tắc đặt tên endpoint](#2-quy-tắc-đặt-tên-endpoint)
  - [3. Quy tắc đặt tên Controller](#3-quy-tắc-đặt-tên-controller)
  - [4. Quy tắc DTO & Model](#4-quy-tắc-dto--model)
  - [5. Error Handling & Validation](#5-error-handling--validation)
  - [6. Code Style](#6-code-style)
  - [7. Quy tắc khác](#7-quy-tắc-khác)
- [📦 Chuẩn JSON trả về (API Response)](#-chuẩn-json-trả-về-api-response)
  - [1. Thành công](#1-thành-công)
  - [2. Lỗi](#2-lỗi)
  - [3. Danh sách (có phân trang)](#3-danh-sách-có-phân-trang)

---

# 🖥️ Frontend (React + TypeScript)

## 1. Cấu trúc dự án
```txt
src/
 ├── components/        # UI components (Button, Card, Modal, ...)
 │    └── Button.tsx
 │    └── UserCard.tsx
 │
 ├── routes/            # Quản lý router
 │    ├── index.tsx
 │    ├── PrivateRoute.tsx
 │    └── publicRoutes.ts
 │
 ├── pages/             # Các trang
 │    └── Home.tsx
 │    └── Profile.tsx
 │
 ├── hooks/             # Custom hooks
 │    └── useAuth.ts
 │    └── useFetch.ts
 │
 ├── services/          # API calls (mapping với RESTful API BE)
 │    └── user.service.ts
 │
 ├── models/            # Mapping entity với DTO từ BE
 │    └── user.model.ts
 │
 ├── types/             # Định nghĩa type generic (ApiResponse, Pagination)
 │    └── api-response.type.ts
 │
 ├── utils/             # Hàm tiện ích
 │    └── formatDate.ts
 │    └── storage.ts
 │
 ├── constants/         # Hằng số, error codes, api endpoints
 │    └── api-endpoints.ts
 │    └── error-codes.ts
 │
 ├── assets/            # Hình ảnh, css, font
 │
 ├── App.tsx
 └── main.tsx
````

## 2. Quy tắc đặt tên

* **Component:** PascalCase

```typescript
function UserProfile() { ... }
```

* **File:** trùng tên component, PascalCase
  `UserProfile.tsx`

* **Hooks:** bắt đầu bằng `use`
  `useAuth.ts, useFetch.ts`

* **Biến & hàm:** camelCase

```typescript
const userName: string = "Doc";
function getUserProfile(): Promise<IUser> {}
```

* **Interface & Type:** PascalCase, prefix `I` với interface

```typescript
interface IUser {
   id: number;
   name: string;
}
```

* **Route path:** lowercase, nhiều từ thì gạch ngang
  `/user-profile`, `/course-detail/:id`

## 3. Code Style

* Dùng **ES6+ + TypeScript features** (arrow function, async/await, destructuring, generics).
* Luôn dùng **functional component + React Hooks**, tránh class component.
* State đặt ngắn gọn, rõ nghĩa

```typescript
const [user, setUser] = useState<IUser | null>(null);
```

* Destructuring props kèm type:

```typescript
type UserCardProps = { name: string; age: number };

function UserCard({ name, age }: UserCardProps) {
  return <div>{name} - {age}</div>;
}
```

## 4. UI & Logic

* **Tách biệt logic và UI**

  * Logic → hooks/services
  * UI → component

Ví dụ logic (service gọi API):

```typescript
import { IUser } from "../models/user.model";

export async function getUsers(): Promise<IUser[]> {
  const res = await fetch("/api/users");
  return res.json();
}
```

Ví dụ UI component:

```tsx
import { getUsers } from "../services/user.service";

export function UserList() {
  const [users, setUsers] = useState<IUser[]>([]);

  useEffect(() => {
    getUsers().then(setUsers);
  }, []);

  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}
```

* Luôn kiểm tra null/undefined trước khi render:

```tsx
{user && <UserCard name={user.name} age={20} />}
```

---

# 📌 Chuẩn viết code RESTful API cho C\#

## 1. Cấu trúc dự án

```txt
src/
 ├── Presentation/       # Xử lý request, response
 ├── BLL/                # Xử lý logic, gọi DB
 ├── DAL/                # Định nghĩa data model
 ├── Applications/       # Helpers
 │     └── utils/
 │     └── mappers/
 │     └── DTOs/
 │     └── auth/
 │     └── …
```

## 2. Quy tắc đặt tên endpoint

* Dùng *danh từ số nhiều* (plural nouns).
* *Không* nhúng hành động trong URL (`/api/users/create` ❌).
* Action được quyết định bằng *HTTP verb*.

Ví dụ cho resource `User`:

| HTTP Verb | Endpoint        | Mô tả                     |
| --------- | --------------- | ------------------------- |
| GET       | /api/users      | Lấy danh sách user        |
| GET       | /api/users/{id} | Lấy chi tiết user theo id |
| POST      | /api/users      | Tạo user mới              |
| PUT       | /api/users/{id} | Cập nhật toàn bộ user     |
| PATCH     | /api/users/{id} | Cập nhật một phần user    |
| DELETE    | /api/users/{id} | Xóa user                  |

👉 Nếu có sub-resource:

```bash
GET /api/users/1/posts        # Lấy tất cả bài post của user 1
GET /api/users/1/posts/99     # Lấy chi tiết post 99 của user 1
```

## 3. Quy tắc đặt tên Controller

* PascalCase + suffix `Controller`.
* Tên controller khớp với resource.
* ASP.NET Core mặc định map: `UsersController` → `/api/users`.

Ví dụ:

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    // GET /api/users
    [HttpGet]
    public async Task<IActionResult> GetAllUsers() { ... }

    // GET /api/users/{id}
    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetUserById(int id) { ... }

    // POST /api/users
    [HttpPost]
    public async Task<IActionResult> CreateUser([FromBody] CreateUserDto dto) { ... }

    // PUT /api/users/{id}
    [HttpPut("{id:int}")]
    public async Task<IActionResult> UpdateUser(int id, [FromBody] UpdateUserDto dto) { ... }

    // DELETE /api/users/{id}
    [HttpDelete("{id:int}")]
    public async Task<IActionResult> DeleteUser(int id) { ... }
}
```

## 4. Quy tắc DTO & Model

* **Entity (DB model):** PascalCase, số ít → `User`.
* **DTO:** PascalCase + suffix `Dto` → `CreateUserDto`, `UpdateUserDto`.
* **Interface:** PascalCase, prefix `I` → `IUserService`.

Ví dụ:

```csharp
public class CreateUserDto
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
}

public class UpdateUserDto
{
    public string? Name { get; set; }
    public string? Email { get; set; }
}
```

## 5. Error Handling & Validation

* Dùng *ModelState* để validate input.
* Trả về mã lỗi chuẩn: `400 BadRequest`, `401 Unauthorized`, `404 NotFound`, `500 InternalServerError`.
* Middleware global để handle exception.
* Tạo 1 class chung để phân loại các request.

Ví dụ Validation:

```csharp
[HttpPost]
public async Task<IActionResult> CreateUser([FromBody] CreateUserDto dto)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(new {
            success = false,
            error = new { code = 400, message = "Invalid request data" }
        });
    }

    var user = await _userService.CreateUserAsync(dto);
    return Ok(new { success = true, data = user });
}
```

## 6. Code Style

* Dùng `async/await` cho tất cả API call tới DB
* Controller chỉ xử lý request/response, logic chính đặt trong Service
* Request validation bằng **FluentValidation** hoặc **DataAnnotation**
* Error handling qua **Middleware chung**

Ví dụ:

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;

    public UsersController(IUserService userService)
    {
        _userService = userService;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetUserById(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
        {
            return NotFound(new {
                success = false,
                error = new { code = 404, message = "User not found" }
            });
        }
        return Ok(new { success = true, data = user });
    }
}
```

## 7. Quy tắc khác

* Tên phương thức trong Controller: PascalCase (`GetUserById`).
* Không viết logic trong Controller → tách sang `Service`.
* Sử dụng *async/await* cho tất cả thao tác DB/IO.
* Swagger/OpenAPI để mô tả API.
* Thêm Service Register để không cần khai báo nhiều trong `program.cs`.
* Mỗi Service/Repo cần 1 Interface riêng.

---

# 📦 Chuẩn JSON trả về (API Response)

## 1. Thành công

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Nguyen Van A"
  }
}
```

## 2. Lỗi

```json
{
  "success": false,
  "error": {
    "code": 404,
    "message": "User not found"
  }
}
```

## 3. Danh sách (có phân trang)

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "A" },
    { "id": 2, "name": "B" }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 52
  }
}
```
