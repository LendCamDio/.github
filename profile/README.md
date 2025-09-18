## 🖥️ Frontend (React + TypeScript)

### 1. Cấu trúc dự án

```txt
src/
 ├── assets/            # Hình ảnh, css, font
 ├── components/        # UI components (Button, Card, Modal, ...)
 ├── contexts/          # React Context (AuthContext, ThemeContext)
 ├── layouts/           # Layout chung (MainLayout, AuthLayout)
 ├── routes/            # Quản lý router
 ├── pages/             # Các trang (Home, Profile...)
 ├── hooks/             # Custom hooks (useAuth, useFetch)
 ├── services/          # API calls (mapping với RESTful API BE)
 ├── models/            # Mapping entity với DTO từ BE
 ├── types/             # Type generic (ApiResponse, Pagination)
 ├── utils/             # Hàm tiện ích (formatDate, storage)
 ├── constants/         # Hằng số, error codes, api endpoints
 ├── tests/             # Unit test / integration test (Jest, Vitest)
 ├── App.tsx
 └── main.tsx
```

### 2. Quy tắc đặt tên

* **Component & File:** PascalCase → `UserProfile.tsx`
* **Hook:** bắt đầu bằng `use` → `useAuth.ts`
* **Biến & hàm:** camelCase → `getUserProfile()`
* **Interface/Type:** PascalCase. Prefix `I` nếu cần → `IUser`
* **Route path:** lowercase + gạch ngang → `/user-profile/:id`

### 3. Code Style

* Luôn dùng **functional component + hooks** (tránh class).
* **ESLint + Prettier**: bắt buộc auto format.
* Import: nhóm (lib, hooks, components).
* Props > 3 field → định nghĩa `interface`, không inline.
* Luôn xử lý **loading + error state**.

Ví dụ:

```tsx
function UserCard({ name, age }: { name: string; age: number }) {
  return <div>{name} - {age}</div>;
}
```

### 4. UI & Logic

* Tách biệt **logic** (service/hook) và **UI** (component).
* API calls luôn đi qua **service layer**.
* Render condition:

```tsx
{user ? <UserCard name={user.name} age={20} /> : <p>No user</p>}
```

---

## 📌 Backend (C# RESTful API)

### 1. Cấu trúc dự án (Clean Architecture)

```txt
src/
 ├── Domain/            # Entity, ValueObject
 ├── Application/       # DTO, Service Interface, Use Case
 ├── Infrastructure/    # EF Core, Repository
 ├── Presentation/      # Controller, Request/Response
```

### 2. Endpoint Naming

* Dùng **danh từ số nhiều** (plural nouns).
* Không nhúng hành động trong URL.
* Action quyết định bằng **HTTP verb**.

Ví dụ:

```
GET    /api/users
GET    /api/users/{id}
POST   /api/users
PUT    /api/users/{id}
DELETE /api/users/{id}
```

Sub-resource:

```
GET /api/users/1/posts
GET /api/users/1/posts/99
```

### 3. Controller

* PascalCase + suffix `Controller`.
* Không chứa business logic → gọi Service.
* Method: PascalCase (`GetUserById`).

Ví dụ:

```csharp
[ApiController]
[Route("api/users")]
public class UsersController : ControllerBase {
    private readonly IUserService _userService;
    public UsersController(IUserService userService) => _userService = userService;

    [HttpGet("{id:int}")]
    public async Task<IActionResult> GetUserById(int id) {
        var user = await _userService.GetUserByIdAsync(id);
        return user == null
            ? NotFound(new { success = false, error = new { code = 404, message = "User not found" }})
            : Ok(new { success = true, data = user });
    }
}
```

### 4. DTO & Model

* **Entity:** PascalCase (User).
* **DTO:** PascalCase + suffix `Dto` (CreateUserDto).
* **Interface:** prefix `I` (IUserService).
* Không expose Entity trực tiếp ra API.

### 5. Error Handling & Validation

* Middleware global handle exception.
* Validation bằng **FluentValidation**.
* Status code chuẩn: `400, 401, 403, 404, 500`.

Ví dụ:

```csharp
if (!ModelState.IsValid) {
    return BadRequest(new {
        success = false,
        error = new { code = 400, message = "Invalid request data" }
    });
}
```

### 6. Code Style

* Bắt buộc **async/await** cho DB/IO.
* **Repository** → chỉ truy cập DB.
* **Service** → chứa business logic.
* **Controller** → request/response.

### 7. Quy tắc khác

* Swagger/OpenAPI cho API docs.
* Serilog cho logging.
* AutoMapper để map DTO ↔ Entity.
* Interface cho mọi Service/Repo.

---

## 📦 API Response Format

### 1. Thành công

```json
{
  "success": true,
  "data": { "id": 1, "name": "Nguyen Van A" }
}
```

### 2. Lỗi

```json
{
  "success": false,
  "error": {
    "code": 404,
    "message": "User not found",
    "traceId": "d13abf8f"
  }
}
```

### 3. Danh sách (có phân trang)

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "A" },
    { "id": 2, "name": "B" }
  ],
  "meta": {
    "page": 1,
    "limit": 10,
    "total": 52
  }
}
```
