7 ŽINGSNIS – Pakeičiam ProductController.php pilnai

<?php

namespace App\Http\Controllers;

use App\Models\Product;
use App\Models\Category;
use App\Models\Brand;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index()
    {
        $products = Product::with('category', 'brand')->paginate(5);
        return view('products.index', compact('products'));
    }

    public function create()
    {
        $categories = Category::all();
        $brands = Brand::all();
        return view('products.create', compact('categories', 'brands'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'name' => 'required|min:3',
            'description' => 'required',
            'price' => 'required|numeric',
            'category_id' => 'required|exists:categories,id',
            'brand_id' => 'required|exists:brands,id',
            'image' => 'nullable|image|mimes:jpg,jpeg,png,gif|max:2048',
        ]);

        $imagePath = null;
        if ($request->hasFile('image')) {
            $imagePath = $request->file('image')->store('images', 'public');
        }

        Product::create([
            'name' => $request->name,
            'description' => $request->description,
            'price' => $request->price,
            'category_id' => $request->category_id,
            'brand_id' => $request->brand_id,
            'image' => $imagePath,
        ]);

        return redirect()->route('products.index')->with('success', 'Prekė sėkmingai pridėta!');
    }

    public function show(Product $product)
    {
        return view('products.show', compact('product'));
    }

    public function edit(Product $product)
    {
        $categories = Category::all();
        $brands = Brand::all();
        return view('products.edit', compact('product', 'categories', 'brands'));
    }

    public function update(Request $request, Product $product)
    {
        $request->validate([
            'name' => 'required|min:3',
            'description' => 'required',
            'price' => 'required|numeric',
            'category_id' => 'required|exists:categories,id',
            'brand_id' => 'required|exists:brands,id',
            'image' => 'nullable|image|mimes:jpg,jpeg,png,gif|max:2048',
        ]);

        $imagePath = $product->image;
        if ($request->hasFile('image')) {
            $imagePath = $request->file('image')->store('images', 'public');
        }

        $product->update([
            'name' => $request->name,
            'description' => $request->description,
            'price' => $request->price,
            'category_id' => $request->category_id,
            'brand_id' => $request->brand_id,
            'image' => $imagePath,
        ]);

        return redirect()->route('products.index')->with('success', 'Prekė atnaujinta sėkmingai!');
    }

    public function destroy(Product $product)
    {
        $product->delete();
        return redirect()->route('products.index')->with('success', 'Prekė ištrinta!');
    }
}


8 ŽINGSNIS – Sukuriam resources/views/products/ aplanką


Eik į resources/views/

Sukurk naują aplanką pavadinimu:

products

ame aplanke sukursim 4 failus:

index.blade.php

create.blade.php

edit.blade.php

show.blade.php

 1. index.blade.php – visų prekių sąrašas

 <!DOCTYPE html>
<html>
<head>
    <title>Prekių sąrašas</title>
</head>
<body>
    <h1>Prekės</h1>

    @if(session('success'))
        <p style="color: green;">{{ session('success') }}</p>
    @endif

    <a href="{{ route('products.create') }}">➕ Pridėti prekę</a>

    <table border="1" cellpadding="10">
        <tr>
            <th>Pavadinimas</th>
            <th>Kategorija</th>
            <th>Kaina</th>
            <th>Veiksmai</th>
        </tr>
        @foreach($products as $product)
        <tr>
            <td>{{ $product->name }}</td>
            <td>{{ $product->category->name ?? '-' }}</td>
            <td>{{ $product->price }} €</td>
            <td>
                <a href="{{ route('products.show', $product->id) }}">Peržiūrėti</a> |
                <a href="{{ route('products.edit', $product->id) }}">Redaguoti</a> |
                <form action="{{ route('products.destroy', $product->id) }}" method="POST" style="display:inline;">
                    @csrf
                    @method('DELETE')
                    <button type="submit">Trinti</button>
                </form>
            </td>
        </tr>
        @endforeach
    </table>

    <div>
        {{ $products->links() }}
    </div>
