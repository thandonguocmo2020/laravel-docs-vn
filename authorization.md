### Authorization - Uỷ quyền cho phép truy cập dữ liệu

- [Giới thiệu](#Introduction)
- [Định nghĩa quyền](#defining-abilities)
- [Kiểm tra quyền](#CheckingAbilities)
- [Sử dụng  Gate Facade kiểm tra quyền](#ckeckgatefacade)
- [Sử dụng User Model kiểm tra quyền](#usergate)
- [Kiểm tra trong Blade Templates](#gateblade)
- [Kiểm tra trong Form Requests](#formrequest)
- [Policies](#Policies)
-  [Creating Policies](#create-prolicies)
-  [Register Policies](#register-prolicies)
-  [Writing Policies](#writing-plicies)
-  [Checking Policies](#check-prolicies
- [Controller Authorization]


<a name="Introduction"></a>
### Giới thiệu

Ngoài việc cung cấp thư viện xác thực người dùng laravel còn cung cấp dịch vụ để tổ chức ủy quyền để cho phép và không cho phép người dùng truy cập vào nguồn dữ liệu. Việc kiểm soát truy cập có nhiều phương pháp để hỗ trợ bạn trong việc ủy quyền chúng tôi sẽ giới thiệu bạn trong tài liệu này.


<a name="defining-abilities"></a>
[Định nghĩa một quyền chức năng](#)

Cách đơn giản nhất để xác định xem người dùng có thể thực hiện một hành động được đưa ra là để xác định một quyền hạn của người user hiện tại sử dụng lớp `Illuminate\Auth\Access\Gate` để định nghĩa.


Class AuthServiceProvider là nơi thuận tiện để định nghĩa một quyền hay một khả năng có thể xảy ra trong ứng dụng của bạn.

Ví dụ chúng ta có thể định nghĩa một khả năng sẽ xảy ra mà nhận diện được người dùng hiện tại và id một bài viết để xem user này có quyền truy cập post có id đó hay không.

update-post là một quyền dùng để kiểm tra xem user đó có được cho phép truy cập tài nguyên hay không.


    <?php
    namespace App\Providers;
    use Illuminate\Contracts\Auth\Access\Gate as GateContract;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Đăng ký bất kỳ ứng dụng xác thực ủy quyền dịch vụ
         *
         * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
         * @return void
         */
        public function boot(GateContract $gate)
        {
            $this->registerPolicies($gate);
         $gate->define('update-post', function ($user, $post) {
                return $user->id == $post->user_id;
            });
        }
    }
    

Lưu ý chúng tôi không thể kiểm tra được người dùng user đó là không xác định.
Các Gate sẽ tự động trả về false vì các khả năng khi người dùng không xác thực hoặc một người dùng được xác định. bằng sử dụng phương pháp [forUser](#forUser).

Bạn rõ ràng có thể thử kiểm tra quyền một người dùng không xác thực bằng cách sử dụng hành vi này forUser

Việc gọi function action luôn cung cấp cho bạn một $user biến là đối số thứ nhất. Bạn không cần phải sử dụng nó trong mã của bạn!  Nhưng hãy để nó ở đó vì bạn có thể cần sử dụng nó trong tuong lai. 

<a name="base-abilities"></a>
[Định nghĩa quyền cơ bản](#)

    Ngoài việc định nghĩa một chức năng ủy quyền trong phương pháp khép kín "function ($user, $post)" bạn có thể định nghĩa sử dụng một method để check quyền. 

    Bạn có thể sử dụng một class method như một chuỗi khi cần thiết service container sẽ giải quyết nó để lấy class.

$gate->define('update-post', 'Class@method');

[Chặn kiểm tra ủy quyền](#)

Đôi khi bạn muốn cung cấp tất cả các quyền với một người dùng cụ thể việc sử dụng các method `before` để xác định một hành động được cần làm trước khi một quyền hạn được gọi.

     isSupperAdmin sẽ cung cấp tất cả quyền cho user hiện tại.

    $gate->before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });

Nếu beforegọi lại trả về một kết quả không null kết quả sẽ trả về từ việc kiểm tra. 

Bạn có thể sử dụng các afterphương pháp để xác định một callback được thực hiện sau mỗi lần kiểm tra ủy quyền.

Tuy nhiên, bạn không thể thay đổi kết quả của việc kiểm tra quyền từ một aftercuộc gọi lại:

    $gate->after(function ($user, $ability, $result, $arguments) {
        //
    });

<a name="CheckingAbilities"></a>

### Kiểm tra ủy quyền
<a name="ckeckgatefacade"></a>
[Sử dụng Gate Facade](#)

Khi một quyền hạn đã được định nghĩa, chúng ta có thể kiểm tra quyền với user đó bằng nhiều cách khác nhau. Đầu tiên chúng ta có thể sử dụng một vài phương pháp  : `check`, `allows`, or `denies` phương pháp có trong Gate. 

Tất cả các phương pháp trên cần nhận được name quyền và các đối số có trong function định nghĩa quyền mà bạn đã định nghĩa.

Bạn không cần phải tìm kiếm người dùng hiện tại và truyền vào quyền đóvì mặc định nó đã được truyền vào làm đối số thứ một để bạn có thể truy xuất dữ liệu từ user và sử dụng.

Chúng ta cần một ví dụ sử dụng cho các phương pháp :   `check`, `allows`, or `denies` của Gate

        <?php
        
        namespace App\Http\Controllers;
        
        use Gate;
        use App\User;
        use App\Post;
        use App\Http\Controllers\Controller;
        
        class PostController extends Controller
        {
            /**
             * Update the given post.
             *
             * @param  int  $id
             * @return Response
             */
            public function update($id)
            {
                $post = Post::findOrFail($id);
        
                if (Gate::denies('update-post', $post)) {
                    abort(403);
                }
        
                // Update Post...
            }
        }
        ?>

Giải thích  Các phương pháp kiểm tra  `check`, `allows`, or `denies` của Gate 

`allows` phương pháp đơn giản là nghịch đảo của các deniesphương pháp và trả về true nếu  method action được ủy quyền.

Các `check` phương pháp là một bí danh của allowsphương pháp.


[Kiểm tra quyền với một người sử dụng chưa đăng nhập](#)


Nếu bạn muốn sử dụng Gate mặt tiền để kiểm tra xem một người dùng khác với người dùng đã xác thực. việc kiểm tra người sử dụng
chưa xác định có những quyền hạn đó hay không, bạn có thể sử dụng các `forUser` phương pháp:


            if (Gate::forUser($user)->allows('update-post', $post)) {
            //
            }       

[Đi qua nhiều đối số để kiểm tra quyền](#)


    Gate::define('delete-comment', function ($user, $post, $comment) {
        //
    });
    

Nếu khả năng của bạn cần nhiều tranh luận, chỉ cần vượt qua một mảng các đối số cho các Gatephương pháp

        if (Gate::allows('delete-comment', [$post, $comment])) {
            //
        }

<a name="usergate"></a>
[Sử dụng User Model](#)


Noài cách sử dụng Face Gate bạn cũng có thể sử dụng model User. Theo mặc định laravel model `App\User` sử dụng 
`Authorizable trait` cung cấp 2 phương pháp `can` and `cannot` các method này giống như `allows` and `denies` của mặt tiền `Gate`. 

Bạn có thể sử dụng nó như ví dụ :

            <?php
            
            namespace App\Http\Controllers;
            
            use App\Post;
            use Illuminate\Http\Request;
            use App\Http\Controllers\Controller;
            
            class PostController extends Controller
            {
                /**
                 * Update the given post.
                 *
                 * @param  \Illuminate\Http\Request  $request
                 * @param  int  $id
                 * @return Response
                 */
                public function update(Request $request, $id)
                {
                    $post = Post::findOrFail($id);
            
                    if ($request->user()->cannot('update-post', $post)) {
                        abort(403);
                    }
            
                    // Update Post...
                }
            }
Of course, the can method is simply the inverse of the cannot method:

         if ($request->user()->can('update-post', $post)) {
            // Update Post...
        }

<a name="gatebalde"></a>
[Sử dụng trong Blade Templates](#)

Để thuận tiện laravel cung cấp một số method trong `Teamplate Blade` là `@can` để kiểm tra người dùng có quyền sử dụng 

dữ liệu trong đó hay không. 

    <a href="/post/{{ $post->id }}">View Post</a>
    
    @can('update-post', $post)
        <a href="/post/{{ $post->id }}/edit">Edit Post</a>
    @endcan

Bạn cũng có thể sử dụng  `@can` và  `@else` :

        @can('update-post', $post)
            <!-- The Current User Can Update The Post -->
        @else
            <!-- The Current User Can't Update The Post -->
        @endcan
<a name="formrequest"></a>
`Sử dụng trong Form Requests`

Bạn cũng cso thể sử dụng định nghĩa Gate quyền trong một `request's authorize` method. ví dụ :

/**
 * Xác định nếu người dùng được ủy quyền để thực hiện yêu cầu này.
 *
 * @return bool
 */
public function authorize()
{
    $postId = $this->route('post');

    return Gate::allows('update', Post::findOrFail($postId));
}

<a name="Policies"></a>
### Policies

Ngoài cách viết các abilities vào trong provider mặc định là AuthServiceProvider, chúng ta có thể viết các abilities vào các Policy tương ứng để dễ quản lý hơn


<a name="create-prolicies"></a>
[Creating Policies](#)

Kể từ khi các lý luận quyền hạn được đặt trong `AuthServiceProvider` nó có thể quá tải và trở lên cồng kềnh. Laravel cho phép bạn chia nhỏ nó ra bằng các class `Policies`.

Trước tiên hãy tạo một `PostPolicy` mà sẽ chứa các định nghĩa quyền hạn về post.


Bạn có thể tạo ra nó bằng cách thủ công lệnh `artisan command` và nó được đặt trong thư mục app/Policies.

        php artisan make:policy PostPolicy

<a name="register-prolicies"></a>
[Đăng ký Policies - Registering Policies](#)


Một khi lý luận quyền hạn đã tồn tại, chúng tôi cần phải đăng ký với Gate class.

Các AuthServiceProviderchứa một `policies` tài sản mà bản đồ mà chỉ đến nơi định nghĩa các class chính sách quản lý quyền hạn.

Để đăng ký chính sách mới chỉ cần thêm PostPolicy class cho  protected $policies :

        <?php
        
        namespace App\Providers;
        
        use App\Post;
        use App\Policies\PostPolicy;
        use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
        
        class AuthServiceProvider extends ServiceProvider
        {
            /**
             * The policy mappings for the application.
             *
             * @var array
             */
            protected $policies = [
                Post::class => PostPolicy::class,
            ];
        
            /**
             * Register any application authentication / authorization services.
             *
             * @param  \Illuminate\Contracts\Auth\Access\Gate  $gate
             * @return void
             */
            public function boot(GateContract $gate)
            {
                $this->registerPolicies($gate);
            }
        }


<a name="writing-plicies"></a>

[Viết Policies](#)


Một khi policies đã được đăng ký chúng ta có thể thêm các phương pháp ủy quyền của nó.

Trong ví dụ này chúng ta định nghĩa một `User` có quyền  "update" một Post hay không là một method của PostPolicy.

        <?php
        namespace App\Policies;
        
        use App\User;
        use App\Post;
        
        class PostPolicy
        {
            /**
             * Determine if the given post can be updated by the user.
             *
             * @param  \App\User  $user
             * @param  \App\Post  $post
             * @return bool
             */
            public function update(User $user, Post $post)
            {
                return $user->id === $post->user_id;
            }
        }

Bạn có thể tiếp tục để xác định phương pháp bổ sung về chính sách khi cần thiết cho những quyền hạn khác nhau.

Ví dụ show, destroyhoặc addComment để ủy quyền tương ứng với  `Post actions`.


`Lưu ý : Tất cả policies được giải quyết qua Service Container. Có nghĩa bạn có thể tiêm bất kỳ phụ thuộc nào cần thiết vào trong Policies của bạn. Mà nó sẽ tự động được giải tiêm. ` 
 
<a name="check-prolicies"></a>
[Chặn tất cả kiểm tra quyền ](#)

Trong một số trường hợp chúng ta có thể chặn việc kiểm tra trong `Gate` bằng hàm `before`, Phương pháp này sẽ được chạy trước khi tất cả các kiểm tra ủy quyền khác:

        public function before($user, $ability)
        {
            if ($user->isSuperAdmin()) {
                return true;
            }
        }

`isSuperAdmin()` là  một ví dụ method có trong user bạn cần định nghĩa nó. 

        public function isSuperAdmin()
        {
            // tác vụ logic của bạn 
            return true; // or false
        }


Nếu các beforephương thức trả về một kết quả không null mà kết quả sẽ được coi là kết quả của việc kiểm tra..


[Kiểm Tra Policies](#)


Các phương pháp có trong Policies được gọi là chính xác theo cùng một cách khi bạn sử dụng  `Gate` Facades, User Model, @can trong Teamplate Blade để kiểm tra quyền đã được định nghĩa.


Sử dụng  Gate Facade

`Gate` tự động xác định `plicies` sử dụng các method trong nó bằng cách kiểm tra các class `object` đối số truyền vào.

demo :

        <?php
        
        namespace App\Http\Controllers;
        
        use Gate;
        use App\User;
        use App\Post;
        use App\Http\Controllers\Controller;
        
        class PostController extends Controller
        {
            /**
             * Update the given post.
             *
             * @param  int  $id
             * @return Response
             */
            public function update($id)
            {
                $post = Post::findOrFail($id);
        
                if (Gate::denies('update', $post)) {
                    abort(403);
                }
        
                // Update Post...
            }
        }
        
[sử dụng User Model](#)

Các Usermô hình của can và cannot phương pháp này cũng sẽ tự động sử dụng policies khi nào object đối số truyền vào.

ví dụ $post = Post::find(3);

truyền vào cho chính sách sử dụng sau đó nó lấy ra kiểm tra. 


if ($user->can('update', $post)) {
    // nếu đúng ủy quyền thông qua
}

if ($user->cannot('update', $post)) {
    // nếu sai
}

[Sử dụng Blade Templates](#)

Tương tự như vậy @can sẽ sử dụng các đối số object class truyền vào để kiểm tra quyền:

    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @endcan

[Sử dụng Policy Helper](#)

Các chức năng `policy global helper` có thể được sử dụng để lấy các `policy` class cho một trường hợp.

 Ví dụ, chúng ta có thể vượt qua một `Post` ví dụ để các `policy helper` để có được một thể hiện của chúng tôi tương ứng `PostPolicy class`

        if (policy($post)->update($user, $post)) {
            //
        }
<a name="controller-authorization"></a>
### Controller Authorization

Theo mặc định, Laravel App\Http\Controllers\Controller class controller cơ sở sử dụng AuthorizesRequests trait. Đặc điểm này cung cấp method `authorize` để sử dụng nhanh chóng xác định một quyền có được cho phép hay không nó tương tự như các phương pháp uỷ quyền khác như Gate::allows and $user->can().


vì vậy hãy sử dụng authorize cho phép request kiểm tra ủy quyền update post.

        <?php
        
        namespace App\Http\Controllers;
        
        use App\Post;
        use App\Http\Controllers\Controller;
        
        class PostController extends Controller
        {
            /**
             * Update the given post.
             *
             * @param  int  $id
             * @return Response
             */
            public function update($id)
            {
                $post = Post::findOrFail($id);
        
                $this->authorize('update', $post);
        
                // Update Post...
            }
        }

Nếu các hành động được ủy quyền, bộ điều khiển sẽ tiếp tục thực hiện bình thường.

Tuy nhiên, nếu các authorizephương pháp xác định rằng hành động là không được phép, một AuthorizationExceptionsẽ tự động được ném ra mà tạo ra một phản ứng HTTP thông báo với một `403 Not Authorized` mã trạng thái lỗi.


Các `AuthorizesRequests` đặc điểm cũng cung cấp các `authorizeForUser` phương pháp để cho phép một làm việc trên một người sử dụng mà không phải là đã đăng nhập xác thực :


$this->authorizeForUser($user, 'update', $post);

TỰ ĐỘNG XÁC ĐỊNH PHƯƠNG PHÁP Policy 

Thông thường, phương pháp của một chính sách sẽ tương ứng với các phương pháp trên một bộ điều khiển `CONTROLLER`.

phương pháp điều khiển và các phương pháp chính sách chia sẻ cùng tên: update.


Vì lý do này, Laravel cho phép bạn chỉ cần vượt qua các đối số ví dụ để các `authorize` phương pháp, 

Trong ví dụ này, kể từ khi authorizeđược gọi là từ `UPDATE CONTROLER` update CỦA PostPolicy CŨNG SẼ ĐƯỢC GỌI



/**
 * Update the given post.
 *
 * @param  int  $id
 * @return Response
 */
public function update($id)
{
    $post = Post::findOrFail($id);

    $this->authorize($post);

    // Update Post...
}
