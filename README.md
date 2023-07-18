## Push Notification

``` Demo ```
### Push Notification sent
![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/cc6a5ca1-2982-472f-a832-b683dcadaa02)

### Push Notification recieved
![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/8a9b4670-40cf-464a-ac01-dccfd9824257)


## Create Project

``` composer create-project --prefer-dist laravel/laravel PushNotifcation ```

## Run command

``` composer require laravel/ui  ```



## Run command

``` php artisan ui bootstrap --auth   ```


## Run command

``` npm install ```


## Run command

``` npm run dev  ```

## Run command

```  npm install --save-dev vite laravel-vite-plugin ```

```  npm install --save-dev @vitejs/plugin-vue  ```
```  npm run build ```



## Add columm in user Table

``` php artisan make:migration add_column_device_token ```

### database/migrations/2020_10_23_144523_add_column_device_token.php

```
<?php
  
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;
  
class AddColumnDeviceToken extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->string('device_token')->nullable();
        });
    }
  
    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
           
    }
}

```

### Replace app/Models/User.php Modal Below with Code

```
<?php
  
namespace App\Models;
  
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
  
class User extends Authenticatable
{
    use HasFactory, Notifiable;
  
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name',
        'email',
        'password',
        'device_token'
    ];
  
    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password',
        'remember_token',
    ];
  
    /**
     * The attributes that should be cast to native types.
     *
     * @var array
     */
    protected $casts = [
        'email_verified_at' => 'datetime',
    ];
}

```