</body>
</html>


 2. create.blade.php – naujos prekės forma

 <!DOCTYPE html>
<html>
<head>
    <title>Pridėti prekę</title>
</head>
<body>
    <h1>Pridėti prekę</h1>

    @if($errors->any())
        <ul style="color:red;">
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    @endif

    <form action="{{ route('products.store') }}" method="POST" enctype="multipart/form-data">
        @csrf
        <p>Pavadinimas: <input type="text" name="name" value="{{ old('name') }}"></p>
        <p>Aprašymas: <textarea name="description">{{ old('description') }}</textarea></p>
        <p>Kaina: <input type="text" name="price" value="{{ old('price') }}"></p>
        <p>Kategorija: 
            <select name="category_id">
                @foreach($categories as $category)
                    <option value="{{ $category->id }}">{{ $category->name }}</option>
                @endforeach
            </select>
        </p>
        <p>Prekės ženklas:
            <select name="brand_id">
                @foreach($brands as $brand)
                    <option value="{{ $brand->id }}">{{ $brand->name }}</option>
                @endforeach
            </select>
        </p>
        <p>Nuotrauka: <input type="file" name="image"></p>
        <button type="submit">💾 Išsaugoti</button>
    </form>

    <a href="{{ route('products.index') }}">🔙 Grįžti</a>
</body>
</html>


3. edit.blade.php – prekės redagavimas

<!DOCTYPE html>
<html>
<head>
    <title>Redaguoti prekę</title>
</head>
<body>
    <h1>Redaguoti prekę</h1>

    @if($errors->any())
        <ul style="color:red;">
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    @endif

    <form action="{{ route('products.update', $product->id) }}" method="POST" enctype="multipart/form-data">
        @csrf
        @method('PUT')
        <p>Pavadinimas: <input type="text" name="name" value="{{ $product->name }}"></p>
        <p>Aprašymas: <textarea name="description">{{ $product->description }}</textarea></p>
        <p>Kaina: <input type="text" name="price" value="{{ $product->price }}"></p>
        <p>Kategorija: 
            <select name="category_id">
                @foreach($categories as $category)
                    <option value="{{ $category->id }}" @if($product->category_id == $category->id) selected @endif>{{ $category->name }}</option>
                @endforeach
            </select>
        </p>
        <p>Prekės ženklas:
            <select name="brand_id">
                @foreach($brands as $brand)
                    <option value="{{ $brand->id }}" @if($product->brand_id == $brand->id) selected @endif>{{ $brand->name }}</option>
                @endforeach
            </select>
        </p>
        <p>Nuotrauka: <input type="file" name="image"></p>
        <button type="submit">💾 Atnaujinti</button>
    </form>

    <a href="{{ route('products.index') }}">🔙 Grįžti</a>
</body>
</html>

4. show.blade.php – vienos prekės peržiūra

<!DOCTYPE html>
<html>
<head>
    <title>Prekės peržiūra</title>
</head>
<body>
    <h1>{{ $product->name }}</h1>
    <p><strong>Aprašymas:</strong> {{ $product->description }}</p>
    <p><strong>Kaina:</strong> {{ $product->price }} €</p>
    <p><strong>Kategorija:</strong> {{ $product->category->name ?? '-' }}</p>
    <p><strong>Prekės ženklas:</strong> {{ $product->brand->name ?? '-' }}</p>

    @if($product->image)
        <p><img src="{{ asset('storage/' . $product->image) }}" width="200"></p>
    @endif

    <a href="{{ route('products.index') }}">🔙 Grįžti</a>
</body>
</html>

 9 ŽINGSNIS – Sukuriam BrandSeeder su prekių ženklais

Terminale įvesk:
php artisan make:seeder BrandSeeder

Atidaryk database/seeders/BrandSeeder.php ir įrašyk:
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;

