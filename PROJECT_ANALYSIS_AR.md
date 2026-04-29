# 📦 تحليل شامل لمشروع متجر إلكتروني متعدد البائعين باستخدام Laravel 12

> **المستودع:** Multi-Vendor-Ecommerce-Website-Using-Laravel-12  
> **الإطار:** Laravel 12  
> **اللغة:** PHP 8.2+  
> **نوع المشروع:** Full-Stack Web Application

---

## 🔹 أولاً: نظرة عامة على المشروع

### ما هو هدف المشروع؟
هذا المشروع هو **موقع تجارة إلكترونية متعدد البائعين (Multi-Vendor Ecommerce)**، يشبه تمامًا مواقع مثل **Amazon** أو **Daraz** — حيث يمكن لأكثر من بائع (Vendor) عرض منتجاته على منصة واحدة، ويدير الموقع مدير (Admin) يتحكم في كل شيء.

### ما المشكلة التي يحلها؟
يحل مشكلة إنشاء سوق رقمي متكامل يجمع بين:
- **البائعين** الذين يريدون بيع منتجاتهم أونلاين دون بناء موقعهم الخاص
- **المشترين** الذين يريدون التسوق من مكان واحد
- **المدير** الذي يتحكم ويراقب كل العمليات ويربح عمولة

### ما نوعه؟
- **Full-Stack Web Application** (تطبيق ويب متكامل)
- له واجهة أمامية للزوار والمشترين
- له لوحة تحكم للمدير
- له لوحة تحكم للبائعين
- يستخدم نمط **MVC (Model-View-Controller)**

### ما التقنيات المستخدمة؟

| التقنية | الدور |
|---|---|
| **Laravel 12** | إطار العمل الرئيسي للباك-إند (PHP) |
| **Blade** | محرك القوالب لعرض صفحات HTML |
| **MySQL** | قاعدة البيانات |
| **Tailwind CSS** | تصميم الواجهة |
| **Vite** | أداة بناء ملفات CSS/JS |
| **Laravel Breeze** | نظام التسجيل والتحقق الجاهز |

---

## 🔹 ثانياً: هيكل المشروع (Project Structure)

تخيّل المشروع كمبنى متعدد الطوابق:

```
📁 المشروع
├── 📁 app/              ← قلب التطبيق (الكود الأساسي)
│   ├── 📁 Http/
│   │   ├── 📁 Controllers/    ← المتحكمون (يستقبلون الطلبات)
│   │   │   ├── 📁 Admin/      ← 40+ متحكم للمدير
│   │   │   ├── 📁 Vendor/     ← 9 متحكمين للبائع
│   │   │   ├── 📁 Frontend/   ← 10 متحكمين للواجهة الأمامية
│   │   │   ├── 📁 User/       ← متحكمون للمستخدم
│   │   │   └── 📁 Auth/       ← متحكمون تسجيل الدخول
│   │   ├── 📁 Middleware/      ← حراس البوابات
│   │   └── 📁 Requests/       ← التحقق من البيانات
│   ├── 📁 Models/        ← النماذج (الجداول في قاعدة البيانات)
│   ├── 📁 DataTables/    ← جداول البيانات الديناميكية
│   ├── 📁 Mail/          ← قوالب الإيميلات
│   ├── 📁 Helper/        ← دوال مساعدة عامة
│   ├── 📁 Library/       ← مكتبات إضافية
│   └── 📁 Providers/     ← إعدادات التطبيق عند الإقلاع
├── 📁 routes/            ← ملفات المسارات
│   ├── web.php           ← مسارات الويب الرئيسية
│   ├── admin.php         ← مسارات المدير
│   └── auth.php          ← مسارات التسجيل
├── 📁 resources/
│   └── 📁 views/         ← صفحات HTML (Blade)
│       ├── 📁 frontend/  ← صفحات الواجهة للزبائن
│       ├── 📁 Admin/     ← صفحات لوحة المدير
│       ├── 📁 vendor/    ← صفحات لوحة البائع
│       └── 📁 user/      ← صفحات حساب المستخدم
├── 📁 database/
│   └── 📁 migrations/    ← 40+ ملف لإنشاء الجداول
├── 📁 config/            ← إعدادات التطبيق (paypal, stripe, etc.)
└── 📁 public/            ← ملفات مرئية للزوار (صور, CSS, JS)
```

---

## 🔹 ثالثاً: تدفق العمل (Workflow)

