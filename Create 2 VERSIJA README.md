5 ŽINGSNIS – Sukuriam Seeder kategorijų duomenims
Kad į categories lentelę būtų įrašyti pradiniai duomenys automatiškai (pvz. „Drabužiai“, „Technika“, „Buitis“), tam naudoti seeder.

Terminale įvesk:
php artisan make:seeder CategorySeeder



Eik į failą:
database/seeders/CategorySeeder.php

Ir pakeisk jį į šį pilną kodą:

<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class CategorySeeder extends Seeder
{
    public function run(): void
    {
        DB::table('categories')->insert([
            ['name' => 'Drabužiai'],
            ['name' => 'Technika'],
            ['name' => 'Buitis'],
            ['name' => 'Sportas'],
            ['name' => 'Maistas']
        ]);
    }
}

Dabar atidaryk DatabaseSeeder.php failą:
Ir PRIDĖK šitą eilutę į run() metodą:

$this->call([
    CategorySeeder::class,
]);

 6 ŽINGSNIS – Sukuriam Product modelį, kontrolerį ir migraciją

php artisan make:model Product -mcr

Dabar redaguojam Product.php modelį:

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = [
        'name',
        'description',
        'price',
        'image',
        'category_id',
        'brand_id',
    ];

    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function brand()
    {
        return $this->belongsTo(Brand::class);
    }
}

Sukurk dar modelius Category ir Brand:

php artisan make:model Category
php artisan make:model Brand


Tada:
 app/Models/Category.php:

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function products()
    {
        return $this->hasMany(Product::class);
    
}


app/Models/Brand.php:

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Brand extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function products()
    {
        return $this->hasMany(Product::class);
    }
}

Pridėkim maršrutus į routes/web.php

use App\Http\Controllers\ProductController;

Route::resource('products', ProductController::class);


