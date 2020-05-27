+++
date = 2019-04-07T18:30:00Z
excerpt = "Android Library to implement Paging support for Realtime Database in RecyclerView. This library is developed by me. It will help you to implement database in pages."
hero = "/images/firebasepagination.png"
timeToRead = 4
title = "Firebase Database Pagination — Android 🔥"

+++
Hi everyone, In this article, we will learn to implement _Paging support_ for _Firebase Database_ in Android. Before starting to the topic, Let’s first take a look at the available components within the Firebase.

**FirebaseUI-Android** library has **FirebaseRecyclerAdapter** for easy implementation of the population of **Firebase Realtime Database**. But if the database is having a total number of children in thousands or around then it becomes a bad presentation of User Interface. Let’s take an example if you are implementing social media app and you are having around 100 Posts. If we load these Posts using _FirebaseRecyclerAdapter_ then it will load all the Posts at the time of loading. So, this will be wastage of memory or hectic for the user to scroll down with a large list or it is not good to present in front of the application user. To overcome this, we will use pagination which will load Firebase Database items in pages.

Recently, Firebase has released the **Firestore Database.** To support pagination with Firestore database **FirebaseUI-Android** library has provided the **API** for that purpose as **FirestorePagingAdapter** but still, it’s not available for **Realtime Database.** Firebase Realtime Database is also important in some type of applications such as chatting apps. Because its speed is faster than Firestore. So, there is a need for pagination in Firebase Database too.

For this purpose, I have developed **API** to implement Firebase Realtime Database Pagination in _RecyclerView_. This will help you to populate your Firebase Database items in _RecyclerView_ with paging support. FirebaseUI-Android will soon publish this API officially. It's expected to be released in **_version-4.4.0_** officially. Till it will happen, let’s have a look…