### مثال من الحياة الواقعية:
> فكّر في المشروع كـ**مول تجاري كبير**:
> - **المدير** = مالك المول، يتحكم في كل شيء
> - **البائع** = صاحب محل داخل المول
> - **المستخدم** = الزبون الذي يتسوق

### كيف يبدأ تشغيل المشروع؟
1. المتصفح يفتح الموقع `http://localhost`
2. Laravel يقرأ ملف `routes/web.php`
3. يبحث عن المسار المناسب `Route::get('/', [HomeController::class, 'home'])`
4. يستدعي `HomeController@home`
5. الكونترولر يسحب البيانات من قاعدة البيانات
6. يرسل البيانات إلى **Blade View** لعرضها

### ما الذي يحدث عند دخول المستخدم؟
```
المستخدم يفتح الصفحة الرئيسية
        ↓
Laravel يستدعي HomeController::home()
        ↓
يجلب: السلايدر + العروض + التصنيفات + المنتجات + الإعلانات
        ↓
يرسل كل هذا إلى resources/views/frontend/home.blade.php
        ↓
المستخدم يرى الصفحة الرئيسية
```

### كيف تنتقل البيانات؟
البيانات تسير في اتجاه واحد:
```
Browser Request → routes/web.php → Controller → Model (قاعدة البيانات) → View (Blade) → Browser Response
```

---

## 🔹 رابعاً: شرح الكود التفصيلي

### أ) نموذج المستخدم `User.php`

```php
class User extends Authenticatable
{
    protected $fillable = [
        'name', 'email', 'password', 'role',    // بيانات أساسية
        'is_vendor', 'vendor_status',            // هل هو بائع؟ وما حالته؟
        'document', 'contact',                   // وثائق البائع
        'banner', 'address', 'desc',             // معلومات الملف الشخصي
        'fb_link', 'tw_link', 'insta_link', ...  // روابط التواصل الاجتماعي
    ];
}
```

**المستخدم الواحد يحمل دورين**: قد يكون `role = 'user'` (مشتري) أو `role = 'vendor'` (بائع). هذا يعني أن جدول `users` يخدم كلا النوعين.

### ب) نموذج المنتج `Product.php`

```php
class Product extends Model
{
    // منتج ينتمي لبائع أو مدير
    public function vendor()    { return $this->belongsTo(User::class); }
    public function admin()     { return $this->belongsTo(Admin::class); }

    // منتج له تصنيف وعلامة تجارية
    public function category()  { return $this->belongsTo(Category::class); }
    public function brand()     { return $this->belongsTo(Brand::class); }

    // منتج له خيارات متعددة (مثل: اللون، الحجم)
    public function variants()  { return $this->hasMany(ProductVariant::class); }

    // منتج له معرض صور
    public function productGallery() { return $this->hasMany(ProductImageGallery::class); }

    // منتج له تقييمات
    public function reviews()   { return $this->hasMany(Review::class); }
}
```

### ج) نظام السلة `CartController.php`

المشروع يستخدم مكتبة **`anayarojo/shoppingcart`** لإدارة السلة. السلة تُخزَّن في **Session** (الجلسة) وليس قاعدة البيانات.

```php
// إضافة منتج للسلة
public function addToCart(Request $request){
    $product = Product::findOrFail($request->product_id);

    // 1. التحقق من المخزون
    if($product->qty === 0) { return 'stockout'; }

    // 2. حساب السعر (هل هناك عرض؟)
    if($product->offer_price > 0 && /* التاريخ صحيح */ ){
        $productPrice = $product->offer_price; // سعر العرض
    } else {
        $productPrice = $product->price;       // السعر العادي
    }

    // 3. إضافة الخيارات (مثل: اللون الأحمر +5 دولار)
    foreach($request->variants_items as $item_id){
        $variantsTotal += $variantItem->price;
    }

    // 4. إضافة للسلة في الـ Session
    Cart::add($cartData);
}
```

### د) دوال الـ Helper المساعدة `helper.php`

هذه الدوال متاحة في كل مكان بالتطبيق (يتم تحميلها تلقائيًا عبر `composer.json`):

