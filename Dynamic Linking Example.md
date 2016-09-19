- Following example is all in one which covers almost all dynamic linking related APIs & Flags.
- API Covers dladdr, dlclose, dlerror, dlopen, dlsym, etc.
- Flag Covers `RTLD_LAZY`, `RTLD_NOW`, `RTLD_GLOBAL`, `RTLD_LOCAL`, `RTLD_NODELETE`, `RTLD_NOLOAD`, `RTLD_DEEPBIND`, `RTLD_NEXT`, `RTLD_DEFAULT`, etc.


### flags.c
```

/*
 * Test Procedure :
 *
 * i. Create "libsum.so" from sum.c (kept in same folder).
 * 
 * gcc -o libgetsum.so -shared -fPIC getsum.c -D_GNU_SOURCE
 *
 * ii. Create "libgetsum.so" from sum.c (kept in same folder).
 * 
 * gcc -o libsum.so -shared -fPIC sum.c -D_GNU_SOURCE
 *
 * iii. Create "flags" binary from flags.c file
 *
 * gcc -o flags flags.c -ldl -D_GNU_SOURCE
 *
 */
 
/*
 * Verification Method:
 *
 * [libsum.so]             : Loaded successfully
 * [dlsym()]               : Executed successfully
 * sum(0,0)                : 1
 * Error relocating /home/vishal/libgetsum.so: sum: symbol not found
 * [libsum.so]             : Closed successfully
 * [libsum.so]             : Loaded again successfully
 * [libgetsum.so]          : Loaded again successfully
 * [dlsym()]               : Executed successfully
 * getsum(4, 5)            : 11
 * [dlsym()]               : Executed successfully
 * func()                  : Executed Successfully From [libgetsum.so]
 * [dladdr()]              : Executed successfully
 * Function                : funcXYZ
 * Shared Lib              : /home/vishal/libgetsum.so
 *
 */

#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <dlfcn.h>


/*Please Provide Absolute Path Of Both Libs */
#define LIB_1	"Absolute_path_to_libsum.so"
#define LIB_2	"Absolute_path_to_libgetsum.so"


int main(int argc, char **argv)
{
	/*------------------- PASS 1 : Loading Shared Lib 1  -----------------*/
	void *lib1 = dlopen(LIB_1, RTLD_LAZY | RTLD_LOCAL | RTLD_NODELETE );
	if (!lib1) {
		dprintf(1, "[libsum.so]		: Error in finding the library : PASS 1\n");
		fputs(dlerror(), stderr);
		exit(1);
	}
	else{
		dprintf(1, "[libsum.so]		: Loaded successfully \n");
	}
	/*---------------------------------------------------------------------*/
	

	/*------------------ PASS 2 : Finding Symbol & Execute it--------------*/
	int (*sum)(int , int) = (int (*)(int , int))dlsym(lib1, "sum");
	char *error = dlerror();
	if (error != NULL)  {
		dprintf(1, "[dlsym()]		: Error : PASS 2\n");
		fputs(error, stderr);
		exit(1);
	}
	else{
		dprintf(1,"[dlsym()]		: Executed successfully\n");
		printf("sum(0,0)		: %d \n", (*sum)(0, 0));
	}
	/*---------------------------------------------------------------------*/



	/*------------------ PASS 3 : Loading Shared Lib 2  -------------------*/
	void *lib2 = dlopen(LIB_2, RTLD_NOW);
	if (!lib2) {
		fputs(dlerror(), stderr);
		if( dlclose(lib1) == 0 ){	/* Closing Shared Lib 1 */
			dprintf(1, "\n[libsum.so]		: Closed successfully \n");
		}
		else{
			dprintf(1, "\n[libsum.so]		: Error in closing : PASS 3 \n");
		}
	}
	else{
		dprintf(1, "[libgetsum.so]		: Loaded successfully \n");
	}
	/*---------------------------------------------------------------------*/


	/*----------------- PASS 4 : Loading Shared Lib 1 Again ---------------*/
	lib1 = dlopen(LIB_1, RTLD_NOW | RTLD_NOLOAD | RTLD_GLOBAL );
	if (!lib1) {
		dprintf(1, "[libsum.so]		: Error in finding the library : PASS 4\n");
		fputs(dlerror(), stderr);
		exit(1);
	}
	else{
		dprintf(1, "[libsum.so]		: Loaded again successfully \n");
	}
	/*---------------------------------------------------------------------*/

	
	/*----------------- PASS 5 : Loading Shared Lib 2 Again ---------------*/
	lib2 = dlopen(LIB_2, RTLD_NOW  | RTLD_GLOBAL );
	if (!lib2) {
		dprintf(1, "[libgetsum.so]		: Error in finding the library : PASS 5\n");
		fputs(dlerror(), stderr);
		exit(1);
	}
	else{
		dprintf(1, "[libgetsum.so]		: Loaded again successfully \n");
	}
	/*---------------------------------------------------------------------*/

	
	/*------------------ PASS 6 : Finding Symbol & Execute it--------------*/
	int (*getsum)(int , int) = (int (*)(int , int))dlsym(RTLD_DEFAULT, "getsum");
	error = dlerror();
	if (error != NULL)  {
		dprintf(1, "[dlsym()]		: Error : PASS 6\n");
		fputs(error, stderr);
		exit(1);
	}
	else{
		dprintf(1,"[dlsym()]		: Executed successfully\n");
		printf("getsum(4, 5)		: %d\n", (*getsum)(4, 5));
	}
	/*---------------------------------------------------------------------*/


	/*------------------ PASS 7 : Finding Symbol & Execute it--------------*/
	void (*func)(void) = (void (*)(void))dlsym( lib2, "funcXYZ");
	error = dlerror();
	if (error != NULL)  {
		dprintf(1, "[dlsym()]		: Error : PASS 7\n");
		fputs(error, stderr);
		exit(1);
	}
	else{
		dprintf(1,"[dlsym()]		: Executed successfully\n");
		printf("func()			: ");
		(*func)();
	}
	/*---------------------------------------------------------------------*/


	/*------------- PASS 8 : Resolving Function Name By Address -----------*/
	Dl_info  DlInfo;

	if( dladdr(func, &DlInfo) != 0 ){
		dprintf(1, "[dladdr()]		: Executed successfully  \n");
		dprintf(1, "Function		: %s  \n", DlInfo.dli_sname);
		dprintf(1, "Shared Lib		: %s  \n", DlInfo.dli_fname);
	}
	else{
		dprintf(1, "[dladdr()]              : Error : PASS 8\n");
		exit(1);
	}
	/*---------------------------------------------------------------------*/
	
	dlclose(lib1);
	dlclose(lib2);

	return 0;
}
```
### sum.c

```
#include <stdio.h>
#include <dlfcn.h>

int sum(int a, int b)
{
        static int i=0;
        i++;
        return a + b + i;
}

void funcXYZ()
{
        static void (*fptr)(void) = NULL;

        /* Come here only first time */
        if (fptr == NULL) {
                fptr = (void (*)(void))dlsym(RTLD_NEXT, "funcXYZ");
                if (fptr == NULL) {
                        printf("dlsym: %s\n", dlerror());
                        return ;
                }
        }

        (*fptr)();
}
```

### getsum.c
```
#include<stdio.h>

extern int sum(int,int);

int getsum(int a, int b)
{
        return sum(a,b);
}

void funcXYZ()
{
        printf("Executed Successfully From [libgetsum.so]\n");
}
```
