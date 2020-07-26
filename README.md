# Handy cheat sheet

### 1. Defining a password mutator

Define a mutator in your model (most likely app\User.php) which would encrypt all your password fields:

    public function setPasswordAttribute($password)
    {   
        $this->attributes['password'] = bcrypt($password);
    }

Now you don't need to write the bcrypt function when dealing with the password field in subsequent controllers.

Found at [Scotch.io tutorial on user authorization](https://scotch.io/tutorials/user-authorization-in-laravel-54-with-spatie-laravel-permission)


### 2 . Appending accessors

Sometimes, particularly when working with APIs or on other specific instances, you may need to append particular accessors (or attributes) to the default collection returned by Eloquent when retrieving records from the database. 

Ex.

Create a full name accessor:

    class User extends Model
    {
        public function getFullNameAttribute()
        {
            return $this->first_name . ' ' . $this->last_name;
        }
    }
    
Convention = get + [name of your attribute] + Attribute

To access an accessor:

    {{ $user->fullName }}
    
To consume it in an API for example, you need the $appends property on your model to include in the default collection return by Eloquent:

    class User extends Model
    {
        protected $appends = ['fullName'];
    }

Found at [Terry Harvey - 5 Laravel Eloquent Tips & Tricks](https://terryharvey.co.uk/5-laravel-eloquent-tips-tricks/)


### 2. Booting a trait

Replicate the boot method of the class being inject into. 

Ex:

    protected static function bootRecordsActivity()
    {
        if (auth()->guest()) return;

        foreach (static::getActivitiesToRecord() as $event) {
            static::$event(function ($model) use ($event) {
                $model->recordActivity($event);
            });
        }

        static::deleting(function($model) {
            $model->activity()->delete();
        });
    }
    
Convention = boot + [name of your trait]

* Note: you should declare it as static method

Found at [Laracast.com - Let's Build A Forum with Laravel and TDD](https://laracasts.com/series/lets-build-a-forum-with-laravel/episodes/25)


### 3. Creating dropdowns using Eloquent lists method

Suppose you want to show a categories dropdown whose information is pulled from an Eloquent model

    $categories = Category::lists('title', 'id');
    
Then somewhere in your view:

    {{ Form::select('category', $categories) }}
 
Found at [laravel-tricks.com - Easy dropdowns with Eloquent's Lists method](http://laravel-tricks.com/tricks/easy-dropdowns-with-eloquents-lists-method)


### 4. Creating/modifying request input value before custom request validation

If you want to create/modify request input before it gets validated, you can use getValidatorInstance().

    // app/Http/Requests/Admin/PostRequest.php
    
    public function getValidatorInstance()
    {
        // Do something here

        return parent::getValidatorInstance();
    }

Then, inside the method, you can execute request merging like so:

    // app/Http/Requests/Admin/PostRequest.php
    
    public function getValidatorInstance()
    {
        $this->getPermalink();

        return parent::getValidatorInstance();
    }

    
    /**
     * Get the permalink using title
     */
    public function getPermalink()
    {
        $this->merge(['permalink' => str_slug(request()->input('title'))]);
    }
    
Found at [Laracasts discussions](https://laracasts.com/discuss/channels/requests/modify-request-input-value-before-validation) while coding my own needs.


### 5. Dynamic search trait

If you want your search functionality to be dynamic as possible, I just did implement it like this:

    
    // app/Models/Shared/CanSearch.php - This is a trait that can be used by any models
    
    public function scopeSearch($query, $keyword, $columns = [], $relativeTables = [])
    {
        if (empty($columns)) {
            $columns = array_except(
                Schema::getColumnListing($this->table), $this->guarded
            );
        }   

        $query->where(function ($query) use ($keyword, $columns) {
            foreach ($columns as $key => $column) {
                $clause = $key == 0 ? 'where' : 'orWhere';
                $query->$clause($column, "LIKE", "%$keyword%");
                    
                if (!empty($relativeTables)) {
                    $this->filterByRelationship($query, $keyword, $relativeTables);
                }
            }
        });

        return $query;
    }
    
    
    private function filterByRelationship($query, $keyword, $relativeTables)
    {
        foreach ($relativeTables as $relationship => $relativeColumns) {
            $query->orWhereHas($relationship, function($relationQuery) use ($keyword, $relativeColumns) {
                foreach ($relativeColumns as $key => $column) {
                    $clause = $key == 0 ? 'where' : 'orWhere';
                    $relationQuery->$clause($column, "LIKE", "%$keyword%");
                }
            });
        }

        return $query;
    }

Then, you can use it like so:
    
    $keyword = $request->keyword;
    $users = User::search($keyword)->latest()->paginate(10);

The scope "search" can optionally accept a 2nd parameter ($columns) to specify specific columns to search on the given model or accept a 3rd parameter ($relativeTables) to search for specific columns on related models.


### 6. Don't hard code primary keys

Instead of hardcoding your primary keys/values like this:

    $post->id // ==  1
    // OR
    $post->post_id // I used to have my primary key like this to practice defining Eloquent relationships

You can get the primary key value or primary key name like this:

    $post->getKey() // == 1
    $post->getKeyName() // returns the name of your primary key


### 7. BelongsTo default relationship values

Let’s say you have Product belonging to a Category:
    
    {{ $product->category->name }}

But what if there is no category, or isn’t set for some reason, You will probably get an error prompting “trying to get the property of non-object”.

I usually prevent it like this:

    {{ $post->author->name ?? '' }}
    
PHP7's "??" is a null coalescing operator that has been added as syntactic shortcut for the common case of needing to use a ternary in conjunction with isset().

But I did not know for a long time that fortunately, we can do it on Eloquent relationship level:

    public function category()
    {
        return $this->belongsTo(Category::class)->withDefault();
    }
    
The category() relation will return an empty App\Category model if no category is attached to the product.

Furthermore, we can assign default property values to that default model:

    public function category()
    {
        return $this->belongsTo(Category::class)->withDefault([
            'name' => 'Uncategorized'
        ]);
    }

Found at [Laravel News](https://laravel-news.com/eloquent-tips-tricks) while searching for a useful plugin or code optimization tips.


### 8. Retrieving random rows

The title just nailed it:

    $products = Product::orderByRaw('RAND()')->take(5)->get();

Found at [cjthomp/50 Laravel Tricks #9](https://gist.github.com/cjthomp/1455c39d4a14292676ea#9-retrieve-random-rows)

### 9. Getting the first or last array element

Oh, another one from CJ Thompson, really useful for me:

    // hide all but the first item
    @foreach ($menu as $item)
        <div @if ($item != reset($menu)) class="hidden" @endif>
            <h2>{{ $item->title }}</h2>
        </div>
    @endforeach

    // apply CSS to last item only
    @foreach ($menu as $item)
        <div @if ($item == end($menu)) class="no_margin" @endif>
            <h2>{{ $item->title }}</h2>
        </div>
    @endforeach
    
Eloquently, you can access the $loop variable which has 'first' and 'last' attributes:

    // hide the first item
    @foreach ($menu as $item)
        <div @if ($loop->first) class="hidden" @endif>
            <h2>{{ $item->title }}</h2>
        </div>
    @endforeach

    // apply CSS to last item only
    @foreach ($menu as $item)
        <div @if ($loop->last) class="no_margin" @endif>
            <h2>{{ $item->title }}</h2>
        </div>
    @endforeach

Found at [cjthomp/50 Laravel Tricks #18](https://gist.github.com/cjthomp/1455c39d4a14292676ea#18-firstlast-array-element)


### 10. Syncing belongsToMany relationship with extra pivot table attribute

As per the documentation, syncing the intermediary table is as easy as passing IDs:
    
    $roleIDs = [1, 2, 3];
    $user->roles()->sync($roleIDs);
    
But if you have an additional attribute in your belongsToMany relationship, you need to format your data like this:

    $user->roles()->sync([
        1 => ['expires_at' => '2042-10-04 04:08:12'], 
        2 => ['expires_at' => '2042-10-04 04:08:12'], 
        3 => ['expires_at' => '2042-10-04 04:08:12']
    ]);
    
In this example, the additional attribute we are pertaining to is the "expires_at" column.

I simply do it like this:

    $roleIDs = [1, 2, 3];
    $roleExpirations = array_fill(0, count($roleIDs), ['expires_at' => '2042-10-04 04:08:12']);
    $rolesToSync = array_combine($roleIDs, $roleExpirations);
    $user->roles()->sync($rolesToSync);

**array_fill** fill an array with values

**array_combine** creates an array by using one array for keys and another for its values

Basically, we use **$roleIDs** to be the keys of the array and **$roleExpirations** to be the values of the array that we passed in the sync method to achieve our desired format.

But what if, you have different role expirations.
Just loop through it and combine just like the previous example:

    $roleIDs = [1, 2, 3];
    $expirationDates = ['2042-10-04 04:08:12', 2042-10-05 04:08:12, 2042-10-06 04:08:12];
    $roleExpirations = [];
    foreach ($expirationDates as $expiration) {
        $roleExpirations[] = ['expires_at' => $expiration];
    }
    $rolesToSync = array_combine($roleIDs, $roleExpirations);
    $user->roles()->sync($rolesToSync);

### 11. Pivot table with timestamps

If you want your pivot table to have automatically maintained created_at and updated_at timestamps, use the withTimestamps method on the relationship definition:

    return $this->belongsToMany('App\Role')->withTimestamps();
    
Found at [Laracasts How Do I > Understanding CSRF](https://laravel.com/docs/5.7/eloquent-relationships#updating-many-to-many-relationships)

### 12. Model Factories > Forget creating a relationship within a closure

Instead of the common & traditional passing of closure to create a relationship in a factory like this:

    $factory->define(App\Task::class, function (Faker $faker) {
        return [
            'body' => $faker->sentence,
            'project_id' => function() {
                return factory(\App\Project::class)->create()->id
            }
        ];
    });
    
You can just pass a factory instance like so:

    $factory->define(App\Task::class, function (Faker $faker) {
        return [
            'body' => $faker->sentence,
            'project_id' => factory(\App\Project::class)
        ];
    });
    
 Behind the scenes, it will call create() method, it would fetch the id, and it will assign that in the 'project_id' column. Pretty cool isn't it?
 
 Found at [Laracasts Build A Laravel App With TDD > Task UI Updates: Part 2](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/14)
 
 
 ### 13. Touching relationship

If find yourself needing to update the relationship's timestamp when updating a record, you may want to use $touches in the model class:


    class Task extends Model
    {
        protected $guarded = [];

        protected $touches = ['project'];

        public function project()
        {
            return $this->belongsTo(Project::class);
        }
    }

This will update the Project model's update_at column accordingly.

Found at [Laracasts Build A Laravel App With TDD > Touch It](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/15)


 ### 14. Create array using variable name

From this code:

    public function recordActivity($type)
    {
        $this->activity()->create([
            'description' => $type
        ]);
    } 

We can use PHP's compact() function to create array using variable names and will output same functionality like the above code:

    public function recordActivity($description)
    {
        $this->activity()->create(compact('description'));
    }

Found at [Laracasts Build A Laravel App With TDD > Project Activity Feeds: Part 3](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/22)


 ### 15. Shortcut for Polymorphic table

When creating Polymorphic relations you will most likely create a column for id & type like this:
    
    $table->integer('subject_id')->unsigned();
    $table->string('subject_type');

But you can use 'morphs' instead of the above code to explicity express that you are creating a column for Polymorphic relationship:

    $table->morphs('subject');

Found at [Laracasts Build A Laravel App With TDD > The Subject of the Activity](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/25)


### 16. Named error bags

If you need to display specific errors using the $errors variable that Laravel provides, you can do it like so:

First, define a specific $errorBag in your form request:
    
    class ProjectInvitationRequest extends FormRequest
    {
        protected $errorBag = 'invitations';
        
        ...
    }
    
Next, use the 'invitations' error bag to loop through the possible errors:

    @if ($errors->invitations->any())
        <div class="field mt-6">
            @foreach ($errors->invitations->all() as $error)
                <li class="text-sm text-red">{{ $error }}</li>
            @endforeach
        </div>
    @endif

So when you hit the submit button on a form, it will only include the errors specific to your named error bag

Found at [Laracasts Build A Laravel App With TDD > Validation Errors For Multiple Forms](https://laracasts.com/series/build-a-laravel-app-with-tdd/episodes/34)


### 17. Orphan record

If you want a record with a relationship contraint to not be deleted when its foreign key reference was deleted you can use onDelete('set null'):

    $table->foreign('user_id')
        ->references('id')
        ->on('users')
        ->onDelete('set null');

It will allow you to delete the parent record (in our example it is user_id in the users table) and the dependent record will just set the user_id to NULL.

Also, if you want to have a default value when the parent record was deleted you can use withDefault() when defining eloquent relationships:

    public function users()
    {
        return $this->belongsTo(User::class, 'user_id')->withDefault([
            'name' => 'Guest User',
        ]);
    }


Found at [Laracasts Laravel 6 From Scratch > Understanding Foreign Keys and Database Factories > comments section](https://laracasts.com/series/laravel-6-from-scratch/episodes/30)


### 18. Replicate: make a copy of a row

The most convenient way in Laravel to make a copy of database entry:

    $post = Post::find(1);
    $postReplica = $post->replicate();
    $postReplica->save();

Found at [Laravel News > 20 Laravel Eloquent Tips & Tricks > #14](https://laravel-news.com/eloquent-tips-tricks)


### 19. Constrained() method for migrations

Laravel provides additional, terser methods that use convention to provide a better developer experience. 

The example above could be written like so:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained();
    });
    
The foreignId method is an alias for unsignedBigInteger while the constrained method will use convention to determine the table and column name being referenced. If your table name does not match the convention, you may specify the table name by passing it as an argument to the constrained method:

    Schema::table('posts', function (Blueprint $table) {
        $table->foreignId('user_id')->constrained('users_table');
    });
    
You may also specify the desired action for the "on delete" and "on update" properties of the constraint:

    $table->foreignId('user_id')
      ->constrained()
      ->onDelete('cascade');    
      
Any additional column modifiers must be called before constrained:
    $table->foreignId('user_id')
      ->nullable()
      ->constrained();
      
Found at [Laravel Documentation > Foreign Key Constraints](https://laravel.com/docs/7.x/migrations#foreign-key-constraints)