```php
// حساب المجموع الفرعي (بدون شحن وبدون خصم)
function getSubTotal() {
    foreach (Cart::content() as $item) {
        $total += ($item->price + $item->options->variantsTotal) * $item->qty;
    }
    return $total;
}

// حساب الخصم من الكوبون
function discount() {
    if ($coupon['discount_type'] === 'amount') {
        return $coupon['discount'];                         // خصم ثابت
    } else if ($coupon['discount_type'] === 'percent') {
        return ($coupon['discount'] / 100) * $subTotal;    // خصم نسبة مئوية
    }
}

// رسوم الشحن
function shippingFee() {
    return Session::get('shipping_rule')['cost'] ?? 0;
}

// التكلفة النهائية = المجموع - الخصم + الشحن
function finalCost() {
    return mainCartTotal() + shippingFee();
}
```

### هـ) نظام الدفع `PaymentController.php`

يدعم المشروع **3 بوابات دفع**:

**1. PayPal:**
```
المستخدم يضغط "ادفع بـ PayPal"
        ↓
payWithPaypal() تنشئ طلب دفع لدى PayPal
        ↓
تحويل المستخدم لصفحة PayPal
        ↓
بعد الدفع، PayPal يحول المستخدم إلى paypalSuccess()
        ↓
يتم التحقق من الدفع ثم استدعاء storeOrder() لحفظ الطلب
```

**2. Stripe:**
```
payWithStripe() تنشئ جلسة دفع Stripe
        ↓
تحويل المستخدم لصفحة Stripe الآمنة
        ↓
stripeSuccess() تتحقق وتحفظ الطلب
```

**3. SSLCommerz:**
```
يُستخدم في السوق البنغالي/الجنوب آسيوي
يتبع نفس المنطق لكن عبر مكتبة SSLCommerz
مسارات: POST /pay → /success → /fail → /cancel
```

### و) حفظ الطلب `storeOrder()`

هذه الدالة هي الأهم في عملية الشراء، تقوم بـ:

```php
public function storeOrder($transactionId, $amount, $currency, $payment_method, $paid_amount)
{
    // 1. إنشاء سجل الطلب
    $order = new Order();
    $order->invoice_id = time() . '-' . rand(1000, 9999);  // رقم فاتورة فريد
    $order->user_id = Auth::user()->id;
    $order->order_address = json_encode(Session::get('shippingAddress'));
    $order->shipping_method = json_encode(Session::get('shipping_rule'));
    $order->coupon = json_encode(Session::get('coupon'));
    $order->order_status = 'pending';
    $order->save();

    // 2. لكل منتج في السلة: إنشاء سجل order_product
    foreach (Cart::content() as $item) {
        $orderProduct = new OrderProduct();
        $orderProduct->vendor_id = $product->vendor_id;
        // تقليص المخزون
        $product->qty -= $item->qty;
        $product->save();
        $orderProduct->save();

        // إرسال إيميل للبائع/المدير
        \Mail::to($recipient->email)->send(new VendorOrders($recipient));
    }

    // 3. تسجيل المعاملة المالية
    $transaction = new Transaction();
    $transaction->save();

    // 4. إرسال إيميل للمستخدم
    \Mail::to($user->email)->send(new PaymentStatus($user));
}
```

### ز) Middleware التحقق من الدور `CheckRoleMiddleware.php`

```php
public function handle(Request $request, Closure $next, $role): Response
{
    // هل دور المستخدم يطابق الدور المطلوب؟
    if($request->user()->role === $role){
       return $next($request); // ✅ اسمح بالمرور
    }
    return redirect('/'); // ❌ ارجع للصفحة الرئيسية
}
```

يُستخدَم هكذا في المسارات:
```php
Route::group(['middleware' => ['auth', 'verified', 'check_role:vendor']], function(){
    // هذه المسارات للبائعين فقط
});
```

---

## 🔹 خامساً: التمييز بين الكود

### الكود المكتوب من قبل المطور (Custom Code):

| الملف | وصف ما كتبه المطور |
|---|---|
| `app/Http/Controllers/**` | كل منطق التطبيق |
| `app/Models/**` | تعريف الجداول والعلاقات |
| `app/Helper/helper.php` | دوال مساعدة لحساب الأسعار والشحن |
| `app/Helper/MailHelper.php` | تغيير إعدادات الإيميل ديناميكيًا |
| `app/Http/Middleware/CheckRoleMiddleware.php` | التحقق من دور المستخدم |
| `app/Providers/AppServiceProvider.php` | تحميل إعدادات الدفع عند الإقلاع |
| `resources/views/**` | جميع صفحات Blade |
| `database/migrations/**` | بنية قاعدة البيانات |
| `routes/web.php` و `routes/admin.php` | تعريف جميع المسارات |

