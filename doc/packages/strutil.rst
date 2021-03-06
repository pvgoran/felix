Package: src/packages/strutil.fdoc


=========================
String Support Utilities.
=========================

====================== ====================================
key                    file                                 
====================== ====================================
flx_strutil.hpp        share/lib/rtl/flx_strutil.hpp        
flx_strutil.cpp        share/src/strutil/flx_strutil.cpp    
flx_i18n.hpp           share/lib/rtl/flx_i18n.hpp           
flx_i18n.cpp           share/src/strutil/flx_i18n.cpp       
unix_flx_strutil.fpc   $PWD/src/config/unix/flx_strutil.fpc 
win_flx_strutil.fpc    $PWD/src/config/win/flx_strutil.fpc  
flx_i18n.fpc           $PWD/src/config/flx_i18n.fpc         
flx_strutil_config.hpp share/lib/rtl/flx_strutil_config.hpp 
flx_strutil.py         $PWD/buildsystem/flx_strutil.py      
====================== ====================================


String utilities
================


.. code-block:: cpp

  //[flx_strutil.hpp]
  
  #ifndef __FLX_STRUTIL_HPP_
  #define __FLX_STRUTIL_HPP_
  
  #include <string>
  #include <sstream>
  #include <iomanip>
  #include <cstdarg>
  #include <cstdlib>
  #include <cstring>
  
  
  #include "flx_strutil_config.hpp"
  
  //RF: was only to commented out to fix macosx problem,
  //but lets see what happens to all the other builds.
  //#ifndef MACOSX
  //template class RTL_EXTERN std::basic_string<char>;
  //#endif
  
  namespace flx { namespace rtl { namespace strutil {
    using namespace std;
    template<class T>
    basic_string<T> mul(basic_string<T> s, int n) {
      basic_string<T> r = "";
      while(n--) r+=s;
      return r;
    }
  
    // normalise string positions Python style
    // note substr requires 0<=b<=size, 0<=n,
    // however n>size is OK
    template<class T>
    basic_string<T> substr(basic_string<T> const &s, int b, int e)
    {
      int n = s.size();
      if(b<0)  b=b+n;
      if(b<0)  b=0;
      if(b>=n) b=n;
      if(e<0)  e=e+n;
      if(e<0)  e=0;
      if(e>=n) e=n;
      int m =  e-b;
      if(m<0)  m=0;
      return s.substr(b,m);
    }
  
    template<class T>
    T subscript(basic_string<T> const &s, int i)
    {
      int n = s.size();
      if(i<0)  i=i+n;
      return i<0 || i >= n ? T(0) : s[i];
    }
  
    template<class T>
    string str(T const &t) {
      std::ostringstream x;
      x << t;
      return x.str();
    }
  
    template<class T>
    string fmt_default(T const &t, int w, int p) {
      std::ostringstream x;
      x << std::setw(w) << std::setprecision(p) << t;
      return x.str();
    }
  
    template<class T>
    string fmt_fixed(T const &t, int w, int p) {
      std::ostringstream x;
      x << std::fixed << std::setw(w) << std::setprecision(p) << t;
      return x.str();
    }
  
    template<class T>
    string fmt_scientific(T const &t, int w, int p) {
      std::ostringstream x;
      x << std::scientific << std::setw(w) << std::setprecision(p) << t;
      return x.str();
    }
  
  
    STRUTIL_EXTERN string atostr(char const *a);
    STRUTIL_EXTERN string flx_asprintf(char const *fmt,...);
  
    STRUTIL_EXTERN string flxid_to_cid(string const&);
    STRUTIL_EXTERN string filename_to_modulename (string const&);
    STRUTIL_EXTERN size_t string_hash(string const &s); 
    STRUTIL_EXTERN char *flx_strdup(char const *); 
    STRUTIL_EXTERN char *flx_cstr(::std::basic_string<char> const&); 
  
  }}}
  
  #endif


