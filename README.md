# 📘 FastAPI Practice Assignment: Products Management API

**Course:** Backend Development with FastAPI  
**Instructor:** djumanov  
**Duration:** 2 weeks  
**Level:** Junior 

---

## 🎯 Loyiha Maqsadi

Siz **FastAPI**, **SQLAlchemy ORM**, va **PostgreSQL** yordamida to'liq funksional mahsulotlar boshqaruv tizimini yaratishingiz kerak. Bu loyihada:

- **2 ta table** bilan ishlaysiz (Category va Product)
- **Relationship** (Foreign Key) o'rganasiz
- **To'liq CRUD** operatsiyalarni amalga oshirasiz
- **Turli xil GET endpoints** (search, filter, pagination) yozasiz
- **Ma'lumotlar validatsiyasi** va **error handling** bilan ishlaysiz

---

## 🗄 Database Structure

### Table 1: `categories`

| Field       | Type         | Constraints              | Description                    |
|-------------|--------------|--------------------------|--------------------------------|
| id          | Integer      | Primary Key, Auto        | Kategoriya ID (auto)           |
| name        | String(100)  | NOT NULL, UNIQUE         | Kategoriya nomi                |
| description | Text         | NULL                     | Kategoriya tavsifi (optional)  |

### Table 2: `products`

