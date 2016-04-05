#What is this?
This is full-featured replacement for Laravel's Redis and file cache storages. All storages support proper tagging. 
Cache pools provided by http://www.php-cache.com/. All classes in this lib only proxies between Laravel's cache
system and cache pools from http://www.php-cache.com/

##What is proper tagging?
For example you have:
    
    Cache::tags(['tag1', 'tag2'])->put('tag-test1', 'ok', 20);
    
How Laravel's native cache works with tags and Redis (Laravel 5.2):

    Cache::tags(['tag1', 'tag2'])->get('tag-test1');    //< 'ok'
    Cache::tags(['tag2', 'tag1'])->get('tag-test1');    //< null
    Cache::tags(['tag1'])->get('tag-test1');            //< null
    Cache::tags(['tag2'])->get('tag-test1');            //< null
    Cache::get('tag-test1');                            //< null
    Cache::forget('tag-test1');                         //< won't delete anything
    Cache::tags(['tag1', 'tag2'])->forget('tag-test1'); //< deleted
    Cache::tags(['tag2', 'tag1'])->forget('tag-test1'); //< won't delete anything
    Cache::tags(['tag1'])->forget('tag-test1');         //< won't delete anything
    Cache::tags(['tag2'])->forget('tag-test1');         //< won't delete anything
    Cache::tags(['tag1'])->flush();                     //< won't delete anything
    Cache::tags(['tag2'])->flush();                     //< won't delete anything
    Cache::tags(['tag1', 'tag2'])->flush();             //< flushed
    Cache::tags(['tag2', 'tag1'])->flush();             //< won't delete anything

If you think that this is correct behavior - go away, you don't need this lib.

How it works with this lib:

    Cache::tags(['tag1', 'tag2'])->get('tag-test1');    //< 'ok' - use Cache::get('tag-test1') instead
    Cache::tags(['tag2', 'tag1'])->get('tag-test1');    //< 'ok' - use Cache::get('tag-test1') instead
    Cache::tags(['tag1'])->get('tag-test1');            //< 'ok' - use Cache::get('tag-test1') instead
    Cache::tags(['tag2'])->get('tag-test1');            //< 'ok' - use Cache::get('tag-test1') instead
    Cache::get('tag-test1');                            //< 'ok'
    Cache::forget('tag-test1');                         //< deleted
    Cache::tags(['tag1', 'tag2'])->forget('tag-test1'); //< deleted - use Cache::forget('tag-test1') instead
    Cache::tags(['tag2', 'tag1'])->forget('tag-test1'); //< deleted - use Cache::forget('tag-test1') instead
    Cache::tags(['tag1'])->forget('tag-test1');         //< deleted - use Cache::forget('tag-test1') instead
    Cache::tags(['tag2'])->forget('tag-test1');         //< deleted - use Cache::forget('tag-test1') instead
    Cache::tags(['tag1'])->flush();                     //< deleted all cache entries with tag 'tag1'
    Cache::tags(['tag2'])->flush();                     //< deleted all cache entries with tag 'tag2'
    Cache::tags(['tag1', 'tag2'])->flush();             //< deleted all cache entries with tag 'tag1' or 'tag2'
    Cache::tags(['tag2', 'tag1'])->flush();             //< deleted all cache entries with tag 'tag2' or 'tag1'

##How to use it:

Add `\AlternativeLaravelCache\Provider\AlternativeCacheStoresServiceProvider::class` to `$providers` in your `config/app.php`: 

    $providers = [
        \AlternativeLaravelCache\Provider\AlternativeCacheStoresServiceProvider::class
    ]
    
##Notes
By default service provider will replace Laravel's `redis` and `file` cache stores. 
You can alter this behavior like this:

    class MyAlternativeCacheStoresServiceProvider extends AlternativeCacheStoresServiceProvider {
        static protected $redisDriverName = 'altredis';
        static protected $fileDriverName = 'altfile';
    }
    
File cache storage currently supports only `'driver' => 'file'`. You can extend list of file cache drivers by  
overwriting `AlternativeCacheStoresServiceProvider->makeFileCacheAdapter()`

Yep, there is no tests right now and possibly they will never appear. 