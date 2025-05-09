# prekiu-katalogas


1 ŽINGSNIS: Laravel projekto sukūrimas

composer create-project laravel/laravel .
 Komentaras- taškas . reiškia, kad Laravel bus įdiegtas į tą patį (laravel_projektas) katalogą
 
-------------------------------------------------

2 ŽINGSNIS: Sukuriam duomenų bazę

Atidaryk http://localhost/phpmyadmin naršyklėje.

Viršuje spausk New.

Sukurk bazę pavadinimu: laravel (arba kitokį pavadinimą – bet turėsi jį įrašyti į .env failą).

Spausk Create.


Dabar VS Code'e paleisk Laravel serverį:

php artisan serve

Starting Laravel development server at http://127.0.0.1:8000

------------------------

Dabar tęsiam: 3 ŽINGSNIS – Migracijos ir lentelės
Sukursim šias lenteles:

categories – užpildysim iškart per seeder (pvz.: Drabužiai, Technika, Buitis).

products – pagrindinė lentelė.

brands – papildoma, kad parodytume JOIN.

comments – kad galima būtų komentuoti prekes.

users – bus tik tam, kad turėtume naudotojų ID komentarams, bet be login.

Terminale įvesk (per VS Code terminalą):

php artisan make:migration create_categories_table
php artisan make:migration create_brands_table
php artisan make:migration create_products_table
php artisan make:migration create_comments_table
php artisan make:migration create_users_table


 Tada atsidaryk šiuos failus kataloge:
database/migrations/ – ir papildyk juos taip (PILNAI VISAS KODAS):

create_categories_table.php - 

<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('categories');
    }
};

 create_brands_table.php - 

 <?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('brands', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('brands');
    }
};
create_products_table.php - 
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('products', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description');
            $table->decimal('price', 8, 2);
            $table->string('image')->nullable();
            $table->unsignedBigInteger('category_id');
            $table->unsignedBigInteger('brand_id');
            $table->timestamps();

            $table->foreign('category_id')->references('id')->on('categories');
            $table->foreign('brand_id')->references('id')->on('brands');
        });
    }

    public function down(): void {
        Schema::dropIfExists('products');
    }
};
create_comments_table.php-
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('product_id');
            $table->unsignedBigInteger('user_id');
            $table->text('comment');
            $table->timestamps();

            $table->foreign('product_id')->references('id')->on('products');
            $table->foreign('user_id')->references('id')->on('users');
        });
    }

    public function down(): void {
        Schema::dropIfExists('comments');
    }
};

create_users_table.php -
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique()->nullable(); // jei naudosi, bet galima palikti
            $table->string('identification_number'); // dėl reikalavimo
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('users');
    }
};

Tada terminale:
php artisan migrate

 
Tai bus tavo duomenų bazės struktūros 1 versijos pradzia