class BrandSeeder extends Seeder
{
    public function run(): void
    {
        DB::table('brands')->insert([
            ['name' => 'Apple'],
            ['name' => 'Samsung'],
            ['name' => 'Sony'],
            ['name' => 'Nike'],
            ['name' => 'Adidas']
        ]);
    }
}

Atidaryk DatabaseSeeder.php dar kartą ir pridėk šitą papildomai prie kitų:
$this->call([
    CategorySeeder::class,
    BrandSeeder::class,
]);

php artisan db:seed

9 žingsnis: Sukurti nuorodą tarp storage ir public

Terminale įvesk:
php artisan storage:link

Patikrink ar failas tikrai buvo įkelta
C:\xampp\htdocs\laravel_projektas\storage\app\public\images\

Eik į resources/views/products/index.blade.php ir pakeisk viską į šį kodą:
<!DOCTYPE html>
<html>
<head>
    <title>Prekių katalogas</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }
        .product-grid {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
        }
        .product-card {
            width: 300px;
            border: 1px solid #ccc;
            padding: 15px;
            border-radius: 10px;
        }
        .product-card img {
            width: 100%;
            height: auto;
            border-radius: 5px;
        }
        .actions {
            margin-top: 10px;
        }
        .actions a,
        .actions form {
            margin-right: 8px;
            display: inline-block;
        }
    </style>
</head>
<body>

    <h1>Prekių katalogas</h1>

    @if(session('success'))
        <p style="color: green;">{{ session('success') }}</p>
    @endif

    <a href="{{ route('products.create') }}">➕ Pridėti prekę</a>

    <div class="product-grid">
        @foreach($products as $product)
        <div class="product-card">
            <h3>{{ $product->name }}</h3>

            @if($product->image)
                <img src="{{ asset('storage/' . $product->image) }}" alt="Prekės nuotrauka">
            @endif

            <p><strong>Kategorija:</strong> {{ $product->category->name ?? '-' }}</p>
            <p><strong>Prekės ženklas:</strong> {{ $product->brand->name ?? '-' }}</p>
            <p><strong>Kaina:</strong> {{ $product->price }} €</p>

            <div class="actions">
                <a href="{{ route('products.show', $product->id) }}">Peržiūrėti</a>
                <a href="{{ route('products.edit', $product->id) }}">Redaguoti</a>
                <form action="{{ route('products.destroy', $product->id) }}" method="POST" style="display:inline;">
                    @csrf
                    @method('DELETE')
                    <button type="submit">Trinti</button>
                </form>
            </div>
        </div>
        @endforeach
    </div>

</body>
</html>


Atnaujink index.blade.php CSS dalį taip:
<style>
    body {
        font-family: Arial, sans-serif;
        background-color: #f8f9fa; /* Šviesus fonas */
        margin: 0;
        padding: 20px;
    }
    h1 {
        margin-bottom: 20px;
    }
    a {
        color: #6f42c1;
        text-decoration: none;
    }
    .product-grid {
        display: grid;
        grid-template-columns: repeat(2, 1fr); /* Tik 2 per eilę */
        gap: 20px;
    }
    .product-card {
        background-color: #fff;
        border: 1px solid #ddd;
        padding: 15px;
        border-radius: 10px;
        box-shadow: 0 0 10px rgba(0,0,0,0.05);
        transition: transform 0.2s;
    }
    .product-card:hover {
        transform: scale(1.02);
    }
    .product-card img {
        width: 100%;
        height: auto;
        border-radius: 5px;
        margin-top: 10px;
    }
    .actions {
        margin-top: 10px;
    }
    .actions a,
    .actions form {
        margin-right: 8px;
        display: inline-block;
    }
    .actions button {
        padding: 5px 10px;
    }
</style>

Įdėk aplink viską wrapper div'ą su klase container

<body>
    <div class="container">
        <h1>Prekių katalogas</h1>
        ...
        <div class="product-grid">
            @foreach($products as $product)
                <div class="product-card">
                    <!-- visa kortelės info čia -->
                </div>
            @endforeach
        </div>
    </div>
</body>


 