### المكتبات الخارجية المستخدمة (Third-Party Libraries):

| المكتبة | الدور |
|---|---|
| `laravel/framework ^12.0` | إطار العمل الأساسي |
| `laravel/breeze` | نظام التسجيل/الدخول الجاهز |
| `anayarojo/shoppingcart ^4.2` | إدارة سلة التسوق في الـ Session |
| `srmklive/paypal ~3.0` | دمج بوابة دفع PayPal |
| `stripe/stripe-php ^17.3` | دمج بوابة دفع Stripe |
| `yajra/laravel-datatables 12.0` | جداول بيانات تفاعلية وقابلة للبحث |
| `spatie/laravel-permission ^6.20` | نظام الصلاحيات والأدوار |
| `php-flasher/flasher-notyf-laravel ^2.1` | إشعارات Notyf الجميلة |
| `barryvdh/laravel-debugbar` | أداة تطوير للمراقبة (dev only) |

---

## 🔹 سادساً: المسارات (Routing) — شرح مفصل

### هناك 3 ملفات للمسارات:

---

### 1. `routes/web.php` — مسارات الموقع الرئيسية

#### الواجهة الأمامية (لا تحتاج تسجيل دخول):

```
GET  /                      → HomeController::home()              ← الصفحة الرئيسية
GET  /about-page            → HomeController::aboutPage()         ← صفحة عن الموقع
GET  /contact-page          → HomeController::ContactPage()       ← صفحة التواصل
POST /contact-page/send-message → HomeController::sendMail()     ← إرسال رسالة
GET  /term-page             → HomeController::TermPage()          ← الشروط والأحكام
GET  /vendor-list           → HomeController::vendorList()        ← قائمة البائعين
GET  /vendor-details/{id}   → HomeController::vendorDetails()     ← صفحة بائع
GET  /product-details       → ProductDetailsController::index()   ← قائمة المنتجات
GET  /product-details/{id}  → ProductDetailsController::show()    ← تفاصيل منتج
GET  /flash-sale            → FlashSaleController::index()        ← عروض الفلاش
POST /add-to-cart           → CartController::addToCart()         ← إضافة للسلة
GET  /cart-count            → CartController::getCartCount()      ← عدد عناصر السلة
GET  /cart-details          → CartController::cartDetails()       ← عرض السلة
POST /cart/qty-update       → CartController::updateQty()         ← تغيير الكمية
GET  /cart/remove-item/{id} → CartController::removeItem()        ← حذف منتج
DELETE /cart/clear-cart/{id} → CartController::clearCart()        ← تفريغ السلة
POST /apply-coupon          → CartController::applyCoupon()       ← تطبيق كوبون
POST /coupon-calculation    → CartController::couponCalculation() ← حساب الخصم
GET  /order-track           → OrderTrackingController::index()    ← تتبع الطلب
POST /news-letter           → NewsLetterController::store()       ← الاشتراك في النشرة
```

#### مسارات المستخدم المسجل (`/user/...`):
> تحتاج: `auth + verified + check_role:user`

```
GET  /user/dashboard            → UserDashboardController::index()
GET  /user/checkout             → CheckoutController::index()
POST /user/checkout/form-submit → CheckoutController::checkoutFormSubmit()
GET  /user/payment              → PaymentController::index()
GET  /user/payment/paypal       → PaymentController::payWithPaypal()
GET  /user/payment/paypal/success → PaymentController::paypalSuccess()
GET  /user/payment/stripe       → PaymentController::payWithStripe()
GET  /user/payment/stripe/success → PaymentController::stripeSuccess()
GET  /user/payment/success      → PaymentController::paymentSuccess()
GET  /user/payment/failed       → PaymentController::paymentFailed()
CRUD /user/order                → UserOrderController
CRUD /user/address              → UserAddressController
CRUD /user/wishlist             → WishlistController
CRUD /user/review               → ReviewController
CRUD /user/profile              → UserProfileController
CRUD /user/vendor-request       → UserVendorRequestController
```

#### مسارات البائع (`/vendor/...`):
> تحتاج: `auth + verified + check_role:vendor`

