# lock

This is a module for enabling file locks. It uses `fcntl` to accomplish this.

POSIX operating systems implement advisory file locking. This enables concurrent processes to interact with the same file without conflict, provided they first check for the existence of a lock held by a different process. Mandatory file locking can be enabled on POSIX systems by mounting volumes using the `mount` `-o mand` option and makes the operating system prevent editing of a locked file.

The `fcntl` functionality is implemented in Python by the `fcntl` standard library module. Using this, a lock can be acquired in a way like the following:

```Python
lock_file = open("data.db", "a")
fcntl.lockf(lock_file, fcntl.LOCK_EX | fcntl.LOCK_NB)
```

If the file is locked, the following exception is raised:

```Python
BlockingIOError: [Errno 11] Resource temporarily unavailable
```

With advisory locking, code should test for the existence of a a lock before editing a file. It is the responsibility of the code, not the operating system, to enforce locking correctly. Information about the locks currently held on a file may be retrieved by passing a bytes object as the third argument to the `fcntl.fcntl` function (which takes the place of a pointer to the process ID in the fcntl header). The lock information can be accessed in a way like the following:

```Python
lock_data = struct.pack("hhllhh", fcntl.F_WRLCK, 0, 0, 0, 0, 0)
fcntl.fcntl(lock_file, fcntl.F_GETLK, lock_data)
```

This returns `lock_data` unchanged if there is no lock on the file.

# examples

The following code saves a dictionary to a file in a loop using `lock` to lock the file while the save is happening:

```Python
import random

import lock

while True:
    config = {"a": 1, "b": random.randint(1, 2)}
    lock.save_JSON("config.json", config)
```

The following code loads a dictionary from a file in a loop using `lock` to lock the file while the load is happening:

```Python
import lock

while True:
    config = lock.load_JSON("config.json")
    if config: print(config)
```

These two pieces of code can be used concurrently.
