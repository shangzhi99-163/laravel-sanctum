
### Step 1: setup database in .env file

```` 
DB_DATABASE=youtube
DB_USERNAME=root
DB_PASSWORD= redhat@123
````

## Step 2:Install Laravel Sanctum.

````
composer require laravel/sanctum
````

## Step 3:Publish the Sanctum configuration and migration files .

````
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

````

## Step 4:Run your database migrations.

````
php artisan migrate


 // mysql 5.7.7以下版本
app\Providers\AppServiceProvider.php
    public function boot()
    {
        Schema::defaultStringLength(191);
    }

````

## Step 5:Add the Sanctum's middleware.

````
../app/Http/Kernel.php

use Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful;

...

    protected $middlewareGroups = [
        ...

        'api' => [
            EnsureFrontendRequestsAreStateful::class,
            'throttle:60,1',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
    ];

    ...
],

````

## Step 6:To use tokens for users.

````
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
  use HasApiTokens,Notifiable;
    public function createToken(string $name, array $abilities = ['*'])
    {
        $token = $this->tokens()->create([
            'name' => $name,
            'token' => hash('sha256', $plainTextToken = Str::random(80)),
            'abilities' => $abilities,
        ]);

        return new NewAccessToken($token, $plainTextToken);
    }
}

````

## Step 7:Let's create the seeder for the User model

```javascript 
php artisan make:seeder UsersTableSeeder
````

## Step 8:Now let's insert as record

```javascript 
DB::table('users')->insert([
    'name' => 'Mr Jiang',
    'email' => 'test@163.com',
    'password' => Hash::make('123456')
]);
````

## Step 9:To seed users table with user

```javascript 
php artisan db:seed --class=UsersTableSeeder
````


## Step 10:  create a controller nad  /login route in the routes/api.php file:


```javascript 
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\User;
use Illuminate\Support\Facades\Hash;
class UserController extends Controller
{
    // 

       function index(Request $request)
    {
        if (Auth::attempt(['email' => $request->email, 'password' => $request->password])) {
            $user = User::where('email', $request->email)->first();
            $token = $user->createToken('my-app-token')->plainTextToken;
            $response = [
                'user' => $user,
                'token' => $token
            ];
            return response($response, 201);
        } else {
            return response([
                'message' => ['账号或密码错误']
            ], 404);
        }

    }

    function users()
    {
        return Auth::user();
    }

    function logout()
    {
        Auth::logout();
        return response('logout');
    }
}


````


## Step 11: Test with [ VUE AXIOS ]

```javascript 

           Login: function () {
                let params_json = {
                    email: 'zhangsan4@163.com',
                    password: '123456',
                };
                axios.post('http://www.cors.com/api/login', params_json, {headers: {'Content-Type': 'application/json;charset=utf-8'}}).then(res => {
                    console.log(res.data);
                    localStorage.setItem('IsLogin', 'true');
                    localStorage.setItem('Token', res.data.token)
                }).catch(error => {
                    console.log('请求失败', error);
                });
            },
            Logout: function () {
                axios.post('http://www.cors.com/api/logout').then(res => {
                    localStorage.removeItem('IsLogin');
                    localStorage.removeItem('Token');
                    console.log('LogOut',res);
                }).catch(error => {
                    console.log('请求失败', error);
                });

            },
            getUser: function () {
                let headers = {
                    'Content-Type': 'application/json;charset=utf-8',
                    'Authorization': 'Bearer '+ localStorage.getItem('Token'),

                };
                axios.get('http://www.cors.com/api/users', {
                    headers: headers
                }).then(res => {
                    console.log(res.data);
                }).catch(error => {
                    console.log('请求失败', error);
                });
            },

````

## Step 11: Make Details API or any other with secure route  

```javascript 

// 无需登录操作
Route::post('login', 'UserController@index');


Route::middleware('auth:sanctum')->get('users', 'UserController@users');

//登录后操作
Route::group(['middleware' => ['auth:sanctum']], function () {
    Route::get('users', 'UserController@users');
    Route::post('logout', 'UserController@logout');

````
  
    
