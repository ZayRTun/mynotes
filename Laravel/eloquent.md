**Home**
- [Home](../index.md)
---
# Eloquent

## Eloquent Relationships
```php
class User {

    public function articles()
    {
        return $this->hasMany(Articles::class); // select * from articles where user_id =
    }

    public function projects()
    {
        return $this->hasMany(Projects::class);
    }
    // you need to have a user_id column in projects database so as in Articles database for this to work
}

class Projects {

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}

// hasOne
// hasMany
// belongsTo
// belongsToMany
// there are many more but above 4 is good enough for many cases
```

## Tinker
**using factory to seed table with fake datas**
```php
>>> factory(App\User::class)->create();
>>> factory(App\User::class, 5)->create(); // to create 5 users

>>> factory(App\Articles::class, 5)->create(['title' => 'Override the title']); // overriding the title
>>> factory(App\Articles::class, 5)->create(['user_id' => 1]); // overriding the title
```
**Create a factory**
```bash
php artisan make:factory AricleFactory -m "App\Article"
```
**Defining a factory**
```php
$factory->define(Article::class, function (Faker $faker) {
    return [
        'user_id' => factory(\App\User::class), // this will create a user get the id
        'title' => $faker->sentence,
        'excerpt' => $faker->sentence,
        'body' => $faker->paragraph,
    ]
});


// Foreign key constraints for not leaving any orphaned data when the user is deleted

public function up()
{
    Schema::create('articles', function(Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedBigInteger('user_id');
        $table->string('title');
        $table->text('excerpt');
        $table->text('body');

        // When a user is deleted the associated articles will also be deleted
        $table->foreign('user_id)
            ->references('id)
            ->on('users)
            ->onDelete('cascade);

    })
}
// to make an eloquent query you call the method as a property

$user = new App\User::find(1);
$user->articles;
```

## Eloquent Many to Many relationship with Linking Tables

**Cases**

// an article has many tags
// a tag belongs to many articles

```php
class Articles {

    public function tags()
    {
        return $this->belongsToMany(Tags::class);
    }
}

// creating tags table
public function up()
{
    Schema::create('articles', function(Blueprint $table) {
        $table->bigIncrements('id');
        $table->string('name');
        $table->timestamps();
    });



        // for belongsToMany() to work we need 3 tables
        // a Tags table, an Articles table and a LINK/PIVOT TABLES LIKE BELOW
        // if you want, can create a separate migration for below

        // Convention for pivot tables is to take the name of two tables an join them alphabetically

    Schema::create('article_tag', function(Blueprint $table) {
        $table->bigIncrements('id');
        $table->unsignedBigInteger('article_id');
        $table->unsignedBigInteger('tag_id');
        $table->timestamps();

        // set up a unique key, so no duplicates
        $table->unique(['article_id', 'tag_id']);
    });

        // When an article is deleted the associated articles will also be deleted
        $table->foreign('article_id')->references('id')->on('articles')->onDelete('cascade');
        $table->foreign('tag_id')->references('id')->on('tags')->onDelete('cascade');

    })
}
```

### And now the inverse
```php
class Tag extends Model
{
    public function articles()
    {
        return $this->belongsToMany(Articles::class);
    }
}


// and now we can query all articals that belongs to a perticular tag
>>> $tag = App\Tag::first();
>>> $tag->articles; // will return all articles associated with the tag
```

## To display the tags::

```php
<div id="content">
    <div class="title">
        <h2>{{ $article->title}}</h2>
    <p><img src="/images/banner.jpg" alt="" class="image image-full" /> </p>
    <p>{{ $article->body }}</p>
    <p style="margin-top: 2em;">

        // This is how to display
        @foreach ($article->tags as $tag)
            <a href="{{ route('articles.index', ['tag' => $tag->name]) }}">{{ $tag->name }}</a>
            // you can use these two options
            <a href="/articles?tag={{ $tag->name }}">{{ $tag->name }}</a>

        @endforeach
    </p>
</div>
```
**In case the tag clicked don't have a associated article, we need to display a relavant message as follows**  
*By using a `@forelse` block*
```php
@forelse($articles as $article)
    <div id="content">
        <div class="title">
            <a href="/articles/{{ $article->id }}">
                <h2>{{ $article->title  }}</h2>
            </a>
        </div>

        <p><img src="images/banner.jpg" alt="" class="image image-full" /> </p>
        <p>{{ $article->excerpt }}</p>
    </div>
@empty
    <p>No relevant articles yet.</p>
@endforelse
```
## To fetch the articles in ArticleController:

```php
class ArticleController extends Controller
{
    public function index()
    {
        if (request('tag')) {
            $articles = Tag::where('name', request('tag'))->firstOrFail()->articles;
        } else {
            $articles = Article::latest()->get();
        }

        return view('articles.index', compact('articles'));
    }
}
```

## To *Attach or Detach* a TAG while creating an article in Store method:
*we can use the `attach()` method*
```php
// attach a single tag with it's id:
$article->tags()->attach(1);

// attach multiple tags with related ids:
$article->tags()->attach([1, 2]);

// detach a single tag with it's ids:
$article->tags()->detach(1);

// detach multiple tags with related ids:
$article->tags()->detach([1, 2]);

// you can also attach tags using a Model:
$tag = App\Tag::find(1);
$article->tags()->attach($tag);

// one more way using findMany
$tags = App\Tag::findMany([1, 2]);

$article->tags()->attach($tags);
```  
**Validating tags**
```php
return request()->validate([
    'title' => 'required',
    'excerpt' => 'required',
    'body' => 'required',
    // check if the tags.id exists in the tags table id column
    'tags' => 'exists:tags,id'
]);
```
### One more tweak to *`store`* method in controller:
**Because tags is not part of the Article table/model, that is only an attachment.**
```php
public function store()
{
    // validate separately
    $this->validateArticles();
    
    // and assign only the related value to Article and create:
    $article = new Article(request(['title', 'excerpt', 'body']));
    $article->user_id = 3;
    $article->save();

    if (request()->has('tags')) {
        $article->tags()->attach(request('tags'));
    }

    return redirect(route('articles.index'));
}
```
**To insert timestamps in `article_tag` table while attaching tags:**
```php
public function tags()
{
    return $this->belongsToMany(Tag::class)->withTimestamps();
}
```