```
GET  /vendor/dashboard          → VendorDashboardController::index()
CRUD /vendor/product            → ProductController
CRUD /vendor/product/image-gallery → ProductImageGalleryController
GET  /vendor/product/variant/{product_id} → ProductVariantController::index()
POST /vendor/product/variant/store → ProductVariantController::store()
GET  /vendor/product/variant-item/{product_id}/{variant_id} → ProductVariantItemController::index()
CRUD /vendor/order              → OrderController
POST /vendor/change-order-status → OrderController::changeOrderStatus()
CRUD /vendor/review             → VendorReviewController
CRUD /vendor/withdraw-request   → WithdrawRequestController
CRUD /vendor/profile            → VendorProfileController
```

---

### 2. `routes/admin.php` — لوحة التحكم (`/admin/...`)

#### بدون تسجيل دخول:
```
GET  /admin/login               → AuthenticatedSessionController::create()
POST /admin/login               → AuthenticatedSessionController::store()
GET  /admin/forgot-password     → PasswordResetLinkController::create()
POST /admin/reset-password      → NewPasswordController::store()
```

#### بعد تسجيل الدخول (تحتاج: `auth:admin`):
```
GET  /admin/dashboard           → AdminDashboardController::index()
CRUD /admin/category            → CategoryController
CRUD /admin/sub-category        → SubCategoryController
CRUD /admin/child-category      → ChildCategoryController
CRUD /admin/brand               → BrandController
CRUD /admin/slider              → SliderController
CRUD /admin/product             → ProductController (منتجات المدير)
CRUD /admin/vendor-product      → VendorProductController (منتجات البائعين)
CRUD /admin/vendor-request      → VendorRequestController
CRUD /admin/approved-vendors    → ApprovedVendorController
CRUD /admin/manage-user         → ManageUserController
CRUD /admin/manage-admin        → ManageAdminController
CRUD /admin/all-orders          → AllOrdersController
POST /admin/change-order-status → AllOrdersController::changeOrderStatus()
CRUD /admin/coupon              → CouponController
CRUD /admin/flash-sale          → FlashSaleController
CRUD /admin/shipping-rule       → ShippingRuleController
CRUD /admin/payment-settings    → PaymentSettingsController
CRUD /admin/settings            → SettingsController
CRUD /admin/transaction         → TransactionController
CRUD /admin/withdraw-method     → WithdrawMethodController
CRUD /admin/withdraw-request    → WithdrawRequestController
CRUD /admin/review              → ReviewController
CRUD /admin/ad                  → AdController
CRUD /admin/news-letter         → AdminNewsLetterController
CRUD /admin/smtp-config         → SMTPConfigController
CRUD /admin/footer-section      → FooterSectionController
CRUD /admin/top-category        → TopCategorySectionController
CRUD /admin/single-category     → SingleCategorySectionController
CRUD /admin/slider              → SliderController
CRUD /admin/about-page          → AboutPageController
CRUD /admin/term-page           → TermConditionController
CRUD /admin/vendor-condition    → VendorConditionController
CRUD /admin/maintainance        → MaintainanceController
CRUD /admin/role                → AdminRoleController
CRUD /admin/permission          → PermissionController
CRUD /admin/role-in-permission  → RoleInPermissionController
CRUD /admin/profile             → AdminProfileController
```

---

## 🔹 سابعاً: قاعدة البيانات

### الجداول الرئيسية وعلاقاتها:

#### جدول `users` (المستخدمون والبائعون)
```
id, name, email, password
role: 'user' | 'vendor'
is_vendor: 0 | 1
vendor_status: 'pending' | 'approved' | 'rejected'
vendor_request: 0 | 1
document, contact, banner, address, desc
fb_link, tw_link, insta_link, tiktok_link, yt_link
```

#### جدول `admins` (المديرون)
```
id, name, email, password
status: 'active' | 'banned'
image, contact, address
created_by (من أنشأ هذا المدير)
```

#### جدول `products` (المنتجات)
```
id, name, slug, thumb_image
vendor_id → users (0 = منتج المدير)
admin_id → admins (0 = منتج البائع)
category_id → categories
sub_category_id → sub_categories (nullable)
child_category_id → child_categories (nullable)
brand_id → brands
qty (الكمية المتاحة)
short_description, long_description, video_link, sku
price, offer_price, offer_start_date, offer_end_date
product_type: 'new_arrival' | 'featured_product' | 'top_product' | 'best_product'
status: 0 | 1
is_approved: 0 | 1 (يحتاج موافقة المدير)
seo_title, seo_description
```

#### جدول `product_variants` (خيارات المنتج مثل: اللون)
```
id, product_id → products, name, status
```

