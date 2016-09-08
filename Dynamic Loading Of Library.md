# dlopen()

```void * dlopen(const char *filename, int flag);```

- If filename begins with `/` (i.e., it's an absolute path), dlopen() will just try to use it (it won't search for a library). 
- Otherwise, `dlopen()` will search for the library in the following order:
	1. A colon-separated list of directories in the user's `LD_LIBRARY_PATH` environment variable.
	2. The list of libraries specified in `/etc/ld.so.cache` (which is generated from `/etc/ld.so.conf`).
	3. `/lib`, followed by `/usr/lib`.

### Main flags

- **RTLD_LAZY**	= resolve undefined symbols as code from the dynamic library is executed
- **RTLD_NOW**	= resolve all undefined symbols before dlopen() returns and fail if this cannot be done

### Flags to use for OR'ing

- **RTLD_GLOBAL**	= Symbols defined by this library will be made available for symbol resolution of subsequently loaded libraries. 
- **RTLD_LOCAL**	= Symbols defined in this library are not made available to resolve references in subsequently loaded libraries. 
- **RTLD_NODELETE**	= Do not unload the library during `dlclose()`. Consequently, the library's static variables are not reinitialized if the library is reloaded with `dlopen()` at a later time.
- **RTLD_NOLOAD**	= Don't load the library. To test library is already loaded or not. Can also be used to reload library, for example a library that was previously loaded with `RTLD_LOCAL` can be reopened with `RTLD_NOLOAD | RTLD_GLOBAL`.
- **RTLD_DEEPBIND**	= Place the lookup scope of the symbols ahead of the global scope. This means use own symbols in preference to other already loaded library symbols


- While you're debugging, you'll probably want to use `RTLD_NOW`; using `RTLD_LAZY` can create inscrutable errors if there are unresolved references. Using `RTLD_NOW` makes opening the library take slightly longer (but it speeds up lookups later); if this causes a user interface problem you can switch to `RTLD_LAZY` later.
