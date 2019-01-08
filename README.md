# embedded-basic

Very small Basic-like script language for resource-constrained embedded system.

Sometime during embedded system development it is very useful to have simple scripting.
For example, to easy change device functionality for different customer requirements.
Such device usually resource constrained, so script implementation should be very light both for code and RAM usage.

The project based of perfect BASIC interpreter from Jerry Williams (see https://github.com/jwillia3/BASIC).
See there also for language description.

This script language implemented as compiler to some intermediate p-code, which is executed on simple vitrual machine in device.
This approach will greatly simplify the requirements for the target device because the lexical analyzer and the code generator run on the host.

The script language support, depend on compile-time directive, one of three type of data - int32, float32 or fixed point 16.16 format.
During compile you can enable match library. Match library implement standard set of math function (sin, cos, tan, ln, log ...).
Also, it is possible to extend script language with user-defined keywords or dynamic function (dynamic function is a function (re)defined during run-time)

The memory requirements of virtual machine on device side (ARM-thumb):

                    |    INT32     |      FX16Q16      |       FLOAT32
--------------------|--------------|-------------------|----------------
Math lib disabled   |    9K        |      11K          |       19K             
Math lib enabled    |    N/A       |      14K          |       43K     


How to use the library.
----------------------

The best way to understand how to use the library is explore file 'test.c',
and follow general recommendations:

1. Set defines in basic.h file:

#define ENABLE_COMPILER			ON/OFF - Compile with ON for host and OFF for target device.
#define ENABLE_CHECK_STACK		ON/OFF - Enable/Disable check stack overflow on target virtual machine.
#define ENABLE_DASM				ON/OFF - Enable/Disable disassembler code. It useful for debug.
#define ENABLE_MATH				ON/OFF - Enable match library
#define DATATYPE				TYPE_FLOAT32 or TYPE_FX16Q16 or TYPE_INT32 - supported data types. Note that TYPE_INT32 cannot be enabled together with ENABLE_MATH directive.

2. Define new language keyword or function (if need) with function 	bs_reg_keyword(), bs_reg_func(), bs_reg_opcode(). The functionality some of then can be empty on host side, but they must be registered.

3. Compile with bs_compile() and export p-code fo file with bs_export_pcode(). In case of error get error description with bs_last_error().

4. On device side disable compiler (ENABLE_COMPILER OFF) and check that DATATYPE and ENABLE_MATH is consistent. 

5. Initialize VM with bs_init_load().

6. Define same opcode as you define before on host, now with real functionality. bs_reg_opcode()

7. Define dynamic function with bs_reg_dyn_fn().

8. You can inject some variables to script before run with bs_set_var() function.

9. Execute script with bs_run(). Here you can also define trace function for debug purposes.

10. Freeing resources with bs_free()


