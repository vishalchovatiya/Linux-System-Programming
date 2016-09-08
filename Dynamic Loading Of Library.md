# dlopen()

```void * dlopen(const char *filename, int flag);```

- filename

	- If filename begins with `/` (i.e., it's an absolute path), dlopen() will just try to use it (it won't search for a library). 
	- Otherwise, `dlopen()` will search for the library in the following order:
		1. A colon-separated list of directories in the user's `LD_LIBRARY_PATH` environment variable.
		2. The list of libraries specified in `/etc/ld.so.cache` (which is generated from `/etc/ld.so.conf`).
		3. `/lib`, followed by `/usr/lib`.

- flag
	- Main flags
	
		- **RTLD_LAZY**	= resolve undefined symbols as code from the dynamic library is executed
		- **RTLD_NOW**	= resolve all undefined symbols before dlopen() returns and fail if this cannot be done
	
	- Flags to use for OR'ing
	
		- **RTLD_GLOBAL**	= Symbols defined by this library will be made available for symbol resolution of subsequently loaded libraries. 
		- **RTLD_LOCAL**	= Symbols defined in this library are not made available to resolve references in subsequently loaded libraries. 
		- **RTLD_NODELETE**	= Do not unload the library during `dlclose()`. Consequently, the library's static variables are not reinitialized if the library is reloaded with `dlopen()` at a later time.
		- **RTLD_NOLOAD**	= Don't load the library. To test library is already loaded or not. Can also be used to reload library with new configuration, for example a library that was previously loaded with `RTLD_LOCAL` can be reopened with `RTLD_NOLOAD | RTLD_GLOBAL`.
		- **RTLD_DEEPBIND**	= Place the lookup scope of the symbols ahead of the global scope. This means use own symbols in preference to other already loaded library symbols


- Note :
	- While you're debugging, you'll probably want to use `RTLD_NOW`; using `RTLD_LAZY` can create inscrutable errors if there are unresolved references. Using `RTLD_NOW` makes opening the library take slightly longer (but it speeds up lookups later); if this causes a user interface problem you can switch to `RTLD_LAZY` later.
	- If the libraries depend on each other (e.g., X depends on Y), then you need to load the dependees first (in this example, load Y first, and then X).
	- `dlopen()` will return NULL if the attempt to load does not succeed, and you need to check for this. If the same library is loaded more than once with `dlopen()`, the same file handle is returned

# dlerror()

```char *dlerror(void);```

- Errors can be reported by calling dlerror(), which returns a string describing the error from the last call to dlopen(), dlsym(), or dlclose().
 
# dlsym()

```void *dlsym(void *handle, const char *symbol);```

- The `handle` is the value returned from dlopen.
- And symbol is a NULL-terminated string. 
- `dlsym()` will return a NULL result if the symbol wasn't found.

# dlclose()

```int dlclose(void *handle);```

- Closes a Dynamically loaded library. 
- The Dynamically loaded library maintains link counts for dynamic file handles, so a dynamic library is not actually deallocated until dlclose has been called on it as many times as dlopen has succeeded on it. Thus, it's not a problem for the same program to load the same library multiple times. 
- `dlclose()` returns 0 on success, and non-zero on error; some Linux manual pages don't mention this.

### Eaample

```
    #include <stdlib.h>
    #include <stdio.h>
    #include <dlfcn.h>

    int main(int argc, char **argv) {
        void *handle;
        double (*cosine)(double);
        char *error;

        handle = dlopen ("/lib/libm.so.6", RTLD_LAZY);
        if (!handle) {
            fputs (dlerror(), stderr);
            exit(1);
        }

        cosine = dlsym(handle, "cos");
        if ((error = dlerror()) != NULL)  {
            fputs(error, stderr);
            exit(1);
        }

        printf ("%f\n", (*cosine)(2.0));
        dlclose(handle);
    }
``
