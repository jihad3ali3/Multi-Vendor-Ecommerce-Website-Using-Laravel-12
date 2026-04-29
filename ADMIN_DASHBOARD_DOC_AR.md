# 🏗️ شرح لوح التحكم الخاص بالأدمن — من الألف للياء

> **المشروع:** Multi-Vendor Ecommerce Website Using Laravel 12
> **التوثيق:** عربي كامل — بنية النظام، الجداول، المخططات، تسلسل العمليات

---

## 📌 الفهرس

1. [البنية العامة للمشروع](#1-البنية-العامة-للمشروع)
2. [نظام المصادقة](#2-نظام-المصادقة-authentication-system)
3. [المسارات Routes](#3-المسارات-routes)
4. [لوحة التحكم الرئيسية Dashboard](#4-لوحة-التحكم-الرئيسية-dashboard)
5. [نظام جداول البيانات DataTables](#5-نظام-جداول-البيانات-datatables)
6. [نظام الفئات الثلاثية](#6-نظام-الفئات-الثلاثية-3-level-category)
7. [نظام المنتجات](#7-نظام-المنتجات-products-system)
8. [نظام الطلبات](#8-نظام-الطلبات-orders-system)
9. [نظام البائعين](#9-نظام-البائعين-vendor-system)
10. [نظام الأدوار والصلاحيات RBAC](#10-نظام-الأدوار-والصلاحيات-rbac)
11. [نظام العروض الخاطفة Flash Sale](#11-نظام-العروض-الخاطفة-flash-sale)
12. [نظام الكوبونات](#12-نظام-الكوبونات-coupons)
13. [نظام الشحن](#13-نظام-الشحن-shipping-rules)
14. [نظام السحب المالي](#14-نظام-السحب-المالي-withdraw-system)
15. [نظام البريد الإلكتروني](#15-نظام-البريد-الإلكتروني-mail-system)
16. [إعدادات الدفع](#16-إعدادات-الدفع-payment-settings)
17. [الإعدادات العامة للموقع](#17-الإعدادات-العامة-للموقع)
18. [وضع الصيانة](#18-وضع-الصيانة-maintenance-mode)
19. [التحقق من الصلاحيات](#19-التحقق-من-الصلاحيات-authorization)
20. [تسلسل العمليات الكاملة End-to-End](#20-تسلسل-العمليات-الكاملة-end-to-end)
21. [ملخص جميع الجداول](#21-ملخص-جميع-الجداول)

---

## 1. البنية العامة للمشروع

المشروع مبني على **Laravel 12** باستخدام نمط **MVC** (Model - View - Controller).

```
app/
├── Http/Controllers/Admin/     ← كل كنترولرات الأدمن
├── Models/                     ← نماذج قاعدة البيانات
├── DataTables/                 ← جداول DataTables الديناميكية
├── Helper/                     ← دوال مساعدة (Mail, helper)
routes/
├── admin.php                   ← كل مسارات الأدمن
database/
└── migrations/                 ← هيكل جداول قاعدة البيانات
resources/
└── views/admin/                ← واجهات لوح التحكم
```

### مخطط طبقات النظام:

```
┌─────────────────────────────────────────────────────┐
│                    المتصفح (Browser)                 │
└──────────────────────────┬──────────────────────────┘
                           │ HTTP Request
┌──────────────────────────▼──────────────────────────┐
│              Middleware (auth:admin)                  │
│         التحقق من جلسة الأدمن في كل طلب             │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                     Routes                           │
│              routes/admin.php                        │
└──────────────────────────┬──────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────┐
│                   Controllers                        │
│         app/Http/Controllers/Admin/                  │
└──────────┬───────────────────────────┬──────────────┘
           │                           │
┌──────────▼──────────┐   ┌────────────▼─────────────┐
│       Models        │   │    DataTables             │
│   app/Models/       │   │    app/DataTables/        │
└──────────┬──────────┘   └──────────────────────────┘
           │
┌──────────▼──────────┐
│    قاعدة البيانات   │
│      MySQL          │
└─────────────────────┘
```

---

## 2. نظام المصادقة (Authentication System)

### 🔑 Guard مستقل للأدمن

المشروع يستخدم **Guard منفصل** للأدمن (غير Guard المستخدمين العاديين):

```
auth:admin  ←  يتحقق من جدول admins
auth:web    ←  يتحقق من جدول users
```

### جدول `admins`:

| العمود | النوع | القيم | الوصف |
|--------|-------|-------|-------|
| id | bigint PK | auto | المعرف الأساسي |
| name | string | - | الاسم الكامل |
| email | string unique | - | البريد الإلكتروني |
| password | string | hashed (bcrypt) | كلمة المرور مشفرة |
| status | enum | approved / banned | حالة الحساب |
| image | string nullable | - | مسار الصورة الشخصية |
| contact | string | - | رقم التواصل |
| address | text | - | العنوان |
| created_by | string | - | id مَن أنشأ هذا الأدمن |
| email_verified_at | timestamp nullable | - | تاريخ تأكيد البريد |
| remember_token | string | - | جلسة "تذكرني" |
| created_at / updated_at | timestamps | - | تواريخ الإنشاء والتعديل |

### 🔄 تسلسل عملية تسجيل الدخول:

```
المتصفح                Laravel                    قاعدة البيانات
   │                      │                             │
   │── GET /admin/login ──▶│                             │
   │                      │── render login view ────────│
   │◀─ صفحة الدخول ────────│                             │
   │                      │                             │
   │── POST /admin/login ─▶│                             │
   │   {email, password}   │── AdminLoginRequest         │
   │                      │   authenticate()            │
   │                      │──── SELECT FROM admins ─────▶│
   │                      │◀─── admin record ───────────│
   │                      │                             │
   │                      │── check credentials ────────│
   │                      │── session()->regenerate()   │
   │                      │                             │
   │                      │── هل status == 'banned'؟    │
   │                      │    ├── نعم: logout + redirect /
   │                      │    └── لا: redirect dashboard
   │◀─ redirect dashboard ─│                             │
```

### 🔄 تسلسل التحقق من الجلسة في كل طلب:

```
كل Request إلى /admin/*
        │
        ▼
middleware: auth:admin
        │
        ├── جلسة موجودة؟
        │       ├── نعم ──▶ تنفيذ الكنترولر المطلوب
        │       └── لا  ──▶ redirect /admin/login
        │
        (داخل AdminDashboardController)
        ▼
فحص إضافي: هل status == 'banned'؟
        ├── نعم ──▶ logout + session invalidate + redirect /
        └── لا  ──▶ عرض الـ Dashboard
```

### الكنترولرات المتعلقة بالمصادقة:

| الكنترولر | المسار | الوظيفة |
|-----------|--------|---------|
| `AuthenticatedSessionController` | `Admin/Auth/` | تسجيل الدخول والخروج |
| `PasswordResetLinkController` | `Admin/Auth/` | إرسال رابط إعادة تعيين كلمة المرور |
| `NewPasswordController` | `Admin/Auth/` | إعادة تعيين كلمة المرور |
| `ConfirmablePasswordController` | `Admin/Auth/` | تأكيد كلمة المرور |
| `EmailVerificationPromptController` | `Admin/Auth/` | طلب تأكيد البريد |
| `VerifyEmailController` | `Admin/Auth/` | تأكيد البريد الإلكتروني |
| `EmailVerificationNotificationController` | `Admin/Auth/` | إعادة إرسال رابط التأكيد |
| `PasswordController` | `Admin/Auth/` | تغيير كلمة المرور |

---

## 3. المسارات (Routes)

ملف `routes/admin.php` يحتوي على مجموعتين رئيسيتين:

### المجموعة الأولى — للزوار (guest:admin):

```php
Route::group(['middleware' => 'guest:admin', 'prefix' => 'admin', 'as' => 'admin.'], function () {
    // يُمنع على المسجّلين دخول هذه المسارات
    GET  /admin/login                → تسجيل الدخول
    POST /admin/login                → معالجة تسجيل الدخول
    GET  /admin/forgot-password      → نموذج نسيت كلمة المرور
    POST /admin/forgot-password      → إرسال رابط الاسترجاع
    GET  /admin/reset-password/{token} → نموذج إعادة تعيين
    POST /admin/reset-password       → معالجة إعادة التعيين
});
```

### المجموعة الثانية — للمسجلين (auth:admin):

```php
Route::group(['middleware' => 'auth:admin', 'prefix' => 'admin', 'as' => 'admin.'], function () {
    // كل مسارات لوح التحكم هنا
});
```

### جدول المسارات الكاملة:

| المسار | الكنترولر | الغرض |
|--------|-----------|-------|
| `GET /admin/dashboard` | AdminDashboardController | الصفحة الرئيسية |
| `CRUD /admin/category` | CategoryController | إدارة الفئات |
| `CRUD /admin/sub-category` | SubCategoryController | إدارة الفئات الفرعية |
| `CRUD /admin/child-category` | ChildCategoryController | إدارة الفئات الثالثية |
| `CRUD /admin/brand` | BrandController | إدارة الماركات |
| `CRUD /admin/product` | ProductController | إدارة منتجات الأدمن |
| `CRUD /admin/vendor-product` | VendorProductController | إدارة منتجات البائعين |
| `CRUD /admin/product/variant` | ProductVariantController | متغيرات المنتج |
| `CRUD /admin/product/variant-item` | ProductVariantItemController | عناصر المتغيرات |
| `CRUD /admin/product/image-gallery` | ProductImageGalleryController | معرض صور المنتج |
| `CRUD /admin/all-orders` | AllOrdersController | جميع الطلبات |
| `POST /admin/change-order-status` | AllOrdersController | تغيير حالة الطلب |
| `CRUD /admin/transaction` | TransactionController | المعاملات المالية |
| `CRUD /admin/coupon` | CouponController | أكواد الخصم |
| `CRUD /admin/flash-sale` | FlashSaleController | العروض الخاطفة |
| `CRUD /admin/shipping-rule` | ShippingRuleController | قواعد الشحن |
| `CRUD /admin/vendor-request` | VendorRequestController | طلبات التسجيل كبائع |
| `CRUD /admin/approved-vendors` | ApprovedVendorController | البائعين المعتمدين |
| `CRUD /admin/manage-admin` | ManageAdminController | إدارة الأدمنيين |
| `CRUD /admin/manage-user` | ManageUserController | إدارة المستخدمين |
| `CRUD /admin/role` | AdminRoleController | إدارة الأدوار |
| `CRUD /admin/permission` | PermissionController | إدارة الصلاحيات |
| `CRUD /admin/role-in-permission` | RoleInPermissionController | ربط أدوار بصلاحيات |
| `CRUD /admin/slider` | SliderController | إدارة السلايدر |
| `CRUD /admin/settings` | SettingsController | إعدادات الموقع العامة |
| `CRUD /admin/payment-settings` | PaymentSettingsController | إعدادات الدفع |
| `CRUD /admin/smtp-config` | SMTPConfigController | إعدادات البريد |
| `CRUD /admin/withdraw-method` | WithdrawMethodController | طرق السحب |
| `CRUD /admin/withdraw-request` | WithdrawRequestController | طلبات السحب |
| `CRUD /admin/review` | ReviewController | إدارة التقييمات |
| `CRUD /admin/ad` | AdController | الإعلانات |
| `CRUD /admin/news-letter` | AdminNewsLetterController | المشتركون بالنشرة |
| `CRUD /admin/maintainance` | MaintainanceController | وضع الصيانة |
| `CRUD /admin/about-page` | AboutPageController | صفحة "من نحن" |
| `CRUD /admin/term-page` | TermConditionController | الشروط والأحكام |
| `CRUD /admin/vendor-condition` | VendorConditionController | شروط البائعين |
| `CRUD /admin/top-category` | TopCategorySectionController | قسم أبرز الفئات |
| `CRUD /admin/single-category` | SingleCategorySectionController | قسم فئة مفردة |
| `CRUD /admin/footer-section` | FooterSectionController | إعدادات الفوتر |
| `CRUD /admin/profile` | AdminProfileController | ملف الأدمن الشخصي |

> **CRUD** = GET (index) + GET (create) + POST (store) + GET (edit) + PUT (update) + DELETE (destroy)

---

## 4. لوحة التحكم الرئيسية (Dashboard)

### `AdminDashboardController@index` — تسلسل جمع الإحصائيات:

```
┌─────────────────────────────────────────────────────────┐
│           AdminDashboardController@index                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. فحص أمني: هل status == 'banned'؟                    │
│     └── نعم ──▶ logout + redirect                       │
│                                                         │
│  2. جلب order_products التابعة للأدمن                   │
│     WHERE vendor_id = 0                                  │
│                                                         │
│  3. حساب الأرباح (من جدول orders):                      │
│     ├── todayEarnings   → اليوم + delivered + paid      │
│     ├── totalEarnings   → الكل + delivered + paid       │
│     ├── monthlyEarnings → الشهر الحالي                  │
│     └── yearlyEarnings  → السنة الحالية                 │
│                                                         │
│  4. حساب الطلبات:                                       │
│     ├── todaysOrders          → طلبات اليوم             │
│     ├── todaysPendingOrders   → معلقة اليوم             │
│     ├── totalOrders           → إجمالي الطلبات          │
│     ├── totalPendingOrders    → إجمالي المعلقة           │
│     ├── totalCompletedOrders  → إجمالي المسلّمة          │
│     └── totalCancelledOrders  → إجمالي الملغية          │
│                                                         │
│  5. إحصائيات متنوعة:                                    │
│     ├── totalProducts    → منتجات الأدمن المعتمدة       │
│     ├── totalReviews     → التقييمات النشطة             │
│     ├── totalBrands      → الماركات                     │
│     ├── totalCategories  → الفئات الرئيسية              │
│     ├── totalSubscribers → المشتركون في النشرة          │
│     ├── totalVendors     → البائعون النشطون             │
│     ├── totalUsers       → المستخدمون العاديون          │
│     ├── totalAdmins      → جميع الأدمنيين              │
│     ├── totalVendorReq   → طلبات الانضمام كبائع         │
│     └── ad_running       → الإعلانات النشطة             │
│                                                         │
│  6. return view('admin.dashboard', compact(...))        │
└─────────────────────────────────────────────────────────┘
```

### منطق حساب أرباح الأدمن:

```php
// الأرباح تُحسب فقط للطلبات التي:
// 1. order_status = 'delivered'   (تم التسليم)
// 2. payment_status = 1           (تم الدفع)
// 3. تحتوي على منتجات vendor_id = 0  (منتجات الأدمن فقط)

$totalEarnings = Order::where('order_status', 'delivered')
    ->where('payment_status', 1)
    ->whereHas('orderProducts', function($query){
        $query->where('vendor_id', 0);
    })->sum('sub_total');
```

---

## 5. نظام جداول البيانات (DataTables)

المشروع يستخدم **Yajra DataTables** لعرض البيانات بشكل ديناميكي مع:
- Pagination (تصفح الصفحات)
- Real-time Search (بحث فوري)
- Sortable Columns (ترتيب الأعمدة)
- بدون reload للصفحة (AJAX)

### قائمة الـ DataTables المتاحة:

| DataTable Class | الاستخدام |
|-----------------|-----------|
| `CategoryDataTable` | قائمة الفئات |
| `SubCategoryDataTable` | قائمة الفئات الفرعية |
| `ChildCategoryDataTable` | قائمة الفئات الثالثية |
| `BrandDataTable` | قائمة الماركات |
| `SliderDataTable` | قائمة السلايدرات |
| `AdminProductDataTable` | قائمة منتجات الأدمن |
| `AdminVendorProductDataTable` | قائمة منتجات البائعين |
| `AdminProductImageGalleryDataTable` | معرض صور المنتج |
| `AdminProductVariantDataTable` | متغيرات المنتج |
| `AdminProductVariantItemDataTable` | عناصر المتغيرات |
| `AdminAllOrdersDataTable` | قائمة الطلبات |
| `AdminTransactionDataTable` | قائمة المعاملات المالية |
| `AdminCouponDataTable` | قائمة الكوبونات |
| `AdminFlashSaleItemDataTable` | منتجات العروض الخاطفة |
| `AdminShippingRuleDataTable` | قواعد الشحن |
| `VendorRequestsDataTable` | طلبات البائعين |
| `ApprovedVendorDataTable` | البائعون المعتمدون |
| `ManageAdminsDataTable` | قائمة الأدمنيين |
| `ManageUsersDataTable` | قائمة المستخدمين |
| `AdminRoleDataTable` | قائمة الأدوار |
| `AdminPermissionDataTable` | قائمة الصلاحيات |
| `RoleInPermissionDataTable` | الأدوار مع صلاحياتها |
| `AdminReviewsDataTable` | قائمة التقييمات |
| `AdminWithdrawMethodDataTable` | طرق السحب |
| `AdminWithdrawRequestDataTable` | طلبات السحب |
| `SubscribersDataTable` | المشتركون في النشرة |

### تسلسل عمل الـ DataTable:

```
1. Controller@index(XxxDataTable $dataTable)
        │
        ▼
2. $dataTable->render('admin.xxx.index')
        │
        ▼
3. View يُعرض مع جدول فارغ + JavaScript
        │
        ▼
4. JavaScript يطلب البيانات عبر AJAX (GET)
        │
        ▼
5. DataTable Class يستعلم قاعدة البيانات
        │
        ▼
6. إرجاع JSON → عرض في الجدول
```

---

## 6. نظام الفئات الثلاثية (3-Level Category)

### الهرم:

```
Category (الفئة الرئيسية)
  ├── SubCategory (الفئة الفرعية)
  │     └── ChildCategory (الفئة الثالثية)
  └── SubCategory
        └── ChildCategory
```

### هيكل الجداول:

**جدول `categories`:**

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| name | string | اسم الفئة |
| slug | string | الرابط المُنسّق |
| icon | string | أيقونة الفئة |
| status | boolean | نشط/معطل |

**جدول `sub_categories`:**

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| category_id | int FK | ربط بالفئة الرئيسية |
| name | string | الاسم |
| slug | string | الرابط |
| status | boolean | نشط/معطل |

**جدول `child_categories`:**

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| sub_category_id | int FK | ربط بالفئة الفرعية |
| name | string | الاسم |
| slug | string | الرابط |
| status | boolean | نشط/معطل |

### التحقق عند الحذف:

```
حذف Category:
    └── هل هناك sub_categories مرتبطة؟
        ├── نعم ──▶ رسالة خطأ "يجب حذف الفئات الفرعية أولاً"
        └── لا  ──▶ حذف الفئة
```

### التحديث الديناميكي (AJAX) عند اختيار الفئة في نموذج المنتج:

```
اختيار Category
        │
        ▼ AJAX GET /admin/product/get-sub-categories?category_id=X
        │
        ▼ ProductController@getSubCategories
        │ SubCategory::where(['category_id'=>X, 'status'=>1])->get()
        │
        ▼ JSON Response
        │
        ▼ تحديث قائمة SubCategory تلقائياً
        │
        ▼ اختيار SubCategory → AJAX → getChildCategories
        │
        ▼ تحديث قائمة ChildCategory
```

---

## 7. نظام المنتجات (Products System)

### جدول `products`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| name | string | اسم المنتج |
| slug | string | الرابط المُنسّق |
| thumb_image | text | مسار الصورة الرئيسية |
| vendor_id | int | **0** = منتج الأدمن، **غير 0** = منتج بائع |
| admin_id | int nullable | id الأدمن المنشئ |
| category_id | int | الفئة الرئيسية |
| sub_category_id | int nullable | الفئة الفرعية |
| child_category_id | int nullable | الفئة الثالثية |
| brand_id | int | الماركة |
| qty | int | الكمية المتاحة |
| short_description | text | وصف مختصر |
| long_description | text | وصف تفصيلي |
| video_link | text nullable | رابط الفيديو |
| sku | string nullable | رمز المنتج |
| price | double | السعر الأساسي |
| offer_price | double nullable | سعر العرض |
| offer_start_date | date nullable | بداية العرض |
| offer_end_date | date nullable | نهاية العرض |
| product_type | string | نوع المنتج |
| status | boolean | نشط/معطل |
| is_approved | int (0/1) | **1** = معتمد، **0** = بانتظار الموافقة |
| seo_title | string nullable | عنوان SEO |
| seo_description | text nullable | وصف SEO |

### هرم المنتج الكامل:

```
Product
├── ProductImageGallery      ← معرض الصور الإضافية
│   (product_image_galleries)
│
├── ProductVariant           ← المتغيرات (اللون، المقاس...)
│   (product_variants)
│   └── ProductVariantItem   ← عناصر كل متغير (أحمر، L، XL...)
│       (product_variant_items)
│
└── FlashSaleItem            ← ارتباط بالعروض الخاطفة
    (flash_sale_items)
```

### تسلسل إضافة منتج جديد:

```
1. GET  /admin/product/create
   ProductController@create
   └── يجلب: brands (status=1) + categories (status=1)
   └── يعرض نموذج الإنشاء

2. POST /admin/product
   ProductController@store
   ├── Validation:
   │   ├── name, thumb_image, category_id, brand_id (required)
   │   ├── qty, short_description, long_description (required)
   │   ├── price (required, numeric)
   │   └── product_type (required)
   │
   ├── رفع الصورة:
   │   ├── rand() + extension → اسم عشوائي فريد
   │   ├── حفظ في /public/uploads/
   │   └── حفظ المسار في DB
   │
   ├── إنشاء Product:
   │   ├── vendor_id = 0       (منتج الأدمن)
   │   ├── admin_id = Auth::id()
   │   └── is_approved = 1    (معتمد تلقائياً)
   │
   └── save() → redirect product.index + notyf success
```

### منطق حذف المنتج (محمي بعدة طبقات):

```
ProductController@destroy:
├── Layer 1: Auth::user()->id != 1 ──▶ abort(404)
│           (فقط Super Admin يحذف)
├── Layer 2: هل هناك ProductVariants؟
│           ├── نعم ──▶ رسالة خطأ
│           └── لا  ──▶ متابعة
├── Layer 3: حذف الصورة من السيرفر
│           File::exists → File::delete
└── Layer 4: $product->delete()
```

---

## 8. نظام الطلبات (Orders System)

### جدول `orders`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| invoice_id | string | رقم الفاتورة |
| transaction_id | string | معرف المعاملة المالية |
| user_id | int | المستخدم الذي أجرى الطلب |
| sub_total | double | المجموع قبل الشحن |
| amount | double | المبلغ الإجمالي (مع الشحن) |
| currency_name | string | اسم العملة (USD, SAR...) |
| currency_icon | string | رمز العملة ($, ر.س) |
| product_qty | int | إجمالي عدد القطع |
| payment_method | string | طريقة الدفع (paypal, stripe...) |
| payment_status | int | 1 = مدفوع، 0 = غير مدفوع |
| order_address | text (JSON) | عنوان التوصيل |
| shipping_method | text (JSON) | معلومات الشحن |
| coupon | text (JSON) | الكوبون المستخدم |
| order_status | string | pending / processing / shipped / delivered / canceled |

### جدول `order_products`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| order_id | int FK | ربط بجدول orders |
| transaction_id | string | نفس transaction_id في orders |
| vendor_id | int | **0** = منتج الأدمن، **غير 0** = بائع |
| product_id | int | المنتج |
| qty | int | الكمية |
| price | double | السعر |

### دورة حياة الطلب:

```
pending (معلق)
    │
    ▼ الأدمن يغير الحالة
processing (قيد المعالجة)
    │
    ▼
shipped (تم الشحن)
    │
    ▼
delivered (تم التسليم) ──▶ تُحسب في الأرباح
    │
    (أو في أي وقت)
    ▼
canceled (ملغي)
```

### تسلسل تغيير حالة الطلب (AJAX):

```
Admin يختار حالة جديدة من القائمة المنسدلة
        │
        ▼ POST /admin/change-order-status
        │ {order_id, order_status}
        │
        ▼ AllOrdersController@changeOrderStatus
        │ $order = Order::findOrFail($request->order_id)
        │ $order->order_status = $request->order_status
        │ $order->save()
        │
        ▼ JSON Response: {status: 'success', message: '...'}
        │
        ▼ JavaScript يحدث الواجهة بدون reload
```

### تسلسل حذف الطلب:

```
AllOrdersController@destroy($id):
    1. Order::findOrFail($id)
    2. Transaction::where('transaction_id', $order->transaction_id)->first()
    3. $transaction->delete()
    4. $order->delete()
    5. notyf success + response JSON
```

---

## 9. نظام البائعين (Vendor System)

### جدول `users` — الحقول الخاصة بالبائعين:

| الحقل | النوع | القيم | الوصف |
|-------|-------|-------|-------|
| role | enum | admin / vendor / user | الدور الحالي |
| vendor_request | boolean | 0 / 1 | هل قدّم طلب انضمام؟ |
| document | text nullable | - | وثيقة الهوية/السجل التجاري |
| is_user | boolean | 0 / 1 | هل مستخدم عادي؟ |
| is_vendor | boolean | 0 / 1 | هل بائع؟ |
| user_status | enum | active / inactive / banned / is_vendor | حالة المستخدم |
| vendor_status | enum | approved / pending / rejected / banned / is_user | حالة طلب البائع |
| banner | text nullable | - | صورة بانر متجر البائع |
| desc | text nullable | - | وصف المتجر |
| fb_link / tw_link / insta_link / tiktok_link / yt_link | text nullable | - | روابط التواصل الاجتماعي |

### تسلسل دورة طلب البائع الكاملة:

```
المستخدم العادي يقدم طلب انضمام
│   vendor_request = 1
│   document = [رفع ملف]
│
▼
جدول users:
│   role = 'user'
│   vendor_request = 1
│   vendor_status = 'pending'
│
▼
الأدمن يفتح /admin/vendor-request
│   VendorRequestController@index
│   └── VendorRequestsDataTable (يعرض من لديهم vendor_request=1)
│
▼
الأدمن يضغط Edit → /admin/vendor-request/{id}/edit
│   عرض تفاصيل المستخدم + نموذج تغيير الحالة
│
▼
الأدمن يختار الحالة → PUT /admin/vendor-request/{id}
│
├── vendor_status = 'approved'
│       role = 'vendor'
│       is_vendor = 1
│       is_user = 0
│       user_status = 'is_vendor'
│
├── vendor_status = 'pending'
│       role = 'user'
│       is_vendor = 0
│       is_user = 1
│       user_status = 'active'
│
├── vendor_status = 'rejected'
│       role = 'user'
│       is_vendor = 0
│       is_user = 1
│       user_status = 'active'
│
└── vendor_status = 'banned'
        role = 'user'
        is_vendor = 0
        is_user = 1
        user_status = 'banned'
│
▼
MailHelper::setMailConfig()   ← تحميل إعدادات SMTP من DB
Mail::to($user->email)->send(new VendorStatus($user))   ← إرسال إشعار
│
▼
redirect → vendor-request.index + رسالة نجاح
```

---

## 10. نظام الأدوار والصلاحيات (RBAC)

### المكتبة: **Spatie Laravel Permission**

### مخطط الجداول:

```
┌──────────┐    ┌─────────────────────┐    ┌─────────────┐
│  admins  │    │   model_has_roles   │    │    roles    │
├──────────┤    ├─────────────────────┤    ├─────────────┤
│ id       │───▶│ model_id            │◀───│ id          │
│ name     │    │ model_type          │    │ name        │
│ ...      │    │ role_id             │    │ guard_name  │
└──────────┘    └─────────────────────┘    └──────┬──────┘
                                                   │
                                    ┌──────────────▼──────────────┐
                                    │      role_has_permissions    │
                                    ├─────────────────────────────┤
                                    │ role_id                     │
                                    │ permission_id               │
                                    └──────────────┬──────────────┘
                                                   │
                                    ┌──────────────▼──────────────┐
                                    │         permissions          │
                                    ├─────────────────────────────┤
                                    │ id                          │
                                    │ name                        │
                                    │ group_name                  │
                                    │ guard_name                  │
                                    └─────────────────────────────┘
```

### تسلسل إنشاء دور وتعيين صلاحيات:

```
الخطوة 1: إنشاء دور
    POST /admin/role
    AdminRoleController@store
    └── Role::create(['name' => $request->name])

الخطوة 2: تعيين صلاحيات للدور
    POST /admin/role-in-permission
    RoleInPermissionController@store
    ├── validate: role_id موجود في roles
    ├── validate: permissions[] كلها موجودة في permissions
    ├── $role = Role::findOrFail($request->role_id)
    ├── $permissions = Permission::whereIn('id', $request->permissions)->get()
    └── $role->syncPermissions($permissions)
        └── يحذف الصلاحيات القديمة ويضيف الجديدة

الخطوة 3: تعيين الدور لأدمن
    (يتم عبر model_has_roles)
```

### الصلاحيات مُجمّعة بـ `group_name`:

```php
// جلب مجموعات الصلاحيات للعرض في نموذج تعيين الصلاحيات
$permission_group_names = Permission::distinct()->pluck('group_name');
$getPermissionByGroupNames = Permission::whereIn('group_name', $permission_group_names)->get();
```

---

## 11. نظام العروض الخاطفة (Flash Sale)

### جدول `flash_sales`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف (سجل واحد فقط = id:1) |
| end_date | date | تاريخ انتهاء العرض |

### جدول `flash_sale_items`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| product_id | int FK | المنتج المشمول بالعرض |
| flash_sale_id | int | ربط بـ flash_sales (دائماً = 1) |
| show_at_home | boolean | هل يظهر في الصفحة الرئيسية؟ |
| status | boolean | نشط/معطل |

### منطق `FlashSaleController@update`:

```
PUT /admin/flash-sale/{id}
        │
        ├── request.date == 'date'
        │       └── تحديث end_date في جدول flash_sales
        │
        ├── request.flash_products == 'flash_products'
        │       ├── validate: product_id[] + status
        │       ├── للكل product_id:
        │       │       ├── هل موجود مسبقاً في flash_sale_items؟
        │       │       │       ├── نعم ──▶ تخطي
        │       │       │       └── لا  ──▶ إضافة جديد
        │       └── إرجاع عدد المنتجات المضافة
        │
        └── request.item_edit == 'item_edit'
                └── تعديل show_at_home + status لعنصر موجود
```

---

## 12. نظام الكوبونات (Coupons)

### جدول `coupons`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| name | string | اسم الكوبون |
| code | string unique | الكود المستخدم (مثل: SAVE20) |
| quantity | numeric | عدد الكوبونات المتاحة إجمالاً |
| max_use | numeric | أقصى استخدام لكل مستخدم |
| start_date | date | تاريخ بداية الصلاحية |
| end_date | date | تاريخ انتهاء الصلاحية |
| discount_type | string | percentage (نسبة) / fixed (ثابت) |
| discount | numeric | قيمة الخصم |
| status | boolean | نشط/معطل |
| total_used | int | عدد مرات الاستخدام الفعلي |

### قواعد التحقق عند الإنشاء:

```
- name: required, string, max:255
- code: required, string, max:255, UNIQUE في جدول coupons
- quantity: required, numeric
- max_use: required, numeric
- start_date: required
- end_date: required
- discount_type: required
- discount: required
- status: required, boolean
```

### قاعدة التحقق عند التعديل (code unique مع استثناء الحالي):

```php
'code' => ['required', 'string', 'max:255', 'unique:coupons,code,'.$id],
```

---

## 13. نظام الشحن (Shipping Rules)

### جدول `shipping_rules`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| name | string | اسم قاعدة الشحن |
| type | string | نوع الشحن (flat_rate / free_shipping / ...) |
| min_cost | numeric nullable | الحد الأدنى لقيمة الطلب |
| cost | numeric | تكلفة الشحن |
| status | boolean | نشط/معطل |

---

## 14. نظام السحب المالي (Withdraw System)

### جدول `withdraw_methods`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| name | string | اسم الطريقة |
| instructions | text | تعليمات الاستخدام |
| status | boolean | نشط/معطل |

### جدول `withdraw_requests`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| vendor_id | int | البائع الطالب |
| withdraw_amount | double | المبلغ المطلوب سحبه |
| withdraw_charge | double | رسوم السحب |
| total_earnings | double | إجمالي أرباح البائع |
| current_balance | double | الرصيد المتبقي بعد السحب |
| status | enum | pending / paid / decline |

### منطق حساب الرصيد عند تغيير الحالة إلى "paid":

```
WithdrawRequestController@update:

إذا (status الجديد == 'paid') && (status الحالي != 'paid'):
    │
    ├── هل هناك سحوبات سابقة لنفس البائع (id أصغر)؟
    │       │
    │       ├── نعم: أخذ current_balance من آخر سحب
    │       │       current_balance = previousBalance - (amount + charge)
    │       │
    │       └── لا (أول سحب):
    │               current_balance = total_earnings - (amount + charge)
    │
    └── تحديث current_balance في السجل
```

---

## 15. نظام البريد الإلكتروني (Mail System)

### جدول `smtp_configs`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| email | string | بريد المُرسِل |
| host | string | خادم SMTP (مثل: smtp.gmail.com) |
| username | string | اسم المستخدم |
| password | string | كلمة المرور |
| port | int | المنفذ (587 = TLS، 465 = SSL) |
| encryption | string | tls / ssl |

### `MailHelper` — كيف يعمل:

```
MailHelper::setMailConfig()
        │
        ▼
SMTPConfig::first()   ← جلب الإعدادات من قاعدة البيانات
        │
        ▼
config(['mail.mailers.smtp' => [
    'host'       => $smtpConfig->host,
    'port'       => $smtpConfig->port,
    'encryption' => $smtpConfig->encryption,
    'username'   => $smtpConfig->username,
    'password'   => $smtpConfig->password,
    'from'       => ['address' => $smtpConfig->email]
]])
        │
        ▼
Mail::to($user->email)->send(new VendorStatus($user))
```

---

## 16. إعدادات الدفع (Payment Settings)

### جدول `payment_settings` (نظام key-value):

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| key | string | اسم الإعداد |
| value | text | قيمة الإعداد |

### مثال على البيانات:

```
key                    | value
-----------------------|------------------
paypal_mode            | sandbox
paypal_currency        | USD
paypal_rate            | 1
paypal_client_id       | AXxx...
paypal_client_secret   | EKxx...
paypal_app_id          | APP-xxx
stripe_status          | active
stripe_currency        | USD
stripe_rate            | 1
stripe_publish_key     | pk_test_xxx
stripe_client_secret   | sk_test_xxx
ssl_mode               | sandbox
ssl_status             | active
ssl_currency           | BDT
ssl_store_id           | xxxxx
store_pass             | xxxxx
```

### تسلسل تحديث إعدادات PayPal:

```
PUT /admin/payment-settings/{id}
    {payment_method: 'paypal', paypal_mode: '...', ...}
        │
        ▼
validate الحقول المطلوبة
        │
        ▼
foreach ($validatedData as $key => $value):
    PaymentSettings::updateOrCreate(['key' => $key], ['value' => $value])
        │
        ▼
Artisan::call('config:clear')   ← تطبيق الإعدادات فوراً
        │
        ▼
notyf success + redirect back
```

---

## 17. الإعدادات العامة للموقع

### جدول `settings` (سجل واحد دائماً id=1):

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف (دائماً = 1) |
| site_name | string | اسم الموقع |
| layout | string | نوع التصميم/القالب |
| contact_email | string | البريد الرسمي |
| contact_phone | string | رقم الهاتف |
| contact_address | text | العنوان |
| map | text nullable | رابط خريطة Google |
| currency_name | string | اسم العملة (USD, SAR...) |
| currency_icon | string | رمز العملة ($, ر.س) |
| time_zone | string | المنطقة الزمنية |

### الآلية المستخدمة:

```php
// SettingsController@update
Settings::updateOrCreate(
    ['id' => 1],       // إذا موجود: حدّث | إذا غير موجود: أنشئ
    [
        'site_name' => $request->site_name,
        // ...
    ]
);
```

---

## 18. وضع الصيانة (Maintenance Mode)

### جدول `maintainances`:

| الحقل | النوع | الوصف |
|-------|-------|-------|
| id | bigint PK | المعرف |
| mode | enum (on/off) | حالة الصيانة |
| secret_key | string | المفتاح السري للوصول أثناء الصيانة |
| down_url | string nullable | URL الوصول عبر المفتاح |

### تسلسل تفعيل/إيقاف الصيانة:

```
POST /admin/maintainance
    {mode: 'on', secret_key: 'mykey123'}
        │
        ▼
validate: mode (in:on,off) + secret_key (required)
        │
        ▼
Maintainance::updateOrCreate(['id' => 1], [...])
        │
        ├── mode == 'on':
        │       Artisan::call('down', [
        │           '--secret' => 'mykey123',
        │           '--redirect' => '/mykey123'
        │       ])
        │       ← الموقع الآن في وضع الصيانة
        │       ← يمكن الوصول عبر: /mykey123
        │
        └── mode == 'off':
                Artisan::call('up')
                ← الموقع عاد للعمل الطبيعي
```

---

## 19. التحقق من الصلاحيات (Authorization)

### حماية Super Admin (id=1):

في عدة أماكن، هناك **حماية صارحة** تمنع الأدمنيين الفرعيين من تنفيذ عمليات حساسة:

```php
// الحالات التي تتطلب Super Admin فقط:

// 1. حذف أدمن آخر
ManageAdminController@destroy:
    if (Auth::user()->id != 1) { abort(404); }

// 2. حذف منتج
ProductController@destroy:
    if (Auth::user()->id != 1) { abort(404); }

// 3. حذف طلب بائع
VendorRequestController@destroy:
    if (Auth::user()->id != 1) { abort(404); }

// 4. حذف سلايدر
SliderController@destroy:
    if (Auth::user()->id != 1) { abort(404); }

// 5. تغيير حالة أدمن (approved/banned)
ManageAdminController@update:
    if (Auth::user()->id == '1') {
        $admin->status = $request->status;
    }

// 6. منع تعديل/حذف الأدمن رقم 1
ManageAdminController@edit:
    if($id == 1){ abort(404); }
```

### طبقات الحماية في النظام:

```
طبقة 1: middleware auth:admin
         └── هل الجلسة صالحة؟

طبقة 2: AdminDashboardController
         └── هل status != 'banned'؟

طبقة 3: RBAC (Spatie)
         └── هل الأدمن لديه الصلاحية المطلوبة؟

طبقة 4: Business Logic
         └── هل Auth::user()->id == 1 للعمليات الحساسة؟
```

---

## 20. تسلسل العمليات الكاملة (End-to-End)

### 🛒 مثال 1: دورة الطلب الكاملة

```
المستخدم يشتري من الموقع
        │
        ▼
إنشاء Order + OrderProducts + Transaction
        │
        ▼
الأدمن يفتح /admin/all-orders
AllOrdersController@index
AdminAllOrdersDataTable → عرض كل الطلبات
        │
        ▼
الأدمن يضغط على طلب → show($id)
Order::findOrFail($id)
عرض تفاصيل الطلب كاملة
        │
        ▼
الأدمن يغير الحالة (pending → processing → shipped → delivered)
POST /admin/change-order-status
{order_id, order_status}
        │
        ▼
$order->order_status = $request->order_status → save()
        │
        ▼
الطلب الآن "delivered" → يُضاف إلى أرباح الأدمن
```

### 🏪 مثال 2: دورة البائع الكاملة

```
1. المستخدم يسجل في الموقع
   (role='user', is_user=1)

2. يقدم طلب الانضمام كبائع
   (vendor_request=1, document=[ملف], vendor_status='pending')

3. الأدمن يراجع الطلب من /admin/vendor-request

4. الأدمن يقبل الطلب
   → role='vendor', is_vendor=1, vendor_status='approved'
   → إرسال بريد للمستخدم

5. البائع يسجل دخول ويضيف منتجاته
   (products.vendor_id = user.id, is_approved=0)

6. الأدمن يراجع المنتجات من /admin/vendor-product
   يوافق عليها: is_approved=1

7. المنتجات تظهر في الموقع للعملاء

8. العميل يشتري → order_products.vendor_id = vendor.id

9. البائع يطلب سحب أرباحه
   /vendor/withdraw-request (من لوح تحكم البائع)

10. الأدمن يراجع /admin/withdraw-request
    يغير الحالة إلى 'paid'
    → يتم حساب current_balance تلقائياً
```

### 📦 مثال 3: دورة المنتج مع المتغيرات

```
1. إنشاء المنتج
   POST /admin/product → ProductController@store
   → product.id = 5

2. إضافة صور إضافية
   POST /admin/product/image-gallery
   → product_image_galleries: {product_id=5, image='...'}

3. إضافة متغير (مثلاً: اللون)
   POST /admin/product/variant/store
   → product_variants: {product_id=5, name='Color'}

4. إضافة عناصر للمتغير
   POST /admin/product/variant-item/store
   → product_variant_items: {variant_id=X, name='Red', price=100}
   → product_variant_items: {variant_id=X, name='Blue', price=110}

5. إضافة المنتج للعرض الخاطف
   PUT /admin/flash-sale/1
   {flash_products: 'flash_products', product_id: [5]}
   → flash_sale_items: {product_id=5, flash_sale_id=1}
```

---

## 21. ملخص جميع الجداول

| الجدول | الغرض | الكنترولر المسؤول |
|--------|-------|------------------|
| `admins` | حسابات الأدمن | ManageAdminController |
| `admin_roles` | أدوار الأدمن المخصصة | AdminRoleController |
| `roles` (Spatie) | أدوار RBAC | AdminRoleController |
| `permissions` (Spatie) | صلاحيات RBAC | PermissionController |
| `role_has_permissions` | ربط الأدوار بالصلاحيات | RoleInPermissionController |
| `model_has_roles` | ربط الأدمنيين بالأدوار | RoleInPermissionController |
| `model_has_permissions` | صلاحيات مباشرة | RoleInPermissionController |
| `users` | المستخدمون والبائعون | ManageUserController |
| `products` | المنتجات | ProductController / VendorProductController |
| `product_variants` | متغيرات المنتج | ProductVariantController |
| `product_variant_items` | عناصر المتغيرات | ProductVariantItemController |
| `product_image_galleries` | معرض صور المنتجات | ProductImageGalleryController |
| `categories` | الفئات الرئيسية | CategoryController |
| `sub_categories` | الفئات الفرعية | SubCategoryController |
| `child_categories` | الفئات الثالثية | ChildCategoryController |
| `brands` | الماركات/العلامات التجارية | BrandController |
| `orders` | الطلبات | AllOrdersController |
| `order_products` | منتجات الطلب | AllOrdersController |
| `transactions` | المعاملات المالية | TransactionController |
| `coupons` | أكواد الخصم | CouponController |
| `flash_sales` | إعداد العرض الخاطف | FlashSaleController |
| `flash_sale_items` | منتجات العرض الخاطف | FlashSaleController |
| `shipping_rules` | قواعد الشحن | ShippingRuleController |
| `withdraw_methods` | طرق السحب | WithdrawMethodController |
| `withdraw_requests` | طلبات السحب | WithdrawRequestController |
| `payment_settings` | إعدادات الدفع (key-value) | PaymentSettingsController |
| `settings` | إعدادات الموقع العامة | SettingsController |
| `smtp_configs` | إعدادات البريد الإلكتروني | SMTPConfigController |
| `sliders` | سلايدر الصفحة الرئيسية | SliderController |
| `advertisements` | الإعلانات | AdController |
| `reviews` | تقييمات المنتجات | ReviewController |
| `review_galleries` | صور التقييمات | ReviewController |
| `newsletters` | المشتركون في النشرة | AdminNewsLetterController |
| `maintainances` | إعداد وضع الصيانة | MaintainanceController |
| `about_pages` | محتوى صفحة "من نحن" | AboutPageController |
| `term_conditions` | الشروط والأحكام | TermConditionController |
| `vendor_conditions` | شروط انضمام البائعين | VendorConditionController |
| `top_category_sections` | قسم أبرز الفئات (الرئيسية) | TopCategorySectionController |
| `single_category_sections` | قسم الفئة المفردة (الرئيسية) | SingleCategorySectionController |
| `footer_sections` | إعدادات الفوتر | FooterSectionController |
| `wishlists` | قوائم أمنيات المستخدمين | - |
| `user_addresses` | عناوين المستخدمين | - |
| `password_reset_tokens` | رموز إعادة تعيين كلمة المرور | PasswordResetLinkController |
| `sessions` | جلسات المستخدمين | تلقائي من Laravel |
| `cache` | الكاش | - |
| `jobs` | مهام Queue | - |

---

## ملاحظات تقنية مهمة

### 1. رفع الصور:
```
جميع الصور تُرفع إلى: /public/uploads/
اسم الملف: rand() + '.' + extension
المسار المحفوظ في DB: '/uploads/filename.ext'
عند التعديل: حذف الصورة القديمة أولاً ثم رفع الجديدة
```

### 2. الـ Slug:
```php
$product->slug = \Str::slug($request->name);
// مثال: "Apple iPhone 15" → "apple-iphone-15"
```

### 3. رسائل النجاح والخطأ:
```php
// المكتبة المستخدمة: notyf (إشعارات مرئية)
notyf()->success('تم بنجاح!');
notyf()->error('حدث خطأ!');
```

### 4. الكاش:
```php
// SliderController بعد كل تعديل:
Cache::forget('sliders');
// لإعادة تحميل السلايدرات من DB
```

### 5. الـ DataTables والـ AJAX Response:
```php
// عند الحذف عبر DataTable:
return response(['status' => 'success']);
return response(['status' => 'error']);
// JavaScript في الواجهة يتعامل مع هذا الـ response
```

---

*تم إعداد هذا التوثيق بناءً على الكود الفعلي للمشروع*
*`Multi-Vendor-Ecommerce-Website-Using-Laravel-12`*
