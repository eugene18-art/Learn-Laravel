# Rest API CRUD With Passport Auth Tutorial
* [steps](https://www.phpcodingstuff.com/blog/laravel-8-rest-api-crud-with-passport-auth-tutorial.html)

* Install laravel with project name blog
    ```
    composer create-project project --prefer-dist laravel/laravel blog
    ```
* Install laravel passport
    ```
    composer require laravel/passport
    ```
* In config/app.php
    ```php
    'providers' => [
        Laravel\Passport\PassportServiceProvider::class,
    ],
    ```
* Migrate and integrate passport to your project
    ```
    cd blog && php artisan migrate && php artisan passport:install
    ```
* Register Passport in app\Providers\AuthServiceProvider.php
    ```php
    protected $policies = [
        'App\Models\Model' => 'App\Policies\ModelPolicy',
    ];
    ```
* In config/auth.php
    ```php
    'api' => [ 
        'driver' => 'passport', 
        'provider' => 'users', 
    ], 
    ```

* Create model
    ```
    php artisan make:model Product -m
    ```
* Configure Product migration
    ```php
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->text('detail');
            $table->timestamps();
            
        });
    }
    ```
* Configure Product model
    ```php
    class Product extends Model
    {
        use HasFactory;
        protected $fillable = [
            'name', 
            'detail'
        ];
    }
    ```
* Migrate to create your db table
    ```bash
    php artisan migrate
    ```
* Configure api.php
    ```php
    use App\Http\Controllers\Api\PassportAuthController;
    use App\Http\Controllers\Api\ProductController;
     
    Route::post('register', [PassportAuthController::class, 'register']);
    Route::post('login', [PassportAuthController::class, 'login']);
      
    Route::middleware('auth:api')->group(function () {
        Route::get('get-user', [PassportAuthController::class, 'userInfo']);
     Route::resource('products', [ProductController::class]);
    });
    ```
* Create controllers
    ```
    php artisan make:controller Api\ProductController --resource && php artisan make:controller Api\PassportAuthController
    ```
* In App\Api\PassportAuthController
    ```php
    <?php

    namespace App\Http\Controllers\Api;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use App\Models\User;
    class PassportAuthController extends Controller
    {
        /**
        * Registration Req
        */
        public function register(Request $request) {
            $this->validate($request, [
                'name'=>'required|min:4',
                'email'=>'required|email',
                'password'=>'required|min:8',
            ]);
            $user = User::create([
                'name'=>$request->name,
                'email'=>$request->email,
                'password'=>bcrypt($request->password)
            ]);
            $token = $user->createToken('Laravel8PassportAuth')->accessToken;
            return response()->json(['token'=>$token], 200);
        }

        /**
        * Login Req
        */
        public function login(Request $request) {
            $data = [
                'email'=>$request->email,
                'password'=>$request->password,
            ];

            if (auth()->attempt($data)) {
                $token = auth()->user()->createToken('Laravel8PassportAuth')->accessToken;
                return response()->json(['token'=>$token], 200);
            } else {
                return response()->json(['error'=>'Unauthorized'], 401);
            }
        }

        public function userInfo() {
            $user = auth()->user();
            return response()->json(['user'=>$user], 200);
        }
    }
    ```
* In App\Api\ProductController
    ```php
    <?php

    namespace App\Http\Controllers\Api;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use App\Models\Product;
    use Validator;

    class ProductController extends Controller
    {
        /**
        * Display a listing of the resource.
        *
        * @return \Illuminate\Http\Response
        */
        public function index()
        {
            $products = Product::all();
            return response()->json([
                "success"=>true,
                "message"=>"Product List",
                "data"=>$products
            ]);
        }

        /**
        * Store a newly created resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @return \Illuminate\Http\Response
        */
        public function store(Request $request)
        {
            $input = $request->all();
            $validator = Validator::make($input, [
                'name'=>'required',
                'detail'=>'required',
            ]);
            if ($validator->fails()) {
                return $this->sendError('Validation Error.', $validator->errors());
            }
            $product = Product::create($input);
            return response()->json([
                "success"=>true,
                "message"=>"Product created successfully",
                "data"=>$product
            ], 201);
        }

        /**
        * Display the specified resource.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */
        public function show($id)
        {
            $product = Product::find($id);
            if (is_null($product)) {
                return $this->sendError('Product not found.');
            }
            return response()->json([
                "success"=>true,
                "message"=>"Product retrieved successfully.",
                "data"=>$product
            ]);
        }

        /**
        * Update the specified resource in storage.
        *
        * @param  \Illuminate\Http\Request  $request
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */
        public function update(Request $request, Product $product)
        {
            $input = $request->all();
            $validator = Validator::make($input, [
                'name'=>'required',
                'detail'=>'required'
            ]);
            if ($validator->fails()) {
                return $this->sendError('Validation error', $validator->errors());
            }
            $product->name = $input['name'];
            $product->detail = $input['detail'];
            $product->save();
            return response()->json([
                "success"=>true,
                "message"=>"Product updated successfully.",
                "data" => $product
            ], 201);
        }

        /**
        * Remove the specified resource from storage.
        *
        * @param  int  $id
        * @return \Illuminate\Http\Response
        */
        public function destroy(Product $product)
        {
            $product->delete();
            return response()->json([
                "success"=>true,
                "message"=>"Product deleted successfully.",
                "data"=>$product
            ], 204);
        }
    }

    ```

* Failure with route login is not defined but clearly it's defined. Not clear where is the error.
* Turns out the error is you must name the route like bellow
    ```php
    Route::post('register', [PassportAuthController::class, 'register'])->name('register');
    Route::post('login', [PassportAuthController::class, 'login'])->name('login');
    ```
* To use the token from login response. Add variable to your http header
    ```javascript
    'Accept':'application/json'
    'Authorization’:'Bearer your_token'
    ```
* Then I test the Api again and **sendError** function does not exists. I look for exetended controller but it does not implement it. I finally made the function myself insede Product Controller.
    ```php
    class ProductController extends Controller
    {
        private function sendError($type, $error_msg, $code=null) {
            return response()->json([
                "success"=>false,
                "type"=>$type,
                "message"=>$error_msg,
            ], (is_null($code) ? 400 : $code));
        }
        //... other functions bellow
    }
    ```
    I also make little adjustment inside **show** function
    ```php
    public function show($id) {
        // .. code before
        if (is_null($product)) {
            return $this->sendError('Not Found', 'Product not found.', 404);
        }
        // .. code after
    }
    ```