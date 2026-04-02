# Boster Starter Kit Documentation

Starter kit Laravel 13 + React (Inertia.js) dengan sistem Role & Permission.

**Tech Stack:** Laravel 13, React 19, Inertia.js v2, Spatie Permission, Laravel Fortify, Tailwind CSS v4, Radix UI

---

## Sistem Permission & Role

### Package

Menggunakan **spatie/laravel-permission** v7.1 dengan model berbasis UUID.

### Database Tables

| Table                   | Keterangan                                         |
| ----------------------- | -------------------------------------------------- |
| `permissions`           | Menyimpan permission dengan kolom `group` (custom) |
| `roles`                 | Menyimpan role (admin, user)                       |
| `model_has_permissions` | Assign permission langsung ke user                 |
| `model_has_roles`       | Assign role ke user                                |
| `role_has_permissions`  | Pivot: permission milik role                       |

### Permission Convention

Format: `{module}.{action}`

```
users.create, users.read, users.update, users.delete
roles.create, roles.read, roles.update, roles.delete
permissions.create, permissions.read, permissions.update, permissions.delete
profile.read, profile.update
activity-logs.read
```

### Roles Default

| Role    | Permissions                                                      |
| ------- | ---------------------------------------------------------------- |
| `admin` | Semua permission (User, Role, Permission, Profile, Activity Log) |
| `user`  | Hanya `profile.read`, `profile.update`                           |

### File Terkait Permission

| File                                          | Fungsi                                                 |
| --------------------------------------------- | ------------------------------------------------------ |
| `database/seeders/PermissionGroupsSeeder.php` | Mendefinisikan semua permission & assign ke role       |
| `app/Services/PermissionGroupService.php`     | Helper untuk membuat group permission & assign ke role |
| `app/Models/Permission.php`                   | Model permission (extends Spatie, UUID)                |
| `app/Models/Role.php`                         | Model role (extends Spatie, UUID)                      |
| `app/Models/User.php`                         | User model dengan trait `HasRoles`                     |

### Cara Permission Dicek di Backend

Controller menggunakan middleware `can:permission.name`:

```php
// app/Http/Controllers/Management/UsersController.php
public function __construct(...)
{
    $this->middleware('can:users.read')->only(['index', 'fetchData']);
    $this->middleware('can:users.create')->only('store');
    $this->middleware('can:users.update')->only('update');
    $this->middleware('can:users.delete')->only(['destroy', 'bulkDestroy']);
}
```

### Cara Permission Dikirim ke Frontend

Middleware `HandleInertiaRequests` sharing permissions ke React:

```php
// app/Http/Middleware/HandleInertiaRequests.php
'auth' => [
    'user' => $user,
    'permissions' => $user
        ? $user->getAllPermissions()->pluck('name')->values()->all()
        : [],
],
```

---

## Sistem Menu (Navigation)

Menu **tidak disimpan di database**. Menu didefinisikan secara statis di TypeScript.

### File Lokasi Menu

`resources/js/config/navigation.ts`

### Struktur Menu

```typescript
// Tipe NavItem: resources/js/types/navigation.ts
type NavItem = {
    title: string; // Nama menu
    href: string; // Link tujuan
    icon?: LucideIcon; // Icon (dari lucide-react)
    requiredPermissions?: string[]; // Permission yang dibutuhkan
    children?: NavItem[]; // Sub-menu (bisa nested)
};
```

### Contoh Menu Saat Ini

```typescript
// resources/js/config/navigation.ts
export const mainNavItems: MainNavGroup[] = [
    {
        title: 'Platform',
        items: [
            { title: 'Dashboard', href: dashboard(), icon: LayoutGrid },
            {
                title: 'Management',
                href: '#',
                icon: FolderTree,
                requiredPermissions: [
                    'users.read',
                    'roles.read',
                    'permissions.read',
                    'activity-logs.read',
                ],
                children: [
                    {
                        title: 'Users',
                        href: users.index(),
                        requiredPermissions: ['users.read'],
                    },
                    {
                        title: 'Access Control',
                        href: '#',
                        requiredPermissions: ['roles.read', 'permissions.read'],
                        children: [
                            {
                                title: 'Permissions',
                                href: permissions.index(),
                                requiredPermissions: ['permissions.read'],
                            },
                            {
                                title: 'Roles',
                                href: roles.index(),
                                requiredPermissions: ['roles.read'],
                            },
                        ],
                    },
                    {
                        title: 'Activity Logs',
                        href: activityLogs.index(),
                        requiredPermissions: ['activity-logs.read'],
                    },
                ],
            },
        ],
    },
];
```

### Cara Menu Difilter

Hook `useFilteredMainNavItems` membaca `auth.permissions` dari Inertia props dan filter secara rekursif:

- Menu muncul jika user punya **minimal 1** dari `requiredPermissions`
- Jika menu punya `children`, menu tetap muncul walau parent tidak punya permission (selama ada child yang visible)

**File:** `resources/js/hooks/use-filtered-main-nav-items.ts`

### Cara Cek Permission di Frontend

Gunakan hook `usePermissions()`:

```typescript
import { usePermissions } from '@/hooks/use-permissions';

const { can, hasPermission, hasAnyPermission, hasAllPermissions } =
    usePermissions();

// Single permission
if (can('users.create')) {
    /* ... */
}

// Any of multiple
if (can(['users.read', 'users.update'], 'any')) {
    /* ... */
}

// All of multiple
if (can(['users.read', 'users.update'], 'all')) {
    /* ... */
}
```

---

## Panduan Menambah Fitur Baru

Contoh: Menambah modul **Products** dengan CRUD.

### Step 1: Tambah Permission di Seeder

Edit `database/seeders/PermissionGroupsSeeder.php`:

```php
$service->createGroup('Product Module', [
    'products.create',
    'products.read',
    'products.update',
    'products.delete',
]);

// Assign ke admin
$service->assignGroupToRole($adminRole, 'Product Module');

// Assign ke user (jika perlu)
$service->assignGroupToRole($userRole, 'Product Module');
```

### Step 2: Buat Controller dengan Middleware Permission

```php
// app/Http/Controllers/Management/ProductsController.php
class ProductsController extends Controller
{
    public function __construct()
    {
        $this->middleware('can:products.read')->only(['index', 'fetchData']);
        $this->middleware('can:products.create')->only('store');
        $this->middleware('can:products.update')->only('update');
        $this->middleware('can:products.delete')->only(['destroy', 'bulkDestroy']);
    }

    // index(), fetchData(), store(), update(), destroy() ...
}
```

### Step 3: Daftarkan Route

Edit `routes/web.php`:

```php
// Di dalam Route::prefix('management')->group(...)
Route::prefix('products')->controller(ProductsController::class)->group(function () {
    Route::get('/', 'index')->name('products.index');
    Route::get('/fetch-data', 'fetchData')->name('products.fetch-data');
    Route::post('/', 'store')->name('products.store');
    Route::put('/{product}', 'update')->name('products.update');
    Route::delete('/{product}', 'destroy')->name('products.destroy');
});
```

### Step 4: Tambah Menu di Navigation

Edit `resources/js/config/navigation.ts`:

```typescript
import products from '@/routes/products';

// Di dalam children array pada Management:
{
    title: 'Products',
    href: products.index(),
    requiredPermissions: ['products.read'],
},
```

### Step 5: Buat Page React

Buat file di `resources/js/pages/management/products/index.tsx` dengan pola:

```typescript
import AppLayout from '@/layouts/app-layout';
import ProductTable from './table/product-table';

const breadcrumbs = [
    { title: 'Management', href: '/management' },
    { title: 'Products', href: '/management/products' },
];

Products.layout = (page) => (
    <AppLayout children={page} breadcrumbs={breadcrumbs} title="Management Products" />
);

export default function Products() {
    return <ProductTable />;
}
```

### Step 6: Jalankan Seeder

```bash
php artisan db:seed --class=PermissionGroupsSeeder
```

Atau jika seeder sudah dijalankan sebelumnya, jalankan seeder ulang:

```bash
php artisan db:seed
```

### Step 7: Assign Role ke User (jika perlu)

Di halaman **Management > Access Control > Roles**, edit role yang diinginkan dan checklist permission baru (`products.*`).

---

## Alur Akses Halaman yang Dilindungi

```
User akses /management/users
    │
    ├─ Middleware auth → redirect ke /login jika belum login
    ├─ Middleware verified → redirect ke verify-email jika belum verifikasi
    ├─ Middleware can:users.read → 403 jika tidak punya permission
    │
    ├─ Controller UsersController::index()
    │   └─ return Inertia::render('management/users/index')
    │
    ├─ HandleInertiaRequests sharing auth.permissions ke React
    │
    └─ Frontend:
        ├─ useFilteredMainNavItems filter menu sidebar
        ├─ usePermissions().can('users.read') untuk cek permission di komponen
        └─ DataTable fetch data dari /management/users/fetch-data
```

---

## Key Files Reference

### Saat Menambah Fitur Baru, Edit File Ini:

| File                                          | Yang Perlu Ditambah         |
| --------------------------------------------- | --------------------------- |
| `database/seeders/PermissionGroupsSeeder.php` | Permission group baru       |
| `routes/web.php`                              | Route untuk controller baru |
| `resources/js/config/navigation.ts`           | Menu item baru              |

### Saat Menambah Fitur Baru, Buat File Ini:

| File                                                    | Keterangan                              |
| ------------------------------------------------------- | --------------------------------------- |
| `app/Http/Controllers/Management/XxxController.php`     | Controller dengan middleware permission |
| `app/Services/Management/Xxx/XxxService.php`            | Service untuk logic bisnis              |
| `app/Services/Management/Xxx/XxxDataTableService.php`   | Service untuk DataTable                 |
| `resources/js/pages/management/xxx/index.tsx`           | Halaman utama                           |
| `resources/js/pages/management/xxx/table/xxx-table.tsx` | Komponen tabel                          |
| `resources/js/pages/management/xxx/form/`               | Komponen form (create/edit)             |

---

## Akses Default

| Akun  | Email         | Password | Role  |
| ----- | ------------- | -------- | ----- |
| Admin | admin@app.com | rahasia! | admin |

---

## Perintah Umum

```bash
# Jalankan development server
npm run dev

# Build production
npm run build

# Jalankan seeder
php artisan db:seed

# Migration
php artisan migrate

# Clear cache permission (setelah edit permission/role)
php artisan permission:cache-reset
```
