# FilamentPHP-Likes-Post-Blog
The source code for the [Filament](https://filamentphp.com) website.

## Docs

> [!NOTE]
> Please submit pull requests for documentation changes to the [`filament/filament`](https://github.com/filamentphp/filament) repository. The relevant documentation files can be found in the `/docs` directory of each package.


Add like button for blog post
1. Create migration
```
php artisan make:migration post_like
```

```
<?php

//use App\Models\Post;
use Firefly\FilamentBlog\Models\Post;
use App\Models\User;
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('post_like', function (Blueprint $table) {
            $table->id();
            $table->foreignIdFor(User::class)->index();
            $table->foreignIdFor(Post::class)->index();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::dropIfExists('post_like');
    }
};

```
2. Create livewire component
```
php artisan make:livewire LikeButton
```
```
<?php

namespace App\Livewire;

use Livewire\Component;
use Firefly\FilamentBlog\Models\Post;

class LikeButton extends Component
{
    public Post $post;


    public function toggleLike()
    {
        if (auth()->guest())
        {
            return $this->redirect(route('login'), true);
        }

        $user = auth()->user();



        if ($user->hasLiked($this->post))
        {
            $user->likes()->detach($this->post);
            return;
        }

        $user->likes()->attach($this->post);

    }

    public function render()
    {
        return view('livewire.like-button');
    }
}
```
3. In resources/views/livewire/ ---- Create like-button.blade.php & add this code
```
<button wire.loading.attr="disabled" wire:click="toggleLike()" class="flex items-center">



    <svg wire:loading.delay aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin fill-red-600"
        viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path
            d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
            fill="currentColor" />
        <path
            d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
            fill="currentFill" />
    </svg>


    <svg wire:loading.delay.remove xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
        stroke-width="1.5" stroke="currentColor"
        class="w-6 h-6 {{ Auth::user()?->hasLiked($post) ? 'text-red-600' : 'text-gray-600' }}">
        <path stroke-linecap="round" stroke-linejoin="round"
            d="M21 8.25c0-2.485-2.099-4.5-4.688-4.5-1.935 0-3.597 1.126-4.312 2.733-.715-1.607-2.377-2.733-4.313-2.733C5.1 3.75 3 5.765 3 8.25c0 7.22 9 12 9 12s9-4.78 9-12z" />
    </svg>

    <span class="ml-2 text-gray-600">
        {{ $post->likes()->count() }}
    </span>
</button>
 
```

4. In App\Models\User add this code
```
    public function likes()
    {
        return $this->belongsToMany(Post::class, 'post_like')->withTimestamps();
    }

    public function hasLiked(Post $post)
    {
        return $this->likes()->where('post_id', $post->id)->exists();
    }
```

5. In App\Models\Post or Firefly\FilamentBlog\Models\Post add this code
```
    public function likes()
    {
        return $this->belongsToMany(User::class, 'post_like')->withTimestamps();
    }

    public function scopePopular($query)
    {
        $query->withCount('likes')
        ->orderBy("likes_count", 'desc');
    }

    public function scopeSearch($query, $search = '')
    {
        $query->where('title', 'like', "%{$search}%");
    }
 
```
6. In FireFlyBlog vendor/firefly/src/resources/PostResource or another PostResource add this code
```
                            TextEntry::make('likes.id')
                                ->label('Likes')
                                ->formatStateUsing(fn ($record) => $record->likes->count())
                                ->color('danger')
                                ->icon('heroicon-o-heart')
                                ->iconColor('danger'),

                            TextEntry::make('likes.name') //Show user name of the like post
                                 ->label('Likes Users')
                                 ->color('danger')
                                 ->badge()
                                 ->icon('heroicon-o-heart')
                                 ->iconColor('danger'),
 
```

7. In vendor/filament-blog/blogs/show.blade.php or another blog part add this line
```
Like: <livewire:like-button :key="'like-' . $post->id" :$post />
 
```