| Field       | Type         | Constraints              | Description                    |
|-------------|--------------|--------------------------|--------------------------------|
| id          | Integer      | Primary Key, Auto        | Mahsulot ID (auto)             |
| name        | String(200)  | NOT NULL                 | Mahsulot nomi                  |
| description | Text         | NULL                     | Mahsulot tavsifi (optional)    |
| price       | Float        | NOT NULL, CHECK > 0      | Narxi (0 dan katta bo'lishi kerak) |
| in_stock    | Boolean      | DEFAULT True             | Mavjudmi? (True/False)         |
| category_id | Integer      | Foreign Key, NOT NULL    | Kategoriya ID (categories.id)  |

**Relationship:** Har bir mahsulot bitta kategoriyaga tegishli (Many-to-One)

---

## 🎯 API Endpoints Documentation

### **📁 CATEGORIES - CRUD Operations**

---

### **1. Create Category**
**`POST /api/categories`**

**Vazifa:** Yangi kategoriya yaratish.

**Request Body:**
```json
{
  "name": "Electronics",
  "description": "Electronic devices and gadgets"
}
```

**Validation Rules:**
- `name`: **required**, 2-100 characters
- `description`: optional, max 500 characters

**Response (201 Created):**
```json
{
  "id": 1,
  "name": "Electronics",
  "description": "Electronic devices and gadgets"
}
```

**Error Cases:**

**400 Bad Request** - Kategoriya nomi allaqachon mavjud:
```json
{
  "detail": "Category with this name already exists"
}
```

**422 Unprocessable Entity** - Validation error:
```json
{
  "detail": [
    {
      "loc": ["body", "name"],
      "msg": "ensure this value has at least 2 characters",
      "type": "value_error.any_str.min_length"
    }
  ]
}
```

**Invalid Request Examples:**
```json
// ❌ Name too short
{
  "name": "E",
  "description": "Test"
}

// ❌ Name missing
{
  "description": "Test category"
}

// ❌ Name too long (>100 chars)
{
  "name": "This is a very long category name that exceeds the maximum allowed length of 100 characters and will cause validation error",
  "description": "Test"
}
```

---

### **2. Get All Categories**
**`GET /api/categories`**

**Vazifa:** Barcha kategoriyalarni olish.

**Query Parameters:** Yo'q

**Validation:** Query parameters yo'q, validation ham yo'q

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices"
  },
  {
    "id": 2,
    "name": "Clothing",
    "description": "Fashion items"
  }
]
```

---

### **3. Get Category by ID**
**`GET /api/categories/{category_id}`**

**Vazifa:** ID bo'yicha bitta kategoriyani olish.

**Path Parameters:**
- `category_id` (integer, required): Kategoriya ID si

**Validation Rules:**
- `category_id`: Must be positive integer

**Response (200 OK):**
```json
{
  "id": 1,
  "name": "Electronics",
  "description": "Electronic devices"
}
```

**Error Cases:**

**404 Not Found** - Kategoriya topilmasa:
```json
{
  "detail": "Category not found"
}
```

**422 Unprocessable Entity** - ID integer emas:
```json
{
  "detail": [
    {
      "loc": ["path", "category_id"],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

**Invalid Request Examples:**
- `/api/categories/abc` ❌ (string instead of integer)
- `/api/categories/-5` ❌ (negative number)
- `/api/categories/0` ❌ (zero)

---

### **4. Update Category**
**`PUT /api/categories/{category_id}`**

**Vazifa:** Kategoriyani to'liq yoki qisman yangilash.

**Path Parameters:**
- `category_id` (integer, required): Yangilanayotgan kategoriya ID si

**Request Body (barcha fieldlar optional):**
```json
{
  "name": "Home Electronics",
  "description": "Updated description for electronics"
}
```

**Validation Rules:**
- `name`: optional, 2-100 characters (agar berilsa)
- `description`: optional, max 500 characters

**Response (200 OK):**
```json
{
  "id": 1,
  "name": "Home Electronics",
  "description": "Updated description for electronics"
}
```

**Error Cases:**

**404 Not Found** - Kategoriya topilmasa:
```json
{
  "detail": "Category not found"
}
```

**400 Bad Request** - Yangi nom bilan boshqa kategoriya mavjud:
```json
{
  "detail": "Category with name 'Electronics' already exists"
}
```

**422 Unprocessable Entity** - Validation error:
```json
{
  "detail": [
    {
      "loc": ["body", "name"],
      "msg": "ensure this value has at most 100 characters",
      "type": "value_error.any_str.max_length"
    }
  ]
}
```

---

### **5. Delete Category**
**`DELETE /api/categories/{category_id}`**

**Vazifa:** Kategoriyani o'chirish (faqat agar unga bog'langan mahsulotlar bo'lmasa).

**Path Parameters:**
- `category_id` (integer, required): O'chiriladigan kategoriya ID si

**Validation:** ID must be positive integer

**Response (200 OK):**
```json
{
  "message": "Category deleted successfully"
}
```

**Error Cases:**

**404 Not Found** - Kategoriya topilmasa:
```json
{
  "detail": "Category not found"
}
```

**400 Bad Request** - Kategoriyaga tegishli mahsulotlar mavjud:
```json
{
  "detail": "Cannot delete category. 5 products are linked to this category"
}
```

---

### **📦 PRODUCTS - CRUD Operations**

---

### **6. Create Product**
**`POST /api/products`**

**Vazifa:** Yangi mahsulot yaratish.

**Request Body:**
```json
{
  "name": "iPhone 15 Pro",
  "description": "Latest Apple smartphone with A17 chip",
  "price": 999.99,
  "in_stock": true,
  "category_id": 1
}
```

**Validation Rules:**
- `name`: **required**, 2-200 characters
- `description`: optional, max 1000 characters
- `price`: **required**, > 0, max 999999.99
- `in_stock`: optional, default true
- `category_id`: **required**, must exist in database

**Response (201 Created):**
```json
{
  "id": 1,
  "name": "iPhone 15 Pro",
  "description": "Latest Apple smartphone with A17 chip",
  "price": 999.99,
  "in_stock": true,
  "category_id": 1,
  "category": {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices"
  }
}
```

**Error Cases:**

**404 Not Found** - Category mavjud emas:
```json
{
  "detail": "Category with id 999 not found"
}
```

**422 Unprocessable Entity** - Validation errors:

1. **Price negative yoki zero:**
```json
{
  "detail": [
    {
      "loc": ["body", "price"],
      "msg": "ensure this value is greater than 0",
      "type": "value_error.number.not_gt",
      "ctx": {"limit_value": 0}
    }
  ]
}
```

2. **Name too short:**
```json
{
  "detail": [
    {
      "loc": ["body", "name"],
      "msg": "ensure this value has at least 2 characters",
      "type": "value_error.any_str.min_length"
    }
  ]
}
```

3. **Multiple validation errors:**
```json
{
  "detail": [
    {
      "loc": ["body", "name"],
      "msg": "field required",
      "type": "value_error.missing"
    },
    {
      "loc": ["body", "price"],
      "msg": "field required",
      "type": "value_error.missing"
    },
    {
      "loc": ["body", "category_id"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

**Invalid Request Examples:**
```json
// ❌ Negative price
{
  "name": "Test Product",
  "price": -50,
  "category_id": 1
}

// ❌ Zero price
{
  "name": "Test Product",
  "price": 0,
  "category_id": 1
}

// ❌ Missing required fields
{
  "description": "Only description provided"
}

// ❌ Price too large
{
  "name": "Expensive Item",
  "price": 10000000,
  "category_id": 1
}

// ❌ Invalid data types
{
  "name": "Test",
  "price": "not a number",
  "in_stock": "yes",
  "category_id": "abc"
}
```

---

### **7. Get All Products**
**`GET /api/products`**

**Vazifa:** Barcha mahsulotlarni olish (kategoriya ma'lumotlari bilan birga).

**Query Parameters:** Yo'q

**Validation:** Yo'q

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "iPhone 15 Pro",
    "description": "Latest Apple smartphone",
    "price": 999.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics",
      "description": "Electronic devices"
    }
  }
]
```

---

### **8. Get Product by ID**
**`GET /api/products/{product_id}`**

**Vazifa:** ID bo'yicha bitta mahsulotni olish.

**Path Parameters:**
- `product_id` (integer, required): Mahsulot ID si

**Validation Rules:**
- `product_id`: Must be positive integer

**Response (200 OK):**
```json
{
  "id": 1,
  "name": "iPhone 15 Pro",
  "description": "Latest Apple smartphone",
  "price": 999.99,
  "in_stock": true,
  "category_id": 1,
  "category": {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices"
  }
}
```

**Error Cases:**

**404 Not Found:**
```json
{
  "detail": "Product with id 999 not found"
}
```

**422 Unprocessable Entity:**
```json
{
  "detail": [
    {
      "loc": ["path", "product_id"],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

---

### **9. Update Product**
**`PUT /api/products/{product_id}`**

**Vazifa:** Mahsulotni to'liq yoki qisman yangilash.

**Path Parameters:**
- `product_id` (integer, required): Yangilanayotgan mahsulot ID si

**Request Body (barcha fieldlar optional):**
```json
{
  "name": "iPhone 15 Pro Max",
  "price": 1099.99,
  "in_stock": false
}
```

**Validation Rules:**
- `name`: optional, 2-200 characters (agar berilsa)
- `description`: optional, max 1000 characters
- `price`: optional, > 0 (agar berilsa)
- `in_stock`: optional, boolean
- `category_id`: optional, must exist in database (agar berilsa)

**Response (200 OK):**
```json
{
  "id": 1,
  "name": "iPhone 15 Pro Max",
  "description": "Latest Apple smartphone",
  "price": 1099.99,
  "in_stock": false,
  "category_id": 1,
  "category": {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices"
  }
}
```

**Error Cases:**

**404 Not Found** - Mahsulot topilmasa:
```json
{
  "detail": "Product with id 999 not found"
}
```

**404 Not Found** - Yangi category_id topilmasa:
```json
{
  "detail": "Category with id 99 not found"
}
```

**422 Unprocessable Entity:**
```json
{
  "detail": [
    {
      "loc": ["body", "price"],
      "msg": "ensure this value is greater than 0",
      "type": "value_error.number.not_gt"
    }
  ]
}
```

---

### **10. Delete Product**
**`DELETE /api/products/{product_id}`**

**Vazifa:** Mahsulotni o'chirish.

**Path Parameters:**
- `product_id` (integer, required): O'chiriladigan mahsulot ID si

**Validation:** ID must be positive integer

**Response (200 OK):**
```json
{
  "message": "Product deleted successfully"
}
```

**Error Cases:**

**404 Not Found:**
```json
{
  "detail": "Product with id 999 not found"
}
```

---

### **🔍 PRODUCTS - Search & Filter Endpoints**

---

### **11. Search Products by Name**
**`GET /api/products/search`**

**Vazifa:** Mahsulot nomida berilgan so'z bor yoki yo'qligini qidirish (partial match, case-insensitive).

**Query Parameters:**
- `name` (string, optional): Qidirilayotgan so'z yoki so'z qismi

**Validation Rules:**
- `name`: optional, min 1 character if provided

**Misol so'rovlar:**
- `/api/products/search?name=iphone`
- `/api/products/search?name=samsung`
- `/api/products/search` (barcha mahsulotlar)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "iPhone 15 Pro",
    "description": "Latest Apple smartphone",
    "price": 999.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics"
    }
  }
]
```

**Izoh:** Agar hech narsa topilmasa, bo'sh array `[]` qaytariladi (404 emas).

---

### **12. Filter Products by Category**
**`GET /api/products/filter/category`**

**Vazifa:** Kategoriya nomi bo'yicha mahsulotlarni filterlash.

**Query Parameters:**
- `category` (string, required): Kategoriya nomi (exact match)

**Validation Rules:**
- `category`: **required**, min 2 characters

**Misol so'rovlar:**
- `/api/products/filter/category?category=Electronics`
- `/api/products/filter/category?category=Clothing`

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "iPhone 15 Pro",
    "price": 999.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics"
    }
  }
]
```

**Error Cases:**

**422 Unprocessable Entity** - Parameter berilmasa:
```json
{
  "detail": [
    {
      "loc": ["query", "category"],
      "msg": "field required",
      "type": "value_error.missing"
    }
  ]
}
```

---

### **13. Filter Products by Price Range**
**`GET /api/products/filter/price`**

**Vazifa:** Narx oralig'i bo'yicha mahsulotlarni filterlash.

**Query Parameters:**
- `min_price` (float, optional): Minimal narx (>= 0)
- `max_price` (float, optional): Maksimal narx (>= 0)

**Validation Rules:**
- `min_price`: optional, must be >= 0
- `max_price`: optional, must be >= 0
- Agar ikkalasi ham berilsa: `min_price` <= `max_price`

**Misol so'rovlar:**
- `/api/products/filter/price?min_price=500&max_price=1000`
- `/api/products/filter/price?min_price=1000`
- `/api/products/filter/price?max_price=500`
- `/api/products/filter/price` (barcha mahsulotlar)

**Response (200 OK):**
```json
[
  {
    "id": 2,
    "name": "Samsung Galaxy S24",
    "price": 899.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics"
    }
  }
]
```

**Error Cases:**

**422 Unprocessable Entity** - Negative values:
```json
{
  "detail": [
    {
      "loc": ["query", "min_price"],
      "msg": "ensure this value is greater than or equal to 0",
      "type": "value_error.number.not_ge"
    }
  ]
}
```

**400 Bad Request** - min_price > max_price:
```json
{
  "detail": "min_price cannot be greater than max_price"
}
```

**Invalid Examples:**
- `/api/products/filter/price?min_price=-50` ❌
- `/api/products/filter/price?min_price=1000&max_price=500` ❌
- `/api/products/filter/price?min_price=abc` ❌

---

### **14. Filter Products by Stock Status**
**`GET /api/products/filter/stock`**

**Vazifa:** Mahsulotlarni mavjudlik holatiga qarab filterlash.

**Query Parameters:**
- `in_stock` (boolean, required): `true` yoki `false`

**Validation Rules:**
- `in_stock`: **required**, must be boolean (true/false, 1/0, yes/no)

**Misol so'rovlar:**
- `/api/products/filter/stock?in_stock=true`
- `/api/products/filter/stock?in_stock=false`
- `/api/products/filter/stock?in_stock=1` (true sifatida qabul qilinadi)
- `/api/products/filter/stock?in_stock=0` (false sifatida qabul qilinadi)

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "iPhone 15 Pro",
    "price": 999.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics"
    }
  }
]
```