.. code-block:: cpp

  //[flx_strutil.cpp]
  
  #include <stdio.h>
  #include <cstdint>
  #include <cstring>
  
  #include "flx_strutil.hpp"
  
  namespace flx { namespace rtl { namespace strutil {
  
    char *flx_strdup(char const *p) {
      if (p==0) return NULL; 
      auto n = ::std::strlen (p);
      auto q = (char*) ::std::malloc(n+1);
      strcpy (q,p);
      return q;
    }
  
    char *flx_cstr(::std::basic_string<char> const& s) {
      auto n = s.size();
      auto q = (char*) ::std::malloc(n+1);
      auto p = s.c_str();
      ::std::memcpy(q,p,n);
      q[n] = 0;
      return q; 
    }
  
    string atostr(char const *a) {
      if(a) return a;
      else return "";
    }
  
    size_t string_hash(string const &s)
    {
      size_t hash = 5381;
      int c;
      char const *str = s.c_str();
      while (c = *str++)
          hash = (hash * 33 + c) % (size_t)1073741823ll; /* hash * 33 + c */
      return hash;
    }
   
    string flxid_to_cid (string const &s)
    {
      string out = "";
      int n = s.size();
      // leading digit
      if (n > 1 && s[0] >= '0' && s[0] <= '9') out += "_";
      for (int i = 0; i < n; ++i)
      {
        char ch = s[i];
        /* from http://www.w3.org/TR/html4/sgml/entities.html */
        switch (ch)
        {
          case ' ': out += "__sp_"; break;
          case '!': out += "__excl_"; break;
          case '"': out += "__quot_"; break;
          case '#': out += "__num_"; break;
          case '$': out += "__dollar_"; break;
          case '%': out += "__percnt_"; break;
          case '&': out += "__amp_"; break;
          case '\'':  out +=  "__apos_"; break;
          case '(': out += "__lpar_"; break;
          case ')': out += "__rpar_"; break;
          case '*': out += "__ast_"; break;
          case '+': out += "__plus_"; break;
          case ',': out += "__comma_"; break;
          case '-': out += "__hyphen_"; break;
          case '.': out += "__period_"; break;
          case '/': out += "__sol_"; break;
          case ':': out += "__colon_"; break;
          case ';': out += "__semi_"; break;
          case '<': out += "__lt_"; break;
          case '=': out += "__equals_"; break;
          case '>': out += "__gt_"; break;
          case '?': out += "__quest_"; break;
          case '@': out += "__commat_"; break;
          case '[': out += "__lsqb_"; break;
          case '\\': out += "__bsol_"; break;
          case ']': out += "__rsqb_"; break;
          case '^': out += "__caret_"; break;
          case '`': out += "__grave_"; break;
          case '{': out += "__lcub_"; break;
          case '|': out += "__verbar_"; break;
          case '}': out += "__rcub_"; break;
          case '~': out += "__tilde_"; break;
          default: out += string (1,ch);
        }
     }
     if (out.size() > 40) 
       return out.substr(0,4) + flx_asprintf("_hash_%zu",string_hash(out));
     else
       return out;
    }
  
    string chop_extension (string const &s)
    {
       int n = s.size();
       for(int i = n - 1; i >= 0; --i) 
       {
         if (s[i] == '/') return s;
         if (s[i] == '\\') return s;
         if (s[i] == '.') return s.substr(0,i);
       }
       return s;
    }
  
    string basename (string const &s) 
    {
       int n = s.size();
       for(int i = n - 1; i >= 0; --i) 
       {
         if (s[i] == '/') return s.substr (i+1,n-i);
         if (s[i] == '\\') return s.substr (i+1,n-i);
       }
       return s;
    }
    string filename_to_modulename (string const &s)
    {
       string a = basename (s);
       a = chop_extension (a);
       a = flxid_to_cid (a);
       return a; 
    }
  
  #ifdef FLX_HAVE_VSNPRINTF
    string flx_asprintf(char const *fmt,...){
      va_list ap;
      va_start(ap,fmt);
      //printf("vsnprintf TRIAL\n");
      int n = vsnprintf(NULL,0,fmt,ap);
      //printf("vsnprintf size=%d\n",n);
      va_end(ap);
      char *res = new char[n + 1];
      va_start(ap,fmt);
      vsnprintf(res,n+1,fmt,ap);
      va_end(ap);
      string s = string(res);
      delete [] res;
      return s;
    }
  #else
    // THIS IS UNSAFE .. but Windows sucks.
    // It documents vsnprintf .. but doesn't provide it
    string flx_asprintf(char const *fmt,...){
      //printf("vsnprintf EMULATION!\n");
      va_list ap;
      int n = 10000; // hack, WILL crash if not enough
      char *res = new char[n+1];
      va_start(ap,fmt);
      vsprintf(res,fmt,ap);
      va_end(ap);
      string s = string(res);
      delete [] res;
      return s;
    }
  #endif
  
  }}}


.. code-block:: cpp

  //[flx_strutil_config.hpp]
  #ifndef __FLX_STRUTIL_CONFIG_H__
  #define __FLX_STRUTIL_CONFIG_H__
  #include "flx_rtl_config.hpp"
  #ifdef BUILD_STRUTIL
  #define STRUTIL_EXTERN FLX_EXPORT
  #else
  #define STRUTIL_EXTERN FLX_IMPORT
  #endif
  #endif