#### جدول `product_variant_items` (قيم الخيارات مثل: أحمر، أزرق)
```
id, product_variant_id → product_variants
name, price (سعر إضافي), status
```

#### جدول `orders` (الطلبات)
```
id, invoice_id, transaction_id
user_id → users
sub_total, amount, currency_name, currency_icon
product_qty
payment_method: 'paypal' | 'stripe' | 'sslcommerz'
payment_status: 0 | 1
order_address (JSON: عنوان التوصيل)
shipping_method (JSON: طريقة الشحن)
coupon (JSON: الكوبون المستخدم)
order_status: 'pending' | 'processing' | 'delivered' | 'canceled'
```

#### جدول `order_products` (تفاصيل المنتجات في كل طلب)
```
id, order_id → orders, transaction_id
product_id → products, vendor_id → users
product_name, variants (JSON), variant_total
unit_price, qty
```

#### جدول `transactions` (سجل المعاملات المالية)
```
id, order_id → orders, transaction_id
payment_method, amount
amount_real_currency, amount_real_currency_name
```

#### جدول `coupons` (أكواد الخصم)
```
id, name, code
discount_type: 'amount' | 'percent'
discount
start_date, end_date
quantity (عدد مرات الاستخدام المتاحة)
total_used (عدد مرات الاستخدام الفعلية)
status: 0 | 1
```

#### جدول `wishlists` (المفضلة)
```
id, user_id → users, product_id → products
```

#### جدول `reviews` (التقييمات)
```
id, user_id → users, product_id → products
vendor_id → users
rating, review, status: 0 | 1
```

#### جدول `withdraw_requests` (طلبات سحب الأرباح)
```
id, vendor_id → users
method (PayPal, بنك، إلخ)
total_earnings, withdraw_amount, withdraw_charge
account_info, current_balance
status: 'pending' | 'paid' | 'rejected'
```

#### جداول التصنيفات (هرمية):
```
categories → sub_categories → child_categories
(كل فئة رئيسية لها فئات فرعية، وكل فئة فرعية لها فئات أعمق)
```

#### جدول `settings` (الإعدادات العامة)
```
site_name, site_email, contact_email
currency_name, currency_icon, time_zone, ...
```

#### جدول `payment_settings` (إعدادات بوابات الدفع)
```
key → value
مثال: 'paypal_client_id' → 'XXXXXXXX'
      'stripe_client_secret' → 'sk_test_...'
      'ssl_store_id' → '...'
```

### العلاقات بشكل مرئي:

```
Category (1) ──→ (many) SubCategory (1) ──→ (many) ChildCategory
Brand (1) ──→ (many) Product
Product (1) ──→ (many) ProductVariant (1) ──→ (many) ProductVariantItem
Product (1) ──→ (many) ProductImageGallery
Product (1) ──→ (many) Review
User/vendor (1) ──→ (many) Product
User/buyer  (1) ──→ (many) Order
Order (1) ──→ (many) OrderProduct
Order (1) ──→ (1) Transaction
User (1) ──→ (many) Wishlist
User (1) ──→ (many) UserAddress
User (1) ──→ (many) WithdrawRequest [as vendor]
```

---

## 🔹 ثامناً: الواجهة الأمامية (Frontend)

### كيف تعمل؟
المشروع يستخدم **Blade Templates** (محرك قوالب Laravel) وليس React أو Vue. يعني أن الـ HTML يُولَّد من جهة الـ Server وليس من جهة المتصفح.

### هيكل الـ Views:
```
resources/views/
├── frontend/
│   ├── home.blade.php              ← الصفحة الرئيسية
│   ├── layout/
│   │   └── master.blade.php        ← القالب الرئيسي (Header + Footer)
│   ├── sections/                   ← أقسام الصفحة الرئيسية
│   │   ├── banner-section.blade.php
│   │   ├── flash-sell-section.blade.php
│   │   ├── top-category-section.blade.php
│   │   ├── brand-slider-section.blade.php
│   │   ├── top-product-type-section.blade.php
│   │   └── single-category-section-*.blade.php
│   └── pages/
│       ├── product-list.blade.php
│       ├── product-details.blade.php
│       ├── cart-details.blade.php
│       ├── checkout.blade.php
│       ├── vendor-list.blade.php
│       ├── vendor-details.blade.php
│       ├── about-page.blade.php
│       ├── contact-page.blade.php
│       └── payment/
│           ├── index.blade.php
│           ├── success.blade.php
│           └── failed.blade.php
├── Admin/                          ← 30+ مجلد لإدارة كل قسم
├── vendor/                         ← لوحة تحكم البائع
└── user/                           ← صفحات المستخدم
```

