Database Testing
 

Introduction
Resetting The Database After Each Test
Using Migrations
Using Transactions
Writing Factories
Factory States
Using Factories
Creating Models
Persisting Models
Relationships
* Giới Thiệu  *

Laravel cung cấp một loạt các công cụ  tool hữu ích mà làm cho việc sử dụng cơ sở dữ liệu của bạn dễ dàng hơn. Đầu tiên bạn có thể sử dụng các  seeInDatabase  Hỗ trợ các thư viện kiểm tra khẳng định dữ liệu có tồn tại trong cơ sở dữ liệu. Khớp với các tiêu chí đã được thiết lập. 

Trong ví dụ của chúng tôi nếu bạn muốn xác minh rằng có một bản ghi của table user với email có giá trị là  sally@example.com   bạn có thể sử dụng.  

Đoạn mã trong  seeInDatabase trông như thế này :


``` 
public function testDatabase()
{
    // Make call to application...

    $this->seeInDatabase('users', [
        'email' => 'sally@example.com'
    ]);
}
```


Tất nhiên method seeInDatebase là hỗ trợ cho tiện việc xác định. Bạn cũng có thể sử dụng bất kỳ các phương pháp có trong PHPUnit để bổ sung cho việc thử nghiệm của bạn.

* Reset Database Sau Mỗi thử nghiệm *

Nó thường là hữu ích để thiết lập lại cơ sở dữ liệu của bạn sau mỗi bài kiểm tra để các dữ liệu từ một thử nghiệm trước đó không can thiệp với các xét nghiệm tiếp theo.

* Sử dụng Migrations *

Một cách tiếp cận để đặt lại trạng thái cơ sở dữ liệu là rollback cho cơ sở dữ liệu sau mỗi lần kiểm tra.
Laravel cung cấp một đơn giản DatabaseMigrations tính trạng đó sẽ tự động xử lý này cho bạn Đơn giản chỉ cần sử dụng trên trait lớp giao diện các method có sẵn sẽ hỗ trợ.

```
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
    use DatabaseMigrations;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $this->visit('/')
             ->see('Laravel 5');
    }
}
```
* Sử dụng Transactions *

Một cách khác để đặt lại trạng thái cơ sở dữ liệu là để bọc từng trường hợp thử nghiệm bên trong một giao dịch transactions trait  lớp giao diện mà có các method hỗ trợ nó sẽ tự động sử lý cho bạn.

```
<?php

use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Illuminate\Foundation\Testing\DatabaseTransactions;

class ExampleTest extends TestCase
{
    use DatabaseTransactions;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $this->visit('/')
             ->see('Laravel 5');
    }
}
```



Viết  Factories

Khi thử nghiệm người ta cần một vài bản ghi được chèn trước vào trong cơ sở dữ liệu của bạn.
Thay vì tự xác định và viết các bản ghi này bạn có thể tự động tạo ra và thêm nó vào trong cơ sở dữ liệu.

Laravel cung cấp cho bạn mặc định các thuộc tính của Eloquent models mà sẽ sử dụng model factories. 

Để tiếp tục nhìn vào file database/factories/ModelFactory.php  trong ứng dụng của bạn. Trong tập tin này có chứa 1 định nghĩa factories.

```
$factory->define(App\User::class, function (Faker\Generator $faker) {
    static $password;

    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'password' => $password ?: $password = bcrypt('secret'),
        'remember_token' => str_random(10),
    ];
});
```

Trong method function define “định nghĩa factories”  cung cấp function đóng cửa “Closure” mà sẽ phục vụ App\User::class Model.
 
Bạn có thể trả lại tất cả các thuộc tính mặc định ban đầu được tự động tạo cho các thuộc tính model. 

Bạn có thể thấy đóng cửa phương pháp function() trả lại một thể hiện của class  Faker\Generator $faker một thư viện của php  “https://github.com/fzaninotto/Faker” mà tự động tạo ra các dữ liệu có giá trị mặc định.

Tất nhiên bạn có thể tự động tạo ra các file giống như ModelFactory.php  file. Ví dụ như các model User hay Comment 
giống như UserFactory.php và CommentFactory.php tập tin trong bạn database/factories thư mục.

Tất cả các tập tin trong các factories thư mục sẽ tự động được load bởi Laravel.


* Factory States *

Có thể sửa đổi các giá trị có trong model factory từ mọi như như một sư phối hợp.
Trong ví dụ dưới đây có một giá trị mặc định của một thuộc tính có trong model là account_status bạn có thể định nghĩa nó để nó nhận sự thay đổi bằng cách truyền vào tham số thứ 2.

được gọ là một state method mà có thể thay đổi  :


$factory->state(App\User::class, 'delinquent', function ($faker) {
    return [
        'account_status' => 'delinquent',
    ];
});

Sử dụng Factory

Tạo ra model 
Một khi bạn đã định nghĩa  factory cho model của bạn, bạn có thể sử dụng chức năng factory() toàn cầu  trong testing hoặc seed files để tự động tạo các trường của model.
Chúng ta cùng xem một ví dụ về việc tạo dữ liệu cho model. Đầu tiên chúng ta sẽ sử dụng một method make()  sex bắt đầu làm việc tạo ra các dữ liệu model nhưng chưa lưu chúng vào cơ sở dữ liệu.

Factory() sử dụng trong  seed files :
``` 
public function testDatabase()
{
    $user = factory(App\User::class)->make();

    // Use model in tests...
}
```
bạn cũng có thể tạo ra hàng loạt bản ghi cho một model cua bạn bằng cách :
```
// Tạo ra ba bản ghi cho  App\User ví dụ ...
$users = factory(App\User::class, 3)->make();

// tạo ra một 1 bản ghi "admin" cho App\User ví dụ ...
$user = factory(App\User::class, 'admin')->make();

// Tạo ra ba bản ghi có giá trị admin cho App\User ví dụ ...
$users = factory(App\User::class, 'admin', 3)->make();
```
Applying States

Bạn cũng có thể áp dụng state bằng cách định nghĩa một thay đổi hoặc nhiều thay đổi giá trị của mặc định bằng 

$users = factory(App\User::class, 5)->states('deliquent')->make();