**Error Cases:**

**422 Unprocessable Entity:**
```json
{
  "detail": [
    {
      "loc": ["query", "in_stock"],
      "msg": "value could not be parsed to a boolean",
      "type": "type_error.bool"
    }
  ]
}
```

**Invalid Examples:**
- `/api/products/filter/stock?in_stock=yes` ❌
- `/api/products/filter/stock?in_stock=maybe` ❌
- `/api/products/filter/stock` ❌ (parameter yo'q)

---

### **15. Get Products with Pagination**
**`GET /api/products/paginated`**

**Vazifa:** Mahsulotlarni qismlarga bo'lib olish (pagination).

**Query Parameters:**
- `limit` (integer, default=10): Bir sahifada nechta mahsulot
- `offset` (integer, default=0): Qayerdan boshlash

**Validation Rules:**
- `limit`: 1 <= limit <= 100
- `offset`: >= 0

**Misol so'rovlar:**
- `/api/products/paginated?limit=5&offset=0`
- `/api/products/paginated?limit=10&offset=20`
- `/api/products/paginated` (default: limit=10, offset=0)

**Response (200 OK):**
```json
{
  "total": 25,
  "limit": 5,
  "offset": 0,
  "items": [
    {
      "id": 1,
      "name": "iPhone 15 Pro",
      "price": 999.99,
      "in_stock": true,
      "category_id": 1,
      "category": {
        "id": 1,
        "name": "Electronics"
      }
    }
  ]
}
```

**Error Cases:**

**422 Unprocessable Entity** - Invalid limit:
```json
{
  "detail": [
    {
      "loc": ["query", "limit"],
      "msg": "ensure this value is less than or equal to 100",
      "type": "value_error.number.not_le"
    }
  ]
}
```

**422 Unprocessable Entity** - Negative offset:
```json
{
  "detail": [
    {
      "loc": ["query", "offset"],
      "msg": "ensure this value is greater than or equal to 0",
      "type": "value_error.number.not_ge"
    }
  ]
}
```

**Invalid Examples:**
- `/api/products/paginated?limit=0` ❌ (too small)
- `/api/products/paginated?limit=200` ❌ (too large)
- `/api/products/paginated?offset=-5` ❌ (negative)
- `/api/products/paginated?limit=abc` ❌ (not integer)

---

### **16. Get Products by Category ID**
**`GET /api/products/category/{category_id}`**

**Vazifa:** Kategoriya ID si bo'yicha barcha mahsulotlarni olish.

**Path Parameters:**
- `category_id` (integer, required): Kategoriya ID si

**Validation Rules:**
- `category_id`: Must be positive integer

**Response (200 OK):**
```json
[
  {
    "id": 1,
    "name": "iPhone 15 Pro",
    "price": 999.99,
    "in_stock": true,
    "category_id": 1,
    "category": {
      "id": 1,
      "name": "Electronics"
    }
  }
]
```

**Error Cases:**

**404 Not Found:**
```json
{
  "detail": "Category with id 999 not found"
}
```

**422 Unprocessable Entity:**
```json
{
  "detail": [
    {
      "loc": ["path", "category_id"],
      "msg": "value is not a valid integer",
      "type": "type_error.integer"
    }
  ]
}
```

---

### **17. Sort Products by Price**
**`GET /api/products/sort/price`**

**Vazifa:** Mahsulotlarni narxi bo'yicha tartiblash.

**Query Parameters:**
- `order` (string, required): `asc` yoki `desc`

**Validation Rules:**
- `order`: **required**, must be either "asc" or "desc"

**Misol so'rovlar:**
- `/api/products/sort/price?order=asc` (arzondan qimmatga)
- `/api/products/sort/price?order=desc` (qimmatdan arzonga)

**Response (200 OK):**
```json
[
  {
    "id": 4,
    "name": "Budget Phone",
    "price": 199.99,
    "in_stock": true,
    "category_id": 1
  },
  {
    "id": 2,
    "name": "Samsung Galaxy S24",
    "price": 899.99,
    "in_stock": true,
    "category_id": 1
  }
]
```

**Error Cases:**

**422 Unprocessable Entity** - Invalid order value:
```json
{
  "detail": [
    {
      "loc": ["query", "order"],
      "msg": "value must be either 'asc' or 'desc'",
      "type": "value_error.const"
    }
  ]
}
```

**Invalid Examples:**
- `/api/products/sort/price?order=ascending` ❌
- `/api/products/sort/price?order=high` ❌
- `/api/products/sort/price` ❌ (parameter yo'q)

---

### **18. Advanced Search (Kombinatsiyalangan filter)**
**`GET /api/products/advanced-search`**

**Vazifa:** Bir nechta filtrlarni birgalikda qo'llash.

**Query Parameters (barchasi optional):**
- `name` (string): Nom bo'yicha qidirish
- `category_id` (integer): Kategoriya ID si
- `min_price` (float): Minimal narx (>= 0)
- `max_price` (float): Maksimal narx (>= 0)
- `in_stock` (boolean): Mavjudlik holati
- `sort_by` (string): `price_asc`, `price_desc`, `name_asc`, `name_desc`
- `limit` (integer, default=20): Natijalar soni (1-100)
- `offset` (integer, default=0): Pagination offset (>= 0)

**Validation Rules:**
- `name`: min 1 character if provided
- `category_id`: must be positive integer, must exist
- `min_price`: >= 0
- `max_price`: >= 0, >= min_price
- `in_stock`: must be boolean
- `sort_by`: must be one of: "price_asc", "price_desc", "name_asc", "name_desc"
- `limit`: 1-100
- `offset`: >= 0

**Misol so'rovlar:**
```
/api/products/advanced-search?category_id=1&min_price=500&in_stock=true&sort_by=price_asc

/api/products/advanced-search?name=phone&max_price=1000&sort_by=price_desc&limit=10

/api/products/advanced-search?in_stock=false&category_id=2
```

**Response (200 OK):**
```json
{
  "total": 3,
  "limit": 20,
  "offset": 0,
  "items": [
    {
      "id": 2,
      "name": "Samsung Galaxy S24",
      "price": 899.99,
      "in_stock": true,
      "category_id": 1,
      "category": {
        "id": 1,
        "name": "Electronics"
      }
    }
  ]
}
```

**Error Cases:**

**400 Bad Request** - min_price > max_price:
```json
{
  "detail": "min_price cannot be greater than max_price"
}
```

**404 Not Found** - Category mavjud emas:
```json
{
  "detail": "Category with id 999 not found"
}
```

**422 Unprocessable Entity** - Multiple validation errors:
```json
{
  "detail": [
    {
      "loc": ["query", "limit"],
      "msg": "ensure this value is less than or equal to 100",
      "type": "value_error.number.not_le"
    },
    {
      "loc": ["query", "sort_by"],
      "msg": "value must be one of: price_asc, price_desc, name_asc, name_desc",
      "type": "value_error.const"
    }
  ]
}
```

**Invalid Examples:**
```
// ❌ Invalid sort_by
/api/products/advanced-search?sort_by=random

// ❌ Limit too large
/api/products/advanced-search?limit=500

// ❌ Negative price
/api/products/advanced-search?min_price=-100

// ❌ min > max
/api/products/advanced-search?min_price=1000&max_price=500

// ❌ Invalid boolean
/api/products/advanced-search?in_stock=maybe
```

---

### **19. Get Products Count**
**`GET /api/products/count`**

**Vazifa:** Jami mahsulotlar sonini olish.

**Query Parameters:**
- `category_id` (integer, optional): Faqat shu kategoriya uchun
- `in_stock` (boolean, optional): Faqat mavjud/mavjud bo'lmaganlar

**Validation Rules:**
- `category_id`: must be positive integer, must exist if provided
- `in_stock`: must be boolean if provided

**Misol so'rovlar:**
- `/api/products/count`
- `/api/products/count?category_id=1`
- `/api/products/count?in_stock=true`
- `/api/products/count?category_id=1&in_stock=false`

**Response (200 OK):**
```json
{
  "total": 25,
  "in_stock": 20,
  "out_of_stock": 5
}
```

**Error Cases:**

**404 Not Found:**
```json
{
  "detail": "Category with id 999 not found"
}
```

**422 Unprocessable Entity:**
```json
{
  "detail": [
    {
      "loc": ["query", "in_stock"],
      "msg": "value could not be parsed to a boolean",
      "type": "type_error.bool"
    }
  ]
}
```

---

### **20. Get Products Statistics**
**`GET /api/products/statistics`**

**Vazifa:** Mahsulotlar statistikasini olish.

**Query Parameters:** Yo'q

**Validation:** Yo'q

**Response (200 OK):**
```json
{
  "total_products": 25,
  "total_categories": 4,
  "in_stock_count": 20,
  "out_of_stock_count": 5,
  "average_price": 567.89,
  "min_price": 99.99,
  "max_price": 1999.99,
  "most_expensive_product": {
    "id": 10,
    "name": "MacBook Pro 16",
    "price": 1999.99
  },
  "cheapest_product": {
    "id": 15,
    "name": "Phone Case",
    "price": 99.99
  }
}
```

---

## 🧪 Test Ma'lumotlari

**Categories:**
```sql
INSERT INTO categories (name, description) VALUES
('Electronics', 'Electronic devices and gadgets'),
('Clothing', 'Fashion and apparel'),
('Books', 'Educational and entertainment books'),
('Home & Garden', 'Home improvement and gardening');
```

**Products:**
```sql
INSERT INTO products (name, description, price, in_stock, category_id) VALUES
('iPhone 15 Pro', 'Latest Apple smartphone', 999.99, true, 1),
('Samsung Galaxy S24', 'Samsung flagship phone', 899.99, true, 1),
('MacBook Pro 14', 'Apple laptop M3 chip', 1999.99, true, 1),
('Dell XPS 15', 'High-performance laptop', 1499.99, false, 1),
('Sony WH-1000XM5', 'Noise cancelling headphones', 349.99, true, 1),
('iPad Air', 'Apple tablet', 599.99, true, 1),
('AirPods Pro', 'Wireless earbuds', 249.99, true, 1),
('Samsung TV 55"', '4K Smart TV', 799.99, false, 1),

('Nike Air Max', 'Running shoes', 129.99, true, 2),
('Adidas Hoodie', 'Comfortable hoodie', 59.99, true, 2),
('Levis Jeans', 'Classic blue jeans', 79.99, true, 2),
('Winter Jacket', 'Warm winter coat', 149.99, false, 2),

('Python Crash Course', 'Programming book', 39.99, true, 3),
('Clean Code', 'Software engineering', 44.99, true, 3),
('Harry Potter Set', 'Complete book series', 89.99, false, 3),

('Garden Tools Set', 'Complete gardening kit', 129.99, true, 4),
('Indoor Plant Pot', 'Decorative pot', 24.99, true, 4),
('LED Lamp', 'Smart home lighting', 49.99, true, 4);
```

---

## 📋 Validation Summary Table

### **Common Validation Rules:**

| Data Type | Rule | Example | Error Code |
|-----------|------|---------|------------|
| String (name) | Min 2, Max 200 chars | "iPhone" ✅, "X" ❌ | value_error.any_str.min_length |
| Float (price) | > 0, Max 999999.99 | 99.99 ✅, -50 ❌, 0 ❌ | value_error.number.not_gt |
| Integer (ID) | Positive integer | 5 ✅, -1 ❌, "abc" ❌ | type_error.integer |
| Boolean | true/false/1/0 | true ✅, "maybe" ❌ | type_error.bool |
| Query limit | 1-100 | 50 ✅, 0 ❌, 200 ❌ | value_error.number.not_le |
| Query offset | >= 0 | 10 ✅, -5 ❌ | value_error.number.not_ge |

---

## 💡 Validation Best Practices

1. **Always validate input data** - Server hech qachon clientga ishonmasligi kerak
2. **Return clear error messages** - Foydalanuvchiga nima xato ekanligini tushuntiring
3. **Use HTTP status codes correctly:**
   - 400: Business logic error
   - 404: Resource not found
   - 422: Validation error
4. **Validate relationships** - Foreign key'lar mavjudligini tekshiring
5. **Handle edge cases** - 0, negative numbers, empty strings, null values
