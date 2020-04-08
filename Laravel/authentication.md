**Home**
- [Home](../index.md)
---

# Laravel Authentication

**To acquire the frontend ui**  
`composer require laravel/ui --dev`  

**Front end Auth scaffolding with Vue**  
`php artisan ui vue --auth`

**The following is whats cause the auth**
```php
// look at the class constructor:
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
}

// or do the below also works
// attaching the middleware with auth
Route::get('/home', 'HomeController@index')->name('home')->middleware('auth');
```  
**You can access the authenticated user info:**  

`{{ Auth::user()->name }}`  

// or below they both are identicle  

`{{ auth()->user()->name }}`

## Check if Auth in not Null
```php
Auth::check(); // to check

<div class="title m-b-md">
    @if (Auth::check())
    // or below, both workds
    @if (auth()->check())
        {{ auth()->user()->name }}
    @else
        Laravel    
    @endif
</div>

// And even better, you can simply do this:
@auth
    Hi, {{ auth()->user()->name }}
@else
    Laravel    
@endauth

// And for the opposite:
@guest
    Please, sign in!
@else
    Hi, {{ Auth::user()->name }}
@endguest
```
#
## The password reset flow:
1. Click "Forgot Password"
2. Fill out a form with their email address.
3. Prepare a unique token and associate it with the user's account.
4. Send an email with a unique link back to our site that confirms email ownership.
5. Link back to website, confirm the token, and set a new password.

