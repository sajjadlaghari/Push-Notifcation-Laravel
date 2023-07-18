
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


### Add Route route/web.php

```
Route::get('/home', [App\Http\Controllers\HomeController::class, 'index'])->name('home');
Route::post('/save-token', [App\Http\Controllers\HomeController::class, 'saveToken'])->name('save-token');
Route::post('/send-notification', [App\Http\Controllers\HomeController::class, 'sendNotification'])->name('send.notification');
```


### Replace Home app/Http/Controllers/HomeController.php Controller with Below Code

```
<?php

namespace App\Http\Controllers;
  
use Illuminate\Http\Request;
use App\Models\User;
  
class HomeController extends Controller
{
  
    public function __construct()
    {
        $this->middleware('auth');
    }
  

    public function index()
    {
        return view('home');
    }
  
  
    public function saveToken(Request $request)
    {
        auth()->user()->update(['device_token'=>$request->token]);
        return response()->json(['token saved successfully.']);
    }
  
    
    public function sendNotification(Request $request)
    {
        $firebaseToken = User::whereNotNull('device_token')->pluck('device_token')->all();
          
        $SERVER_API_KEY = 'XXXXXX';
  
        $data = [
            "registration_ids" => $firebaseToken,
            "notification" => [
                "title" => $request->title,
                "body" => $request->body,  
            ]
        ];
        $dataString = json_encode($data);
    
        $headers = [
            'Authorization: key=' . $SERVER_API_KEY,
            'Content-Type: application/json',
        ];
    
        $ch = curl_init();
      
        curl_setopt($ch, CURLOPT_URL, 'https://fcm.googleapis.com/fcm/send');
        curl_setopt($ch, CURLOPT_POST, true);
        curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $dataString);
               
        $response = curl_exec($ch);
  
        dd($response);
    }
}

```



### Replace resources/views/home.blade.php With Below Code 


```

@extends('layouts.app')
   
@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <center>
                <button id="btn-nft-enable" onclick="initFirebaseMessagingRegistration()" class="btn btn-danger btn-xs btn-flat">Allow for Notification</button>
            </center>
            <div class="card">
                <div class="card-header">{{ __('Dashboard') }}</div>
  
                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif
  
                    <form action="{{ route('send.notification') }}" method="POST">
                        @csrf
                        <div class="form-group">
                            <label>Title</label>
                            <input type="text" class="form-control" name="title">
                        </div>
                        <div class="form-group">
                            <label>Body</label>
                            <textarea class="form-control" name="body"></textarea>
                          </div>
                        <button type="submit" class="btn btn-primary">Send Notification</button>
                    </form>
  
                </div>
            </div>
        </div>
    </div>
</div>
  
<script src="https://www.gstatic.com/firebasejs/7.23.0/firebase.js"></script>
<script>
  
    var firebaseConfig = {
        apiKey: "XXXX",
        authDomain: "XXXX.firebaseapp.com",
        databaseURL: "https://XXXX.firebaseio.com",
        projectId: "XXXX",
        storageBucket: "XXXX",
        messagingSenderId: "XXXX",
        appId: "XXXX",
        measurementId: "XXX"
    };
      
    firebase.initializeApp(firebaseConfig);
    const messaging = firebase.messaging();
  
    function initFirebaseMessagingRegistration() {
            messaging
            .requestPermission()
            .then(function () {
                return messaging.getToken()
            })
            .then(function(token) {
                console.log(token);
   
                $.ajaxSetup({
                    headers: {
                        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
                    }
                });
  
                $.ajax({
                    url: '{{ route("save-token") }}',
                    type: 'POST',
                    data: {
                        token: token
                    },
                    dataType: 'JSON',
                    success: function (response) {
                        alert('Token saved successfully.');
                    },
                    error: function (err) {
                        console.log('User Chat Token Error'+ err);
                    },
                });
  
            }).catch(function (err) {
                console.log('User Chat Token Error'+ err);
            });
     }  
      
    messaging.onMessage(function(payload) {
        const noteTitle = payload.notification.title;
        const noteOptions = {
            body: payload.notification.body,
            icon: payload.notification.icon,
        };
        new Notification(noteTitle, noteOptions);
    });
   
</script>
@endsection

```


### Create File in public/firebase-messaging-sw.js and Add Below Code

``` 
/*
Give the service worker access to Firebase Messaging.
Note that you can only use Firebase Messaging here, other Firebase libraries are not available in the service worker.
*/
importScripts('https://www.gstatic.com/firebasejs/7.23.0/firebase-app.js');
importScripts('https://www.gstatic.com/firebasejs/7.23.0/firebase-messaging.js');
   
/*
Initialize the Firebase app in the service worker by passing in the messagingSenderId.
* New configuration for app@pulseservice.com
*/
firebase.initializeApp({
        apiKey: "XXXX",
        authDomain: "XXXX.firebaseapp.com",
        databaseURL: "https://XXXX.firebaseio.com",
        projectId: "XXXX",
        storageBucket: "XXXX",
        messagingSenderId: "XXXX",
        appId: "XXXX",
        measurementId: "XXX"
    });
  
/*
Retrieve an instance of Firebase Messaging so that it can handle background messages.
*/
const messaging = firebase.messaging();
messaging.setBackgroundMessageHandler(function(payload) {
    console.log(
        "[firebase-messaging-sw.js] Received background message ",
        payload,
    );
    /* Customize notification here */
    const notificationTitle = "Background Message Title";
    const notificationOptions = {
        body: "Background Message body.",
        icon: "/itwonders-web-logo.png",
    };
  
    return self.registration.showNotification(
        notificationTitle,
        notificationOptions,
    );
});

```


## Run Server

``` php artisan serve  ```



### Create Firebase Console if you don't have yet
``` Link: https://console.firebase.google.com/ ```


### Click Add Project 

![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/7bce5e68-d286-497e-a579-4d93ec350d11)


### Enter Project Name

![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/2ab949b1-6624-422e-aef5-1f1663f4d280)## Push Notification



![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/2cddc0eb-7d14-4b4b-80d4-9f515869f576)



### Select Account 


![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/28494c3a-46b1-493e-817b-368e99d803d7)



### Select Web


![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/b275dc1d-1918-4386-b3fb-746f3116d111)



### Project name and Register 


![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/3c4cdd65-a6f5-4434-870d-4ac728badb8a)



### You will get Code Like Belew


![Untitled](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/75b9ceae-0205-4395-89ba-e4487e3161aa)



### Replace Below code in Home.blade php file and in public firebase-messaging.js with Above Screenshoot code

``` home.blade.php ```
 
![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/d122e7e5-0573-4cd6-b134-6ec85fcac1fc)


``` firebase-messaging.js ```


![image](https://github.com/sajjadlaghari/Push-Notifcation-Laravel/assets/68752819/29e97fe5-8c02-4adb-9a61-ffa14e6716e1)





