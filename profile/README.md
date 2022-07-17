# apistorm

Create stractured APIs and classified responses that can know by modern IDEs

## What is problem?

A big problem in most SDKs is input/output array|objects

For example you have a [virtualizor api SDK](https://github.com/bennetgallein/VirtualizorPHP)!

In this SDK for use of every method you should read documents
is [virtualizor site](https://www.virtualizor.com/admin-api/create-vps)

For example i want use ``addsrv`` method:


```php
$sdk->addsrv(
[
    'serid' => 0,
    'slave_server' => 1, // Pass the slave server ID if you want to create VPS on the Slave Server
    'virt' => 'kvm',
    'uid' => 0,
    'user_email' => 'test@test.com',
    'user_pass' => 'test123',
    'fname' => '',
    'lname' => '',
    'plid' => 0,
    'osid' => 88,
    'hostname' => 'test12345.com',
    'rootpass' => 'test123',
    'ips' => ['192.168.111.49'],
    'num_ips6' => 0,
    'num_ips6_subnet' => 0,
    'stid' => 1,
    'space' => [
                0=>[
                     'size' => 2,
                     'st_uuid'=>'ngavdbegidnbme2d'
                   ],
                1=>[
                    'size'=>2,
                    'st_uuid'=>'ngavdbegidnbme2d'
                   ],
                   
                ],//For VPS with Multi-Disk
    'ram' => 1024,
    'swapram' => 0,
    'bandwidth' => 0,
    'network_speed' => 0,
    'cpu' => 1000,
    'cores' => 4,
    'cpu_percent' => 100.00,
    'vnc' => 1,
    'vncpass' => 'test123',
    'kvm_cache' => 0,
    'io_mode' => 0,
    'vnc_keymap' => 'en-us',
    'nic_type' => 'default',
    'osreinstall_limit' => 0,
]
);
```

This is dirty for an API method

You can use this api just after read API documents from their website everytime,because you dont know array parameters

Then you'll get an array after request from this method!!!

This is wrong and dirty to

## What is ``apiStorm``?

apiStorm can classify your input/output arguments for every API method

By ``apiStorm``, every modern IDE can parse your input arguments and your output method data

You can validate data before send to API with ``apiStorm``

## What is different in `apiStorm`

This simple traditional API dataSend:

```php
$api = new TestApi();//This is a test API SDK class
$response=$api->createProduct(//This method will send data to server
    [//Our data is an array!
        'name'=>'Pen',//We dont know this parameter is require on optional!
        'category'=>10,//We can't validate data with simple not classified array input
        'color'=>'#fffff',//We dont know accepted data types for every array index
        'status'=>1,
        'price'=>3000
    ]
);
//$response is an array! and we dont know what is happening in every method without read documents
```

This is modern API with apiStorm:

```php
$api = new TestApi();//This is a test API SDK class
$data = new PostCreateProduct();//This is our input Class that extended from apiStorm classes

$data->name0 = 'Pen';//This is an required input field because its ends with "0"
$data->category0 = 10; 
$data->color0 = "#fffff";//if fields that ends with "0" be empty, then will be error on validate
$data->status0 = 1;//Now name of this field is "status0" that's mean this field is required but when data sending to server it will convert to "status"
$data->price=3000;//This is an optional input field

if ($data->validate()) {//You can validate data types before send
    $response = $api->createProduct($data);//This method will send data to server
    if ($response && $response->isSuccess()) {//Here we check status of sent request
        echo $response->created_id;//If success we have classified response 
    }else{
        var_export($response->getError());//If error, we have classified error
    }
}
```

In second example ``PHPStorm`` can parse `input/output` fileds like this:

### Input Fields in `PHPSTORM`:

![](https://github.com/yiiman-dev/apistorm/blob/main/.pictures/1.png)

### Classified response in `PHPSTORM`:

![](https://github.com/yiiman-dev/apistorm/blob/main/.pictures/2.png)


## Install apiStorm
`composer require yiiman/apistorm`

OR

```json
"require": {
    "yiiman/apistorm": "^0.0.1"
  }
```


## concept

Suggested strusture for standard SDK is like this:

```
---src\
    |
    ---Responses\
        |
        ---ResponseClass1.php                      // extended from YiiMan\ApiStorm\Response\BaseResponse
        |--- public $responseField1='int';         // you can define one of this data types : int|string|float|serialize|json|array|class|classArray
        |--- public $responseField2='';            // if you set empty string, its main string type
        |
        ---ResponseClass2.php                      // extended from YiiMan\ApiStorm\Response\BaseResponse
        |--- public $responseField1='int';         // you can define one of this data types : int|string|float|serialize|json|array|class|classArray
        |--- public $responseField2='';            // if you set empty string, its main string type
        |
        .
        .
        .
    |
    ---Posts\
       |
       ---PostClass1.php                          // extended from YiiMan\ApiStorm\Post\BasePostData
       |--- public int    $field0;                // Required fields will ends by "0"
       |--- public string $anotherField='test';   // Optional field
       |--- public function rules(): array
                {
                    return
                        [
                            'field'             => 'int',//You should define input field type: int|string|array|float|object
                            'anotherField'      => 'string',
                        ];
                }
       |
       ---PostClass2.php                          // extended from YiiMan\ApiStorm\Post\BasePostData
       |--- public array $field0;                 // Required fields will ends by "0"
       |--- public int   $anotherField=2;         // Optional field
       |--- public function rules(): array
                {
                    return
                        [
                            'field'             => 'float',//You should define input field type: int|string|array|float|object
                            'anotherField'      => 'int',
                        ];
                }
       |
       |
       .
       .
       .
    |
    --- SDKClass.php
    |--- public $protocol = 'https';
    |--- public $baseURl = 'someURL';
    |--- firstMethod(PostClass1 $data):ResponseClass1{
        if ($data->validated()) {

            // you will send $data to your server
            $response = $this->call('path/to/api/url', $data);


            if ($response->isSuccess()) {
                $response = new CreateProductResponse($response);
            }
            // <  Here, you have classified response >
              return $response;
            // </ Here, you have classified response >
        } else {
            return false;
        }
    }
    |
    .
    .
    .
  
```


## Usage

### First way

Clone project and check [index.php](https://github.com/yiiman-dev/apistorm/blob/main/index.php) file for full example
you can execute [index.php](https://github.com/yiiman-dev/apistorm/blob/main/index.php) in console to see results:

`composer install`


`php index.php`

### Second way
#### Step 1
Create new class for your SDK with name `TestApi.php`:

```php
class TestApi
{
    public $protocol = 'https';
    public $baseURl = 'n8.yiiman.ir/webhook';
}
```

We need a call method for our connections, then we need config an instant of ``ApiStorm``  `call` method inside our call method like this:
```php
    /**
     * @param  YiiMan\ApiStorm\Post\BasePostData  $dataClass
     * @return YiiMan\ApiStorm\Core\Res
     */
    private function call($path, $dataClass, $method = 'post')
    {
        $servedArrayOfDataClass = $dataClass->serve();
        $connection = new YiiMan\ApiStorm\Core\Connection();
        $connection->baseURL = $this->baseURl;
        $connection->protocol = 'https';

        return $connection->call($path, [], $servedArrayOfDataClass, [],$method);
    }

```

Use ``src/examples`` directory to create new API SDK

Now we need some method for use `call` for create connection in our [`TestApi.php`](https://github.com/yiiman-dev/apistorm/blob/main/src/examples/TestApi.php) class like this:

```php
class TestApi
{
    public $protocol = 'https';
    public $baseURl = 'n8.yiiman.ir/webhook';
    
    
    /**
     * @param  YiiMan\ApiStorm\Post\BasePostData  $dataClass
     * @return YiiMan\ApiStorm\Core\Res
     */
    private function call($path, $dataClass, $method = 'post')
    {
        $servedArrayOfDataClass = $dataClass->serve();
        $connection = new YiiMan\ApiStorm\Core\Connection();
        $connection->baseURL = $this->baseURl;
        $connection->protocol = 'https';

        return $connection->call($path, [], $servedArrayOfDataClass, [],$method);
    }
    
    
    
    /**
     * @param  PostCreateProduct  $product
     * @return CreateProductResponse|bool
     */
    public function createProduct(PostCreateProduct $product)
    {
        if ($product->validated()) {

            // you will send $product to your server
            $response = $this->call('6c81aa91-63a1-43d9-abb6-6b5398716f81', $product,'get');


            if ($response->isSuccess()) {
                $response = new CreateProductResponse($response);
            }

            return $response;
            // </ Here, you will classify response >
        } else {
            return false;
        }
    }

     /**
     * @param  PostCreateProduct  $product
     * @return CreateProductResponse|bool
     */
    public function createProduct2(PostCreateProduct $product)
    {
        if ($product->validated()) {

            // you will send $product to your server
            $response = $this->call('6c81aa91-63a1-43d9-abb6-6b5398716f81', $product);


            if ($response->isSuccess()) {
                $response = new CreateProductResponse($response);
            }

            return $response;
            // </ Here, you will classify response >
        } else {
            return false;
        }
    }
}
```

#### Step 2

## Credits
Special thanks to [arianet](https://www.ariaservice.net) Company