This API is available on [this](https://github.com/PatilShreyas/FirebaseRecyclerPagination) GitHub repository.

_FirebaseRecyclerPagingation_ Library binds Firebase Realtime Database Query to a `RecyclerView` by loading Data in pages. `FirebaseRecyclerPagingAdapter` is built on top of Android Paging Support Library.

**See Output :**

!\[1_0B8ZLJrE7avCzNjZeC-SLg.gif =253x450\](https://miro.medium.com/max/253/1_0B8ZLJrE7avCzNjZeC-SLg.gif =253x450 "1*0B8ZLJrE7avCzNjZeC-SLg.gif =253x450")

# 💻 Getting Started :

Let’s get start to the code!

Open _Android Studio._ Create a new project OR you can simply _clone this repository:_ [https://github.com/PatilShreyas/FirebaseRecyclerPagination.git](https://github.com/PatilShreyas/FirebaseRecyclerPagination.git "https://github.com/PatilShreyas/FirebaseRecyclerPagination.git")

First of all, go to Firebase Console and create a new Android Project. Download configuration file i.e. _google-services.json_ and place it in the **/app** directory.

In this app, you are showing a paginated list of Posts. Posts will load in `RecyclerView.`

## Gradle Setup

```gradle
    dependencies {
    
        //RecyclerView
        implementation 'com.android.support:recyclerview-v7:28.0.0'
    
        //Firebase Database
        implementation 'com.google.firebase:firebase-database:16.1.0'
        implementation 'com.google.firebase:firebase-core:16.0.7'
        
        //Firebase-UI Library
        implementation 'com.firebaseui:firebase-ui-database:4.3.1'
    
        //Android Paging Libray
        implementation "android.arch.paging:runtime:1.0.1"
    
        //Firebase Pagination Library
        implementation 'com.shreyaspatil:FirebaseRecyclerPagination:1.0.1'
    }
```

## App Setup

Make model class (Consider `Post.java`) in the app.

```java
public class Post {
    public String title;
    public String body;

    public Post(){}

    public Post(String title, String body) {
        this.title = title;
        this.body = body;
    }
}
```

### Initialize :

Don’t forget to set _`LayoutManager`_ to the RecyclerView.  
Set it using _`RecyclerView#setLayoutManager()`_

```java

       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);

           mSwipeRefreshLayout = findViewById(R.id.swipe_refresh_layout);
 
           //Initialize RecyclerView
           mRecyclerView = findViewById(R.id.recycler_view);
           mRecyclerView.setHasFixedSize(true);

           LinearLayoutManager mManager = new LinearLayoutManager(this);
           mRecyclerView.setLayoutManager(mManager);

           //Initialize Database
           mDatabase = FirebaseDatabase.getInstance().getReference().child("posts");
           
```

### Setup Configuration for PagedList

First of all configure PagedList  
_Remember that, the size you will pass to `setPageSize()` a method will load x3 items of that size._ (Here, in this example we passed value 10. So, it will load 10x3 i.e. 30 items)

```java
PagedList.Config config = new PagedList.Config.Builder()
                .setEnablePlaceholders(false)
                .setPrefetchDistance(5)
                .setPageSize(10)
                .build();
```

Then Configure Adapter by building FirebasePagingOptions. It will generic.  
_Remember one thing, don’t pass Query with `orderByKey()`, `limitToFirst()` or `limitToLast()`. This will cause an error._

```java
DatabasePagingOptions<Post> options = new DatabasePagingOptions.Builder<Post>()
                .setLifecycleOwner(this)
                .setQuery(mDatabase, config, Post.class)
                .build();
```

### Initialize Adapter

_`FirebaseRecyclerPagingAdapter`_ is built on the top of Android Architecture Components - Paging Support Library. To implement, you should already have _`RecyclerView.ViewHolder`_ subclass. Here We used _`PostViewHolder`_ class.

```java
        mAdapter = new FirebaseRecyclerPagingAdapter<Post, PostViewHolder>(options) {
            @NonNull
            @Override
            public PostViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
                return new PostViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.item_list, parent, false));
            }

            @Override
            protected void onBindViewHolder(@NonNull PostViewHolder holder,
                                         int position,
                                         @NonNull Post model) {

                holder.setItem(model);
            }
            
            @Override
            protected void onLoadingStateChanged(@NonNull LoadingState state) {
                switch (state) {
                    case LOADING_INITIAL:
                    case LOADING_MORE:
                        // Do your loading animation
                        mSwipeRefreshLayout.setRefreshing(true);
                        break;

                    case LOADED:
                        // Stop Animation
                        mSwipeRefreshLayout.setRefreshing(false);
                        break;

                    case FINISHED:
                        //Reached end of Data set
                        mSwipeRefreshLayout.setRefreshing(false);
                        break;

                    case ERROR:
                        retry();
                        break;
                }
        };
```

Any changes occur in the adapter will result in the callback _`onLoadingStateChanged()`_

### Get Child Reference

To get the reference of a child from a list. `FirebaseRecyclerPagingAdapter` has a method called `getRef()`. You can obtain `DatabaseReference` of the child using it.  
Get it using `FirebaseRecyclerPagingAdapter#getRef()` For e.g.

```java
      @Override
      protected void onBindViewHolder(@NonNull PostViewHolder holder,
                                   int position,
                                   @NonNull Post model) {

          DatabaseReference reference = getRef(position);
      }
```

### Error Handling

To get to know about `DatabaseError` caught during Paging, Override method `onError()` in the adapter.

```java
      @Override
      protected void onError(@NonNull DatabaseError databaseError) {
          mSwipeRefreshLayout.setRefreshing(false);
          databaseError.toException().printStackTrace();
          // Handle Error

      }
```

### Retrying List (After Error / Failure)

To retry items loading in RecyclerView, `retry()` method from Adapter class is used.  
Use it as `FirebaseRecyclerPagingAdapter#retry()`.  
This method should be used only after caught in Error. `retry()` should not be invoked anytime other than ERROR state.  
Whenever `LoadingState` becomes `LoadingState.ERROR` we can use `retry()` to load items in RecyclerView which were unable to load due to recent failure/error and to maintain Paging List stable.  
See the demo for a method.

```java

        @Override
        protected void onError(@NonNull DatabaseError databaseError) {
            retry();          
        }
```

Or outside `FirebaseRecyclerPagingAdapter`

```java
	mAdapter.retry();
```

### Refreshing List

To refresh items in RecyclerView, `refresh()` method from Adapter class is used.  
Use it as `FirebaseRecyclerPagingAdapter#refresh()`.  
This method clears all the items in RecyclerView and reloads the data again from the beginning.  
See the demo for a method.

```java
            mSwipeRefreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
                @Override
                public void onRefresh() {
                    mAdapter.refresh();
                }
            });
```

### Set Adapter

Finally, Set adapter to RecyclerView.

```java
    mRecyclerView.setAdapter(mAdapter);
```

### Lifecycle

At last, To begin populating data, call `startListening()` method. `stopListening()` stops the data being loaded.

```java
        //Start Listening Adapter
        @Override
        protected void onStart() {
            super.onStart();
            mAdapter.startListening();
        }
    
        //Stop Listening Adapter
        @Override
        protected void onStop() {
            super.onStop();
            mAdapter.stopListening();
        }
```

_Thus, we have implemented the **Firebase Recycler Pagination**._ 😃

You can see the _full app demo_ on below-listed resources with source code and step-by-step guide.

**_Thank You!_ 😃**

If you need any help get in touch with me on [Facebook](https://www.facebook.com/shreyaspatil99), [Twitter](https://twitter.com/imShreyasPatil), [LinkedIn](https://www.linkedin.com/in/patil-shreyas), [GitHub](https://github.com/PatilShreyas), [Personal Site](https://patilshreyas.github.io/).

## Resources

* GitHub Repository:

  [https://github.com/PatilShreyas/FirebaseRecyclerPagination](https://github.com/PatilShreyas/FirebaseRecyclerPagination "https://github.com/PatilShreyas/FirebaseRecyclerPagination")
* Firebase Open Source:

  [https://firebaseopensource.com/projects/patilshreyas/firebaserecyclerpagination/](https://firebaseopensource.com/projects/patilshreyas/firebaserecyclerpagination/ "https://firebaseopensource.com/projects/patilshreyas/firebaserecyclerpagination/")