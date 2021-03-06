Package: src/packages/memory.fdoc


=================
Memory Operations
=================

=========== ================================
key         file                             
=========== ================================
memory.flx  share/lib/std/scalar/memory.flx  
address.flx share/lib/std/scalar/address.flx 
=========== ================================


Raw Address
===========


.. index:: Address(class)
.. index:: isNULL(fun)
.. index:: NULL(const)
.. index:: addrstrfmt(const)
.. index:: addrreprfmt(const)
.. code-block:: felix

  //[address.flx]
  
  //$ Core operations on addresses.
  open class Address {
    //$ Construct from Felix object pointer.
    ctor[T] address: &T = "(void*)$1";
  
    //$ Construct from possibly NULL pointer.
    ctor[T] address: cptr[T] = "(void*)$1"; //@
  
    //$ Construct from possibly array element pointer.
    ctor[T] address: +T = "(void*)$1";
  
    //$ Construct from C function
    ctor[D,C] address: D --> C = "(void*)$1";
  
  
    //$ Check is an address is NULL.
    fun isNULL: address -> bool = "(0==$1)";
  
    //$ Define NULL address.
    const NULL : address = "NULL";
  
    instance Eq[address] {
      fun == : address * address -> bool = "$1==$2";
    }
    instance Tord[address] {
      fun < : address * address -> bool = "::std::less<void*>()($1,$2)";
    }
  
    const addrstrfmt : +char = '"%" PRIxPTR' requires C99_headers::inttypes_h;
    const addrreprfmt : +char = '"0x%" PRIxPTR' requires C99_headers::inttypes_h;
  
    instance Str[address] {
      fun str (t:address) : string => vsprintf (addrstrfmt, C_hack::cast[uintptr] t);
    }
    instance Repr[address] {
      fun repr (t:address) : string => vsprintf (addrreprfmt, C_hack::cast[uintptr] t);
    }
  
  
    instance Str[byte] {
      fun str (t:byte) : string => vsprintf (c"%02x", C_hack::cast[uint] t);
    }
  
    instance Repr[byte] {
      fun repr (t:byte) : string => vsprintf (c"0x%02x", t);
    }
  
  
    fun + : address * !ints -> address = "(void*)((char*)$1+$2)";
    fun - : address * !ints -> address = "(void*)((char*)$1-$2)";
    fun - : address * address -> ptrdiff = "(char*)$1-(char*)$2";
  }
  
  open Eq[byte];
  open Tord[address];


.. index:: Memory(class)
.. index:: memcpy(proc)
.. index:: memmove(proc)
.. index:: memcmp(fun)
.. index:: memchr(fun)
.. index:: memset(proc)
.. index:: calloc(gen)
.. index:: free(proc)
.. index:: realloc(gen)
.. index:: raw_malloc(gen)
.. index:: malloc(gen)
.. index:: search(fun)
.. code-block:: felix

  //[memory.flx]
  class Memory
  {
    proc memcpy: address * address * !ints =
      "{if($1 && $2 && $3)::std::memcpy($1,$2,$3);}"
      requires Cxx_headers::cstring
    ;
  
    proc memmove: address * address * !ints =
      "{if($1 && $2 && $3)::std::memmove($1,$2,$3);}"
      requires Cxx_headers::cstring
    ;
  
    fun memcmp: address * address * !ints -> int = 
      "::std::memcmp($1,$2,$3)"
      requires Cxx_headers::cstring
    ;
  
    fun memchr: address * byte * !ints -> address = 
      "::std::memchr($1,$2,$3)"
      requires Cxx_headers::cstring
    ;
  
  
    proc memset: address * !ints * byte = 
      "::std::memset($1,$2,$3);"
      requires Cxx_headers::cstring
    ;
  
    //$ Heap operations
    gen calloc: !ints -> address = 
      "::std::calloc($1)"
      requires Cxx_headers::cstdlib
    ;
  
    proc free: address = 
      "::std::free($1);"
      requires Cxx_headers::cstdlib
    ;
  
    gen realloc: address * !ints -> address = 
      "::std::realloc($1,$2)"
      requires Cxx_headers::cstdlib
    ;
  
    //$ Raw unchecked malloc.
    gen raw_malloc: !ints -> address = 
      '::std::malloc($1)' 
      requires Cxx_headers::cstdlib
    ;
  
    //$ Malloc with memory check.
    //$ Throws c"out of memory" if out of memory.
    body checked_malloc = """
      void *checked_malloc(size_t n) {
        void *p = ::std::malloc(n);
        if(p) return p;
        else throw "out of memory";
      }
    """; 
  
    gen malloc: !ints -> address = 'checked_malloc($1)' 
      requires Cxx_headers::cstdlib, checked_malloc
    ;
  
    // Standard C++ Search algorithm, 
    // returns address of found string
    // or $2 = pointer past end on fail
    fun search: address ^ 4 -> address = 
      """
      (void*)::std::search(
        (::std::uint8_t*)$1,
        (::std::uint8_t*)$2,
        (::std::uint8_t*)$3,
        (::std::uint8_t*)$4)
      """
      requires Cxx_headers::algorithm
    ;
  }
  
  
  