### كيف يتم جلب البيانات ديناميكيًا؟
بعض العمليات تتم بـ **AJAX** (JavaScript بدون إعادة تحميل الصفحة):

| العملية | النوع | المسار |
|---|---|---|
| إضافة للسلة | POST AJAX | `/add-to-cart` |
| تغيير الكمية | POST AJAX | `/cart/qty-update` |
| تطبيق كوبون | POST AJAX | `/apply-coupon` |
| حساب الخصم | POST AJAX | `/coupon-calculation` |
| جلب الفئات الفرعية | GET AJAX | `/vendor/product/get-sub-categories` |
| عدد عناصر السلة | GET AJAX | `/cart-count` |

### `AppServiceProvider` — السحر الخفي:

```php
// يُرسَل هذا لكل صفحة في التطبيق تلقائيًا!
View::composer('*', function($view) use ($settings, $paymentSettings){
    $view->with(['settings' => $settings, 'paymentSettings' => $paymentSettings]);
});
```

لذلك يمكنك في أي Blade استخدام `{{ $settings->site_name }}` مباشرة.

---

## 🔹 تاسعاً: نقاط مهمة للتعلم

### ✅ أفضل الممارسات الموجودة في المشروع:

1. **فصل المسارات**: ملفات `web.php` و `admin.php` منفصلة ومرتبة
2. **Middleware للصلاحيات**: `check_role:vendor` يمنع الوصول غير المصرح
3. **مجموعات المسارات (Route Groups)**: تطبيق Prefix ومدل على مجموعة بدل التكرار
4. **Eloquent Relationships**: علاقات واضحة بين النماذج (hasMany, belongsTo)
5. **DataTables Library**: لجداول البيانات الكبيرة مع البحث والترتيب تلقائيًا
6. **Cache**: `Cache::rememberForever('sliders', ...)` لتقليل استعلامات قاعدة البيانات
7. **Dynamic Config**: إعدادات الدفع تُحمَّل من قاعدة البيانات (مرونة عالية)
8. **Dynamic Mail Config**: إعدادات الإيميل تُغيَّر برمجيًا دون تغيير `.env`
9. **Multiple Payment Gateways**: 3 بوابات دفع بنفس الواجهة
10. **SEO Fields**: المنتجات تحتوي على `seo_title` و `seo_description`
11. **Spatie Permissions**: نظام أدوار وصلاحيات متكامل للمديرين

### ⚠️ تحسينات ممكنة ونقاط ضعف:

1. **مسارات تشخيص خطرة في Production**:
   ملف `web.php` يحتوي على مسارات `/debug-paypal-config` و `/debug-sslcommerz-config` — هذه **خطرة جداً في الإنتاج** لأنها تكشف بيانات حساسة مثل API Keys!

2. **عدم استخدام Foreign Key Constraints**:
   الهجرات تستخدم `$table->integer('vendor_id')` بدلاً من `$table->foreignId('vendor_id')->constrained()` — لا توجد حماية على مستوى قاعدة البيانات من الروابط المكسورة.

3. **السلة تُخزَّن في Session فقط**:
   إذا انتهت الجلسة تضيع السلة — يُفضَّل حفظها في قاعدة البيانات للزبائن المسجلين.

4. **عدم استخدام Form Request Classes**:
   بعض الكونترولرات تحقق البيانات مباشرة فيها بدلاً من ملفات `Requests/` المنفصلة.

5. **تحميل الإعدادات في كل طلب**:
   `AppServiceProvider` يستعلم `PaymentSettings::all()` في كل طلب — يمكن استخدام **Cache** لتقليل الضغط على قاعدة البيانات.

---

## 🔹 عاشراً: تبسيط الفهم — رحلة المستخدم الكاملة

### مثال كامل: مستخدم يشتري منتجاً

```
1️⃣  المستخدم يفتح الموقع
     ↓ HomeController::home() يجلب بيانات الصفحة الرئيسية
     ↓ السلايدر + العروض + التصنيفات + المنتجات + الإعلانات

2️⃣  المستخدم يضغط على منتج
     ↓ ProductDetailsController::show($id)
     ↓ يجلب: تفاصيل المنتج + التقييمات + هل يمكن للمستخدم التقييم؟

3️⃣  المستخدم يضغط "أضف للسلة"
     ↓ JavaScript يرسل AJAX POST إلى /add-to-cart
     ↓ CartController::addToCart() يتحقق من المخزون ويضيف للـ Session

4️⃣  المستخدم يذهب لصفحة السلة
     ↓ CartController::cartDetails() يعرض محتويات السلة

5️⃣  المستخدم يطبق كوبون خصم
     ↓ CartController::applyCoupon() يتحقق من:
        - هل الكوبون موجود؟
        - هل التاريخ صالح؟
        - هل بقي منه استخدامات؟
     ↓ يحفظ الكوبون في Session

6️⃣  المستخدم يذهب للـ Checkout
     ↓ CheckoutController::index() يعرض العناوين وطرق الشحن
     ↓ CheckoutController::checkoutFormSubmit() يحفظ العنوان والشحن في Session

7️⃣  المستخدم يختار طريقة الدفع
     ↓ PaymentController::payWithPaypal() أو payWithStripe()
     ↓ يحوّل المستخدم لبوابة الدفع الخارجية

8️⃣  بعد الدفع الناجح
     ↓ PaymentController::storeOrder() يحفظ:
        - سجل Order في جدول orders
        - سجل OrderProduct لكل منتج في order_products
        - سجل Transaction في جدول transactions
        - يُنقَّص المخزون من كل منتج
        - يُرسَل إيميل للمستخدم (PaymentStatus)
        - يُرسَل إيميل لكل بائع (VendorOrders)
     ↓ تُفرَّغ السلة والـ Session (cart, coupon, shippingAddress)
     ↓ يُوجَّه المستخدم لصفحة النجاح

9️⃣  البائع يرى الطلب في لوحة تحكمه
     ↓ VendorDashboardController / OrderController
     ↓ يغير حالة الطلب: pending → processing → delivered

🔟  المستخدم يكتب تقييماً بعد التوصيل
     ↓ يتحقق أن المستخدم اشترى فعلاً وتم التسليم (canReview)
     ↓ ReviewController::store() يحفظ التقييم والصور
```

---

### مثال: بائع جديد يريد الانضمام

```
1️⃣  المستخدم العادي يطلب أن يصبح بائعاً
     ↓ UserVendorRequestController يحفظ الطلب (vendor_request = 1)

2️⃣  المدير يرى الطلب في لوحة التحكم
     ↓ VendorRequestController::index() يعرض الطلبات

3️⃣  المدير يوافق على الطلب
     ↓ user.role = 'vendor'
     ↓ user.vendor_status = 'approved'
     ↓ يُرسَل إيميل للمستخدم (VendorStatus Mail)

4️⃣  البائع الجديد يسجل دخوله كبائع
     ↓ يضيف منتجاته من /vendor/product/create
     ↓ المنتجات تنتظر موافقة المدير (is_approved = 0)

5️⃣  المدير يوافق على المنتجات
     ↓ is_approved = 1
     ↓ المنتج يظهر للزبائن في الموقع
     ↓ يُرسَل إيميل للبائع (ProductStatus Mail)

6️⃣  بعد تحقيق مبيعات، البائع يطلب سحب أرباحه
     ↓ WithdrawRequestController::store() يحسب:
        - إجمالي الأرباح
        - الرسوم (withdraw_charge)
        - المبلغ الصافي
     ↓ المدير يوافق ويدفع
     ↓ status = 'paid'
```

---

## 📊 ملخص المشروع الكامل

| الجانب | التفاصيل |
|---|---|
| **نوع المشروع** | Multi-Vendor E-commerce (Full-Stack) |
| **Framework** | Laravel 12 |
| **قاعدة البيانات** | MySQL |
| **Frontend** | Blade Templates + Tailwind CSS |
| **عدد الجداول** | 40+ جدول |
| **بوابات الدفع** | PayPal + Stripe + SSLCommerz |
| **الأدوار** | Admin + Vendor + User |
| **نظام الإيميل** | SMTP ديناميكي من قاعدة البيانات |
| **الصلاحيات** | Spatie Permissions |
| **الإشعارات** | Notyf Flash Messages |
| **الجداول التفاعلية** | Yajra DataTables |
| **عدد Controllers** | 70+ Controller |

---

*تم إنشاء هذا المستند تلقائياً بتاريخ 2026-04-29*
