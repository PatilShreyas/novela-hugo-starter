+++
authors = []
date = 2020-05-05T18:30:00Z
excerpt = "hi"
hero = "/images/firebaseflow.jpeg"
tags = ["android", "kotlin", "firebase"]
timeToRead = 3
title = "Fire flow"

+++
# Hi there

This is body

```cpp
#include<stdio.h>
void main() 
{
	printf("Hello World");
}
```

```kotlin
package dev.shreyaspatil.firebase.coroutines.utils

import com.google.firebase.firestore.CollectionReference
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.cancel
import kotlinx.coroutines.channels.awaitClose
import kotlinx.coroutines.flow.callbackFlow

@ExperimentalCoroutinesApi
fun CollectionReference.getSnapshotFlow() = callbackFlow {
    
    // Register listener
    val listener = addSnapshotListener { snapshot, exception ->
        
        offer(snapshot)

        // If exception occurs, cancel this scope with exception message.
        exception?.let {
            cancel(it.message.toString())
        }
    }

    awaitClose {
        // This block is executed when producer channel is cancelled
        // This function resumes with a cancellation exception.
        // Dispose listener
        listener.remove()
        cancel() 
    }
}
```

Thanks!