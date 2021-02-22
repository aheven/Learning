### Contentpage 组件

在后台线程中加载 ContentProviider 数据并进行分页。

#### 依赖声明

```groovy
dependencies {
    implementation "androidx.contentpager:contentpager:1.0.0"
}
```

#### 使用步骤

1. ##### 初始化ContentPager

   ```kotlin
   val contentPager = ContentPager(
       contentResolver,
       LoaderQueryRunner(this, LoaderManager.getInstance(this))
   )
   ```

2. ##### 构建查询条件

   ```kotlin
   val uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI
   val projection = arrayOf(
       MediaStore.Images.ImageColumns._ID
   )
   contentPager.query(
       uri,
       projection,
       bundleOf(
           ContentPager.QUERY_ARG_OFFSET to offset,
           ContentPager.QUERY_ARG_LIMIT to limit
       ),
       null,callback
   )
   ```

   在 callback 中处理回调数据。

3. ##### 重写 LoaderQueryRunner 以支持 androidx

   ```kotlin
   /**
    * A {@link ContentPager.QueryRunner} that executes queries using a {@link LoaderManager}.
    * Use this when preparing {@link ContentPager} to run in an Activity or Fragment scope.
    */
   public final class LoaderQueryRunner implements ContentPager.QueryRunner {
   
       private static final boolean DEBUG = false;
       private static final String TAG = "LoaderQueryRunner";
   
       @SuppressWarnings("WeakerAccess") /* synthetic access */
       final Context mContext;
       @SuppressWarnings("WeakerAccess") /* synthetic access */
       final LoaderManager mLoaderMgr;
   
       public LoaderQueryRunner(@NonNull Context context, @NonNull LoaderManager loaderMgr) {
           mContext = context;
           mLoaderMgr = loaderMgr;
       }
   
       @Override
       public void query(final @NonNull Query query, @NonNull final Callback callback) {
           if (DEBUG) Log.d(TAG, "Handling query: " + query);
   
           LoaderManager.LoaderCallbacks<Cursor> callbacks = new LoaderManager.LoaderCallbacks<Cursor>() {
               @NonNull
               @Override
               public Loader<Cursor> onCreateLoader(int id, @Nullable Bundle args) {
                   if (DEBUG) Log.i(TAG, "Loading results for query: " + query);
                   if (id != query.getId()) throw new RuntimeException("Id doesn't match query id.");
   
                   return new CursorLoader(mContext) {
                       @Override
                       public Cursor loadInBackground() {
                           return callback.runQueryInBackground(query);
                       }
                   };
               }
   
               @Override
               public void onLoadFinished(@NonNull Loader<Cursor> loader, Cursor cursor) {
                   if (DEBUG) Log.i(TAG, "Finished loading: " + query);
                   mLoaderMgr.destroyLoader(query.getId());
                   callback.onQueryFinished(query, cursor);
               }
   
               @Override
               public void onLoaderReset(@NonNull Loader<Cursor> loader) {
                   if (DEBUG) Log.w(TAG, "Ignoring loader reset for query: " + query);
               }
   
           };
   
           mLoaderMgr.restartLoader(query.getId(), null, callbacks);
       }
   
       @Override
       public boolean isRunning(@NonNull Query query) {
           Loader<Cursor> loader = mLoaderMgr.getLoader(query.getId());
           return loader != null && loader.isStarted();
           // Hmm, when exactly would the loader not be started? Does it imply that it will
           // be starting at some point?
       }
   
       @Override
       public void cancel(@NonNull Query query) {
           mLoaderMgr.destroyLoader(query.getId());
       }
   }
   
   ```