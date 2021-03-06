Package: src/packages/dynamic_metatyping.fdoc


===================
Dynamic Meta-typing
===================

============ ========================================
key          file                                     
============ ========================================
typedesc.flx share/lib/std/felix/dyntype/typedesc.flx 
============ ========================================


First Class Type Descriptor.
============================



.. index:: DynamicMetaTyping(class)
.. index:: typedesc_t(ctor)
.. index:: def(type)
.. index:: def(type)
.. index:: def(type)
.. index:: FelixValueType_class(header)
.. index:: FelixValueType(type)
.. index:: FelixValueType(ctor)
.. index:: FelixValueType(ctor)
.. index:: ValueType_from_FelixValueType(fun)
.. code-block:: felix

  //[typedesc.flx]
  include "std/felix/rtti";
  
  class DynamicMetaTyping
  {
    open Rtti;
   
    interface typedesc_t 
    {
      name: 1 -> string;
      len: 1 -> size; // all the objects
      object_size: 1 -> size; // one object
      object_alignment: 1 -> size;
      dflt_init : address -> 0;
      destroy : address -> 0;
      copy_init : address * address -> 0; 
      move_init : address * address -> 0; // leaves src dflt init
      copy_assign : address * address -> 0;
      move_assign : address * address -> 0; // leaves source dflt init
    }
  
    object CxxType (fcops: Rtti::fcops_t, tname:string, tlen: size)
      implements typedesc_t
    =
    {
      method fun name () => tname;
  
      method fun len() => tlen;
  
      method fun object_size() => Rtti::object_size fcops;
      method fun object_alignment() => Rtti::object_alignment fcops;
  
      method proc dflt_init (dst:address)
        => Rtti::dflt_init (fcops, dst);
  
      method proc destroy(dst:address)
        => Rtti::destroy(fcops, dst);
  
      method proc copy_init (dst:address, src:address)
        => Rtti::copy_init (fcops, dst, src);
  
      method proc move_init (dst:address, src:address)
        => Rtti::move_init (fcops, dst, src);
  
      method proc copy_assign (dst:address, src:address) {
        println$ "Felix CxxType.copy_assign "+dst.str+ " <- " + src.str;
        Rtti::copy_assign (fcops, dst, src);
      }
  
      method proc move_assign(dst:address, src:address)
        => Rtti::move_assign (fcops, dst, src);
    }
  
    ctor typedesc_t (ptd: gc_shape_t) => 
      let fcops = ptd.get_fcops in
      let name = ptd.cname.string in
      let esize = ptd.bytes_per_element * ptd.number_of_elements in
      CxxType (fcops, name, esize)
    ;
  
  
    typedef binproc_t = address * address -> 0; // P2
    typedef unproc_t = address -> 0; // P1
    typedef get_size_t = 1 -> size; // FZ
  
    // NOTE: currently NOT GC aware!
  
    body FelixValueType_poly[P2,A2,P1,FZ] = """
      class FelixValueType : public virtual ValueType 
      {
        ?4 object_size_ptr;
        ?4 object_size_ptr;
        ?3 dflt_init_ptr;
        ?3 destroy_ptr;
        ?1 copy_init_ptr;
        ?1 move_init_ptr;
        ?1 copy_assign_ptr;
        ?1 move_assign_ptr;
  
        size_t object_size() { return sizeof(?1); }
        size_t object_alignment() { return alignof(?1); }
   
        void dflt_init_impl (void *dst) { 
          ::flx::rtl::executil::run (dflt_init_ptr->call(0,dst)); 
        }
  
        void destroy_impl (void *dst) { 
          ::flx::rtl::executil::run (destroy_ptr->call(0,dst)); 
        }
  
        void copy_init_impl (void *dst, void *src) { 
          ::flx::rtl::executil::run (copy_init_ptr->call(0,?2(dst,src))); 
        }
  
        void move_init_impl (void *dst, void *src) { 
          ::flx::rtl::executil::run (move_init_ptr->call(0,?2(dst,src))); 
        }
        
        void copy_assign_impl (void *dst, void *src) { 
          fprintf(stderr, "C++FelixValueType.copy_assign %p<-%p\\n",dst,src);
          ::flx::rtl::executil::run (copy_assign_ptr->call(0,?2(dst,src))); 
        }
  
        void move_assign_impl (void *dst, void *src) { 
          ::flx::rtl::executil::run (move_assign_ptr->call(0,?2(dst,src))); 
        }
  
      public:
        FelixValueType (?3 di, ?3 de, ?1 ci, ?1 mi, ?1 ca, ?1 ma) : 
          dflt_init_ptr (di), 
          destroy_ptr (de), 
          copy_init_ptr (ci), 
          move_init_ptr (mi),
          copy_assign_ptr (ca),
          move_assign_ptr (ma)
          {}
      };
    """;
  
    // Tricky! Declare incomplete type in header
    // Implement class in body
    header FelixValueType_class = "class FelixValueType;" 
      requires FelixValueType_poly[binproc_t, address^2, unproc_t, get_size_t],
      package "flx_executil" 
    ;
  
    type FelixValueType = "FelixValueType*" requires FelixValueType_class;
  
    ctor FelixValueType : copy_t * copy_t * copy_t * copy_t = 
      "new FelixValueType ($1, $2, $3, $4)"
    ;
  
    ctor FelixValueType (x:DynamicMetaTyping::typedesc_t) =>
      FelixValueType (
        x.object_size, 
        x.object_alignment, 
        x.dflt_init, 
        x.destroy_init, 
        x.copy_init, 
        x.move_init, 
        x.copy_assign, 
        x.move_assign
      )
    ;
  
    fun ValueType_from_FelixValueType: FelixValueType -> fcops_t = "(ValueType*)$1";
  
    object TupleType (tname:string, elts: list[typedesc_t]) implements typedesc_t =
    { 
       fun align : size -> size =
         | 0uz => 0uz
         | 1uz => 1uz
         | 2uz => 2uz
         | 3uz => 4uz
         | 4uz => 4uz
         | 5uz => 8uz
         | 6uz => 8uz
         | 7uz => 8uz
         | 8uz => 8uz
         | _ => 16uz
       ;
  
       var n = len elts;
       assert n != 0uz;
  println$ "Tuple " + tname + " with " + n.str + " fields";
       var aligned = varray[typedesc_t * size] n;
       var ofset = 0uz;
       var tl = elts;
       var counter = 0;
    next_elt:>
  println$ "Offset " + ofset.str;
       match  tl with
       | #Empty => ;
       | Cons (head, (Cons (nxt, _) as tail)) =>
  println$ "Add field " + counter.str + "/" + n.str;
         push_back (aligned, (head,ofset));
         // alignment rules: the offset of the next object is 
         // aligned to the greater of the alignment of the current
         // and next objects
         var hz = head.len ();
         var nz = nxt.len ();
         var alignment = max (align hz, align nz);
         ofset = ((ofset + hz + alignment - 1) / alignment) * alignment;
         tl = tail ;
         ++counter;
         goto next_elt;
  
       | Cons (head, #Empty) =>
  println$ "Add last field " + counter.str + "/" + n.str;
         push_back (aligned, (head,ofset));
         hz = head.len ();
         alignment = align hz;
         ofset = ((ofset + hz + alignment - 1) / alignment) * alignment;
       endmatch;
       var length = ofset;
       println$ "Tuple " + tname + " length= " + length.str;
       println$ "Tuple " + tname + " fields= ";
       for var i in 0uz upto n - 1uz do
         var typ,ofs = aligned.i;
         println$ "Field #"+i.str+ " at offset " + ofs.str + " type " + #(typ.name).str;
       done
  
      method fun len () => length;
      method fun name () => tname;
  
      method proc dflt_init (dst:address) =>
        for var i in 0uz upto n - 1uz do
           var typ,ofs = aligned.i;
           typ.dflt_init (dst + ofs);   
        done
  
      method proc destroy(dst:address) =>
        for var i in 0uz upto n - 1uz do
           var typ,ofs = aligned.i;
           typ.destroy(dst + ofs);   
        done
  
  
  
      method proc copy_init (dst:address, src:address) =>
        for var i in 0uz upto n - 1uz do
           var typ,ofs = aligned.i;
           typ.copy_init (dst + ofs, src + ofs);   
        done
  
      method proc move_init (dst:address, src:address)  => 
       perform assert false;
  
      method proc copy_assign (dst:address, src:address) => 
        perform assert false;
  
      method proc move_assign(dst:address, src:address) =>
        perform assert false;
  
    }
  
  } // end class DynamicMetaTyping

