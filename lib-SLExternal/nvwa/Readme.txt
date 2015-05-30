                         === Stones of Nvwa ===

The Name

  Nvwa ("v" is pronounced like the French "u"), is one of the most
  ancient Chinese goddesses.  She was said to have created human beings,
  and, when the evil god Gong-gong crashed himself upon one of the
  pillars that support the sky and made a hole in it, she mended the sky
  with five-coloured stones.

  I thought of the name Nvwa by analogy with Loki.  Since it is so small
  a project and it contains utilities instead of a complete framework, I
  think "stones" a good metaphor.


Code Organization

  Nvwa versions prior to 1.0 did not use a namespace.  It looked to me
  overkill to use a namespace in this small project.  However, having
  had more project experience and seen more good examples, I have
  changed my mind.  All nvwa functions and global variables are now
  inside the namespace "nvwa".

  This said, I do not want to break backward compatibility abruptly.
  First, people are still able to disable the namespace by defining the
  macro NVWA_USE_NAMESPACE to 0 (not really recommended, though).
  Second, I still use the quotation mark (") to include header files
  internally so that old users do not have to change the include
  directory (again, not recommended and intended to help the transition
  only).  Third, I have not changed most of the macro names -- that
  would break existing code and could do more harm than good now.  In
  addition, I am not a fan of macro prefixes anyway.

  Overall, I am quite happy with the changes: some code is simplified,
  and in many cases I do not have to use the uglified names to avoid
  potential name conflicts.  The changes also do not mean anything to
  new users.  You just need to follow the modern C++ way.  You should
  add the root directory of this project to your include directory
  (there should be a "nvwa" directory inside it with the source files).
  To include the header file, use "#include <nvwa/...>".


Contents

  A brief introduction follows.  Check the Doxygen documentation for
  more (technical) details.

  * boolarray.cpp
  * boolarray.h

   A replacement of std::vector<bool>.  I wrote it before I knew of
   vector<bool>, or I might not have written it at all.  However, it is
   faster than any vector<bool> implementation I know, and it has
   members like at, set, reset, flip, and count.  I myself find "count"
   very useful.

  * c++11.h

   Detection macros for certain features of C++11 that might be of
   interest to code/library writers.  I have used existing online
   resources, and I have tested them on popular platforms and compilers
   that I have access to (Windows, OS X, and Linux; MSVC, Clang, and
   GCC), but of course not all versions or all combinations.  Patches
   are welcome for corrections and amendments.

  * class_level_lock.h

   The Loki ClassLevelLockable adapted to use the fast_mutex layer.  One
   minor divergence from Loki is that the template has an additional
   template parameter _RealLock to boost the performance in non-locking
   scenarios.  Unless HAVE_CLASS_TEMPLATE_PARTIAL_SPECIALIZATION is
   explicitly defined to 0, this is enabled automatically.  Cf.
   object_level_lock.h.

  * cont_ptr_utils.h

   Utility functors for containers of pointers adapted from Scott Meyers'
   Effective STL.

  * debug_new.cpp
  * debug_new.h

   A cross-platform, thread-safe memory leak detector.  It is a
   light-weight one designed only to catch unmatched pairs of
   new/delete.  I know there are already many existing memory leak
   detectors, but as far as I know, free solutions are generally slow,
   memory-consuming, and quite complicated to use.  This solution is
   very easy to use, and has very low space/time overheads.  Just link
   in debug_new.cpp for leakage report, and include debug_new.h for
   additional file/line information.  It will automatically switch on
   multi-threading when the appropriate option of a recognized compiler
   is specified.  Check fast_mutex.h for more threading details.

   Special support for gcc/binutils has been added to debug_new lately.
   Even if the header file debug_new.h is not included, or
   _DEBUG_NEW_REDEFINE_NEW is defined to 0 when it is included,
   file/line information can be displayed if debugging symbols are
   present in the executable, since debug_new stores the caller
   addresses of memory allocation/deallocation routines and they will be
   converted with addr2line (or atos on a Mac) on the fly.  This makes
   memory tracing much easier.

   NOTE for Mac OS X users: If you use OS X 10.7 or later, you may need
   to add "-Wl,-no_pie" at the end of the command line.  By default OS X
   will now create position-independent executables, and the OS will
   load a PIE at a random address each time it is executed, making it
   difficult to convert the address to symbols.  (I Googled a while for
   a way to get the base address of a process, but could not find
   working code.  If you know how to do it right, drop me a message.)

   With an idea from Greg Herlihy's post in comp.lang.c++.moderated, the
   implementation was much improved in 2007.  The most significant
   result is that placement new can be used with debug_new now!  Full
   support for new(std::nothrow) is provided, with its null-returning
   error semantics (by default).  Memory corruption will be checked on
   freeing the pointers and checking the leaks, and a new function
   check_mem_corruption is added for your on-demand use in debugging.
   You may also want to define _DEBUG_NEW_TAILCHECK to something like 4
   for past-end memory corruption check, which is off by default to
   ensure performance is not affected.

   An article on its design and implementation is available at

   http://nvwa.sourceforge.net/article/debug_new.htm

  * fast_mutex.h

   The threading transparent layer simulating a non-recursive mutex.  It
   supports POSIX threads and Win32 threads currently, as well as a
   no-threads mode.  Unlike Loki and some other libraries, threading
   mode is not to be specified in code, but detected from the
   environment.  It will automatically switch on multi-threading when
   the "-MT"/"-MD" option of MSVC, the "-mthreads" option of MinGW GCC,
   or the "-pthread" option of GCC under POSIX environments, is used.
   One advantage of the current implementation is that the construction
   and destruction of a static object using a static fast_mutex not yet
   constructed or already destroyed are allowed to work (with
   lock/unlock operations ignored), and there are re-entry checks for
   lock/unlock operations when the preprocessing symbol _DEBUG is
   defined.

  * fc_queue.h

   A queue that has a fixed capacity (maximum number of allowed items).
   All memory is pre-allocated when the queue is initialized, so memory
   consumption is well controlled.  When the queue is full, inserting a
   new item will discard the oldest item in the queue.  Care is taken to
   ensure this class template provides the strong exception safety
   guarantee.

   This class template is a good exercise for me to make a STL-type
   container.  Depending on your needs, you may find circular_buffer in
   Boost more useful, which provides similar functionalities and a
   richer API.  However, the design philosophies behind the two
   implementations are quite different.  fc_queue is modelled closely on
   std::queue, and I optimize mainly for a lockless producer-consumer
   pattern.

  * fixed_mem_pool.h

   A memory pool implementation that requires initialization (allocates
   a fixed-size chunk) prior to its use.  It is simple and makes no
   memory fragmentation, but the memory pool size cannot be changed
   after initialization.  Macros are provided to easily make a class use
   pooled new/delete.

  * mem_pool_base.cpp
  * mem_pool_base.h

   A class solely to be inherited by memory pool implementations.  It is
   used by static_mem_pool and fixed_mem_pool.

  * object_level_lock.h

   The Loki ObjectLevelLockable adapted to use the fast_mutex layer.
   The member function get_locked_object does not exist in Loki, but is
   also taken from Mr Alexandrescu's article.  Cf. class_level_lock.h.

  * pctimer.h

   A function to get a high-resolution timer for Win32/Cygwin/Unix.  It
   is quite useful for measurement and optimization.

  * set_assign.h

   Utility routines to make up for the fact that STL only has set_union
   (+) and set_difference (-) algorithms but no corresponding += and -=
   operations available.

  * static_mem_pool.cpp
  * static_mem_pool.h

   A memory pool implementation to pool memory blocks according to
   compile-time block sizes.  Macros are provided to easily make a class
   use pooled new/delete.

   An article on its design and implementation is available at

   http://nvwa.sourceforge.net/article/static_mem_pool.htm

  * type_traits.h

   This file was separated from fc_queue.h.  Type traits was not
   standardized until C++11, and, in addition, there were late changes
   in the trait names that did not go into even the 2012 releases of
   MSVC and GCC.  So I made a nvwa::is_trivially_destructible that can
   work across main compilers.  I may add more traits later as need be.


$Id: README,v 1.20 2013/10/02 08:11:59 adah Exp $

vim:autoindent:expandtab:formatoptions=tcqlm:textwidth=72:
