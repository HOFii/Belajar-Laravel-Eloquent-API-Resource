# LARAVEL ELOQUENT-API-RESOURCE

## POINT UTAMA

### 1. Instalasi

-   Minimal PHP versi 8 atau lebih,

-   Composer versi 2 atau lebih,

-   Lalu pada cmd ketikan `composer create-project laravel/laravel=v10.2.5 belajar-laravel-eloquent-api-resource`.

---

### 2. Pengenalan

-   Laravel Eloquent API Resource adalah cara untuk mentransformasi data dari model Eloquent ke format yang sesuai untuk ditampilkan melalui API. Ini memungkinkan Anda untuk dengan mudah mengatur data yang dikirimkan oleh model Anda dalam bentuk yang diinginkan, seperti JSON, tanpa harus mengubah struktur model itu sendiri.

-   Dengan menggunakan Laravel Eloquent API Resource, Anda dapat menentukan secara tepat bagaimana data dari model Anda harus diformat ketika dikirim sebagai respons API. Ini memberi Anda kontrol penuh atas representasi JSON dari model Anda, termasuk pemformatan data, menyembunyikan kolom tertentu, atau menambahkan data tambahan.

---

### 3. Model Category

-   Pada terminal gunakan perintah `php artisan make:model Category --migration --seed`. perintah yang sama saat di Laravel Eloquent.

-   Kode Category Migration

    ```PHP
     public function up(): void
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string("name", 100)->nullable(false);
            $table->text("description")->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('categories');
    }
    ```

-   Kode Model Category

    ```PHP
    class Category extends Model
    {
        protected $table = "categories";
        protected $primaryKey = "id";
        protected $keyType = "int";
        public $incrementing = true;
        public $timestamps = true;

        public function products(): HasMany
    {
        return $this->hasMany(Product::class, "category_id", "id"); // category relation
    }
    }
    ```

---

### 4. Model Product

-   Kode Product Migration

    ```PHP
    public function up(): void
    {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string("name", 100)->nullable(false);
            $table->bigInteger("price")->nullable(false)->default(0);
            $table->integer("stock")->nullable(false)->default(0);
            $table->unsignedBigInteger("category_id")->nullable(false);
            $table->timestamps();
            $table->foreign("category_id")->on("categories")->references("id");
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('products');
    }
    ```

-   kode Model Product

    ```PHP
    class Product extends Model
    {
        protected $table = "products";
        protected $primaryKey = "id";
        protected $keyType = "int";
        public $incrementing = true;
        public $timestamps = true;

        public function category(): BelongsTo
        {
            return $this->belongsTo(Category::class, "category_id", "id"); // product resali ke category
        }
    }
    ```

---

### 5. Resource

-   Resource merupakan representasi dari cara melakukan transformasi dari Model menjadi _Array_ /
    JSON yang kita inginkan,

-   Untuk membuat Resource, kita bisa menggunakan perintah `php artisan make:resource NamaResource`,

-   Class Resource adalah class turunan dari class _JsonResource_, dan kita perlu mengubah implementasi dari method `toArray` nya.

-   kode Category Resource

    ```PHP
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at
        ];
    }
    ```

-   Kode Route Category Resource

    ```PHP
    Route::get('/categories/{id}', function ($id) {
    $category = \App\Models\Category::findOrFail($id);
    return new \App\Http\Resources\CategoryResource($category);
    });
    ```

-   kode Category Seed

    ```PHP
     public function run(): void
    {
        Category::create([
            "name" => "Food"
        ]);
        Category::create([
            "name" => "Gadget"
        ]);
    }
    ```

-   kode Resource Test

    ```PHP
    public function testResource()
    {
        $this->seed([CategorySeeder::class]);

        $category = Category::first();

        $this->get("/api/categories/$category->id")
            ->assertStatus(200)
            ->assertJson([
                'data' => [ // wrap attribute
                    'id' => $category->id,
                    'name' => $category->name,
                    'created_at' => $category->created_at->toJSON(),
                    'updated_at' => $category->updated_at->toJSON(),
                ]
            ]);
    }
    ```

---

### 6. Wrap Attribute

-   Secara default, data JSON yang kita kembalikan dalam method `toArray()` akan di _wrap_ dalam
    attribute bernama `"data"`,

-   Jika kita ingin ubah nama attribute di JSON nya, kita bisa ubah menggunakan attribute `$wrap` di
    Resource nya.

---

### 7. Resource Collection

-   Secara default, Resource yang sudah kita buat, bisa kita gunakan untuk menampilkan data multiple object atau dalam bentuk JSON _Array_,

-   Bisa menggunakan static method `collection()` ketika membuat Resource nya, dan gunakan parameter berisi data collection.

-   Kode Resource Collection

    ```PHP
    Route::get('/categories', function () {
    $categories = \App\Models\Category::all();
    return \App\Http\Resources\CategoryResource::collection($categories);
    });
    ```

-   Kode Test Resource Collection

    ```PHP
    public function testResourceCollection()
    {
        $this->seed([CategorySeeder::class]);

        $categories = Category::all();

        $this->get("/api/categories")
            ->assertStatus(200)
            ->assertJson([
                'data' => [
                    [ // data pertama
                        'id' => $categories[0]->id,
                        'name' => $categories[0]->name,
                        'created_at' => $categories[0]->created_at->toJSON(),
                        'updated_at' => $categories[0]->updated_at->toJSON(),
                    ],
                    [ // data kedua
                        'id' => $categories[1]->id,
                        'name' => $categories[1]->name,
                        'created_at' => $categories[1]->created_at->toJSON(),
                        'updated_at' => $categories[1]->updated_at->toJSON(),
                    ]
                ]
            ]);
    }
    ```

---

### 8. Custom Resource Collection

-   Saat ingin membuat class Resource Collection secara manual, tanpa menggunakan Resource Class yang sudah di buat, bisa membuat Resource, namun menggunakan tambahan parameter `--collection`,

-   Gunakan perintah `php artisan make:resource NamaCollection --collection`,

-   Secara otomatis class Resource adalah turunan dari class Resource Collection,

-   Untuk mengambil informasi collection nya, kita bisa menggunakan _attribute_ `$collection`.

-   Kode Resource Collection

    ```PHP
     public function toArray(Request $request): array
    {
        return [
            "data" => CategorySimpleResource::collection($this->collection),
            "total" => count($this->collection)
        ];
    }
    ```

-   Kode Route

    ```PHP
    Route::get('/categories-custom', function () {
    $categories = \App\Models\Category::all();
    return new \App\Http\Resources\CategoryCollection($categories);
    });
    ```

-   Kode Test Custom Resource Collection

    ```PHP
    public function testResourceCollectionCustom()
    {
        $this->seed([CategorySeeder::class]);

        $categories = Category::all();

        $this->get("/api/categories-custom")
            ->assertStatus(200)
            ->assertJson([
                'total' => 2,
                'data' => [
                    [
                        'id' => $categories[0]->id,
                        'name' => $categories[0]->name
                    ],
                    [
                        'id' => $categories[1]->id,
                        'name' => $categories[1]->name
                    ]
                ]
            ]);
    }
    ```

---

### 9. Nested Resource

-   Saat kita menggunakan Resource, contoh pada Resource Collection, kita juga bisa menggunakan
    Resource lainnya,

-   Secara default, method `toArray()` akan dikonversi menjadi JSON, namun bisa juga menggunakan Resource lain jika mau.

-   Kode Category Simple Resource

    ```PHP
    public function toArray(Request $request): array
    {
        return [
            "id" => $this->id,
            "name" => $this->name,
        ];
    }
    ```

-   Kode Category Collection

    ```PHP
     public function toArray(Request $request): array
    {
        return [
            "data" => CategorySimpleResource::collection($this->collection),
            "total" => count($this->collection)
        ];
    }
    ```

---

### 10. Data Wrap

-   Secara default, data JSON yang dibuat oleh Resource akan disimpan dalam _attribute_ `"data"`,

-   Jika kita ingin menggantinya, kita bisa ubah _attribute_ `$wrap` di Resource dengan nama _attribute_ yang kita mau,

-   Secara default, jika dalam `toArray()` kita mengembalikan array yang terdapat _attribute_ sama dengan `$wrap`, maka data JSON tidak akan di wrap.

-   Kode Product Resource

    ```PHP
    public static $wrap = "value"; // udab dari "data" ke "value"

    public function toArray(Request $request): array
    {
        return [
            "id" => $this->id,
            "name" => $this->name,
            "category" => new CategorySimpleResource($this->category),
            "price" => $this->price,
            "created_at" => $this->created_at,
            "updated_at" => $this->updated_at,
        ];
    }
    ```

-   Kode Product Seed

    ```PHP
     public function run(): void
    {
        Category::all()->each(function (Category $category) {
            for ($i = 0; $i < 5; $i++) {
                $category->products()->create([
                    'name' => "Product $i of $category->name",
                    'price' => rand(100, 1000)
                ]);
            }
        });
    }
    ```

-   kode Route

    ```PHP
    Route::get('/products/{id}', function ($id) {
    $product = \App\Models\Product::find($id);
    return (new \App\Http\Resources\ProductResource($product));
    });
    ```

-   Kode Product Test

    ```PHP
    public function testProduct()
    {
        $this->seed([CategorySeeder::class, ProductSeeder::class]);

        $product = Product::first();

        $this->get("/api/products/$product->id")
            ->assertStatus(200)
            ->assertJson([
                "value" => [
                    "name" => $product->name,
                    "category" => [
                        "id" => $product->category->id,
                        "name" => $product->category->name,
                    ],
                    "price" => $product->price,
                    "created_at" => $product->created_at->toJSON(),
                    "updated_at" => $product->updated_at->toJSON(),
                ]
            ]);
    }
    ```

---

## PERTANYAAN & CATATAN TAMBAHAN

-   Tidak ada.

### KESIMPULAN

-   Pengenalan API Resource menjelaskan bahwa API Resource di Laravel digunakan untuk mengubah model Eloquent menjadi bentuk JSON yang sesuai untuk API. Ini membantu dalam mengontrol dan memformat data yang dikirimkan ke klien.

-   Menunjukkan cara membuat resource menggunakan perintah Artisan php artisan make:resource. Resource yang dibuat akan diletakkan di direktori app/Http/Resources.

-   Menjelaskan bagaimana kita dapat mengkustomisasi output JSON dengan menambahkan atau mengubah atribut yang disertakan dalam response.
