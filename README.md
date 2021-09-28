
# compile but not link
# The -I flag points to where the "header files" with extension .h 
# live (usually the "include" directory) 
# That's just so that the compiler can make sense of the
# references made in embed.cpp, but at this point the actual 
# substance of those referenced functions is not needed. The header
# files just "declare" the functions and their signatures. The
# actual implementation should be in a library file that will be
# loaded and "linked" in the next step. The output of this step
# is an "object file" with the actual machine code of embed.cpp but
# only references to Julia functions called from embed.cpp
# references to functions 
g++ -c -I/home/navid/Software/julia/include/julia/ embed.cpp 

# Link the object file (linker options missing, there will be problems)
# Here we "link" the object file for embed.cpp (embed.o) with the library
# whose functions it referenced. The -L flag defines the path where
# the linker will be looking for the said library (those library files
# usually have the .so extension when they are "shared libraries" which
# is the case with the julia library) and the -l flag specifies the name
# of the library that should be linked with embed.o. This example will
# compile, but will have runtime errors. Continue to read!
g++ -L/home/navid/Software/julia/lib/ embed.o -ljulia 

# To properly link the julia library, here's what you do:
# note the -Wl,... flag. the -fPIC flag is also important, so do 
# include it, but the main point is the -Wl flag. This should compile
# AND run without runtime errors.
g++ -fPIC  -L$JULIA/lib/ embed.o -Wl,-rpath,$JULIA/lib/ -ljulia 

# Or to do the whole thing in one step,
g++ -o myprogram -fPIC -I$JULIA/include/julia  -L$JULIA/lib/ embed.cpp -Wl,-rpath,$JULIA/lib/ -ljulia 



# But why?  why do we have to specify both -L$JULIA/lib/
# and -Wl,-rpath,$JULIA/lib/? The first one tells the linker
# itself where to find the library files (in this case,
# shared objects). In the second case, the linker needs to
# insert into the final executable the path to needed
# runtime libraries that it will need to load at runtime, 
# and -rpath specifies which runtime library path to insert.
# You can omit the second one and the linker succeeds, but at runtime,
# the executable won't be able to find the correct shared 
# libraries and will fail.



# FURTHER READING:
# WHy not try to link julia statically like this?
g++  -L/home/navid/Software/julia/lib/ embed.o -Wl,-Bstatic,-rpath,/home/navid/Software/julia/lib/ -ljulia

# This won't work because currently, julia only exists as a shared
# library and the -Bstatic flag forces the linker to only look for
# static versions of libraries which in this case fails.
# Therefore, currently, linking julia into a standalone executable 
# or a static library is not possible and you'll have to have the
# julia shared libraries installed on the machine where the final
# executable runs.