.. code-block:: fpc

  //[unix_flx_strutil.fpc]
  Name: flx_strutil
  Description: String utilities
  provides_dlib: -lflx_strutil_dynamic
  provides_slib: -lflx_strutil_static
  includes: '"flx_strutil.hpp"'
  macros: BUILD_STRUTIL
  library: flx_strutil
  srcdir: src/strutil
  src: .*\.cpp


.. code-block:: fpc

  //[win_flx_strutil.fpc]
  Name: flx_strutil
  Description: String utilities
  provides_dlib: /DEFAULTLIB:flx_strutil_dynamic
  provides_slib: /DEFAULTLIB:flx_strutil_static
  includes: '"flx_strutil.hpp"'
  macros: BUILD_STRUTIL
  library: flx_strutil
  srcdir: src/strutil
  src: .*\.cpp


UTF codec.
----------


.. code-block:: cpp

  //[flx_i18n.hpp]
  
  #ifndef __FLX_I18N_H__
  #define __FLX_I18N_H__
  #include <string>
  #include "flx_strutil_config.hpp"
  
  namespace flx { namespace rtl { namespace i18n {
     STRUTIL_EXTERN std::string utf8(unsigned long);
  }}}
  #endif


.. code-block:: cpp

  //[flx_i18n.cpp]
  
  #include "flx_i18n.hpp"
  namespace flx { namespace rtl { namespace i18n {
    std::string utf8(unsigned long i)
    {
      char s[7];
      if (i < 0x80UL )
      {
        s[0]= i;
        s[1]= 0;
      }
      else if (i < 0x800UL )
      {
        s[0]=0xC0u | (i >> 6ul)  & 0x1Fu;
        s[1]=0x80u | i           & 0x3Fu;
        s[2]=0;
      }
      else if (i < 0x10000UL )
      {
        s[0]=0xE0u | (i >> 12ul) & 0xFu;
        s[1]=0x80u | (i >> 6ul)  & 0x3Fu;
        s[2]=0x80u | i           & 0x3F;
        s[3]=0;
      }
      else if (i < 0x200000UL )
      {
        s[0]=0xF0u | (i >> 18ul) & 0x7u;
        s[1]=0x80u | (i >> 12ul) & 0x3Fu;
        s[2]=0x80u | (i >> 6ul)  & 0x3Fu;
        s[3]=0x80u | i           & 0x3F;
        s[4]=0;
      }
      else if (i < 0x4000000UL )
      {
        s[0]=0xF8u | (i >> 24ul) & 0x3u;
        s[1]=0x80u | (i >> 18ul) & 0x3Fu;
        s[2]=0x80u | (i >> 12ul) & 0x3Fu;
        s[3]=0x80u | (i >> 6ul)  & 0x3Fu;
        s[4]=0x80u | i           & 0x3Fu;
        s[5]=0;
      }
      else
      {
        s[0]=0xFCu | (i >> 30ul) & 0x1u;
        s[1]=0x80u | (i >> 24ul) & 0x3Fu;
        s[2]=0x80u | (i >> 18ul) & 0x3Fu;
        s[3]=0x80u | (i >> 12ul) & 0x3Fu;
        s[4]=0x80u | (i >> 6ul)  & 0x3Fu;
        s[5]=0x80u | i           & 0x3Fu;
        s[6]=0;
      }
      return s;
    }
  }}}


Config database entry 
======================


.. code-block:: fpc

  //[flx_i18n.fpc]
  Name: flx_i18n
  Description: Internationalisation support, Unicode, utf8
  Requires: flx_strutil
  includes: '"flx_i18n.hpp"'


.. code-block:: python

  #[flx_strutil.py]
  import fbuild
  from fbuild.path import Path
  from fbuild.record import Record
  from fbuild.builders.file import copy
  
  import buildsystem
  
  # ------------------------------------------------------------------------------
  
  def build_runtime(phase):
      print('[fbuild] [rtl] build strutil')
      path = Path(phase.ctx.buildroot/'share'/'src'/'strutil')
      srcs = [f for f in Path.glob(path / '*.cpp')]
      includes = [phase.ctx.buildroot / 'host/lib/rtl', phase.ctx.buildroot / 'share/lib/rtl']
      macros = ['BUILD_STRUTIL']
  
      dst = 'host/lib/rtl/flx_strutil'
      return Record(
          static=buildsystem.build_cxx_static_lib(phase, dst, srcs,
              includes=includes,
              macros=macros),
          shared=buildsystem.build_cxx_shared_lib(phase, dst, srcs,
              includes=includes,
              macros=macros))


