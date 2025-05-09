10 ŽINGSNIS: Sukuriam Comment modelį su migracija

Terminale paleisk:
php artisan make:model Comment -m

Atidaryk database/migrations/XXXX_create_comments_table.php
Ir pakeisk į šį pilną kodą:
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('product_id');
            $table->string('author');
            $table->text('content');
            $table->timestamps();

            $table->foreign('product_id')->references('id')->on('products')->onDelete('cascade');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
};


Paleisk migraciją

php artisan migrate

Comment modelis ir ryšys

app/Models/Comment.php (jeigu jo dar nėra, sukurk: php artisan make:model Comment)

<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;

class Comment extends Model
{
    use HasFactory;

    protected $fillable = [
        'product_id',
        'author',
        'content',
    ];

    public function product()
    {
        return $this->belongsTo(Product::class);
    }
}

Product modelyje pridėk komentarų ryšį

public function comments()
{
    return $this->hasMany(Comment::class);
}
Sukuriam CommentController
php artisan make:controller CommentController

 Įrašyk šį pilną kodą į
 app/Http/Controllers/CommentController.php:


 <?php

namespace App\Http\Controllers;

use App\Models\Comment;
use Illuminate\Http\Request;

class CommentController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([
            'product_id' => 'required|exists:products,id',
            'author' => 'required|min:2',
            'content' => 'required|min:3',
        ]);

        Comment::create([
            'product_id' => $request->product_id,
            'author' => $request->author,
            'content' => $request->content,
        ]);

        return back()->with('success', 'Komentaras pridėtas sėkmingai!');
    }
}
Pridėk maršrutą į routes/web.php:
use App\Http\Controllers\CommentController;

Route::post('/comments', [CommentController::class, 'store'])->name('comments.store');


Pridėk komentarų formą ir atvaizdavimą į resources/views/products/show.blade.php (apačioje):

<hr>

<h3>Palikti komentarą</h3>

@if(session('success'))
    <p style="color: green">{{ session('success') }}</p>
@endif

<form action="{{ route('comments.store') }}" method="POST">
    @csrf
    <input type="hidden" name="product_id" value="{{ $product->id }}">

    <div>
        <label>Vardas:</label><br>
        <input type="text" name="author" required>
    </div>

    <div>
        <label>Komentaras:</label><br>
        <textarea name="content" rows="4" required></textarea>
    </div>

    <button type="submit">Komentuoti</button>
</form>

<hr>

<h3>Komentarai:</h3>
@forelse ($product->comments as $comment)
    <p><strong>{{ $comment->author }}</strong> sako:</p>
    <p>{{ $comment->content }}</p>
    <hr>
@empty
    <p>Komentarų dar nėra.</p>
@endforelse

Įdiek dompdf biblioteką
composer require barryvdh/laravel-dompdf

Sukurk PDF maršrutą ir metodą ProductController

Route::get('/products-pdf', [ProductController::class, 'generatePDF'])->name('products.pdf');


Sukurk PDF generavimo metodą
ProductController.php faile pridėk metodą:


use Barryvdh\DomPDF\Facade\Pdf; // viršuje

public function generatePDF()
{
    $products = Product::with('category', 'brand')->get();
    $pdf = Pdf::loadView('products.pdf', compact('products'));
    return $pdf->download('prekiu_katalogas.pdf

Maršrutas vienos prekės PDF generavimui
Route::get('/products/{product}/pdf', [ProductController::class, 'generateSinglePDF'])->name('products.single.pdf');


Metodas ProductController faile

ProductController.php faile pridėk šį metodą:

use Barryvdh\DomPDF\Facade\Pdf; // jei dar nepridėta

public function generateSinglePDF(Product $product)
{
    $product->load('category', 'brand'); // įkeliame ryšius
    $pdf = Pdf::loadView('products.single_pdf', compact('product'));
    return $pdf->download($product->name . '_preke.pdf');
}

