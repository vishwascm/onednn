Transitioning from Intel MKL-DNN to oneDNN {#dev_guide_transition_to_dnnl}
==========================================================================

To simplify library naming and differentiate it from Intel MKL, starting with
version 1.1 the library name is changed to
**Deep Neural Network Library (DNNL)**.

@note The subsequent library name change to
oneAPI Deep Neural Network Library (oneDNN) does not impact API, environment
variables, or build options.

There are some incompatibilities, described in the
[Broken Compatibility with Intel MKL-DNN](@ref dg_mtdt_s2) section below.

## Summary of Changes

In short, the migration can be as simple as just replacing all
`MKLDNN/mkldnn` substrings with `DNNL/dnnl`.

### Source Code Changes

All headers, functions, types, and namespaces are renamed by replacing `mkldnn`
with `dnnl`. The macros with `MKLDNN` are replaced with `DNNL` counterparts.

An example of code with Intel MKL-DNN v1.0:
~~~ cpp
#include "mkldnn.hpp"

using namespace mkldnn;

mkldnn_memory_desc_t md;
if (md.format_kind == mkldnn_blocked) {}
conv.exec(stream, {{MKLDNN_ARGS_SRC, src}, ...});
~~~

The updated example with DNNL v1.1:
~~~ cpp
#include "dnnl.hpp"

using namespace dnnl;

dnnl_memory_desc_t md;
if (md.format_kind == dnnl_blocked) {}
conv.exec(stream, {{DNNL_ARGS_SRC, src}, ...});
~~~

To API compatibility with Intel MKL-DNN is based on
``include/mkldnn_dnnl_mangling.h`` header file that maps all Intel MKL-DNN
symbols to DNNL ones using C preprocessor:

~~~ cpp
// ...
#define mkldnn_memory_desc_t           dnnl_memory_desc_t
#define mkldnn_memory_desc_init_by_tag dnnl_memory_desc_init_by_tag
// ...
~~~

This file is included to every former Intel MKL-DNN header files
(for instance, see `mkldnn.h`) along with the DNNL counterpart.

### Build Process

The changes to the build options are similar to the ones in the source code.
All the options and namespaces with `MKLDNN` are replaced with `DNNL`:

| Intel MKL-DNN                 | DNNL                        |
|:------------------------------|:----------------------------|
| MKLDNN (namespace)            | DNNL (namespace)            |
| MKLDNN_ARCH_OPT_FLAGS         | DNNL_ARCH_OPT_FLAGS         |
| MKLDNN_BUILD_EXAMPLES         | DNNL_BUILD_EXAMPLES         |
| MKLDNN_BUILD_FOR_CI           | DNNL_BUILD_FOR_CI           |
| MKLDNN_BUILD_TESTS            | DNNL_BUILD_TESTS            |
| MKLDNN_CPU_RUNTIME            | DNNL_CPU_RUNTIME            |
| MKLDNN_ENABLE_CONCURRENT_EXEC | DNNL_ENABLE_CONCURRENT_EXEC |
| MKLDNN_ENABLE_JIT_PROFILING   | DNNL_ENABLE_JIT_PROFILING   |
| MKLDNN_GPU_BACKEND            | DNNL_GPU_BACKEND            |
| MKLDNN_GPU_RUNTIME            | DNNL_GPU_RUNTIME            |
| MKLDNN_INSTALL_MODE           | DNNL_INSTALL_MODE           |
| MKLDNN_LIBRARY_TYPE           | DNNL_LIBRARY_TYPE           |
| MKLDNN_THREADING              | DNNL_THREADING              |
| MKLDNN_USE_CLANG_SANITIZER    | DNNL_USE_CLANG_SANITIZER    |
| MKLDNN_VERBOSE                | DNNL_VERBOSE                |
| MKLDNN_WERROR                 | DNNL_WERROR                 |

You must switch to DNNL build options as well:

~~~ sh
# Through find package
find_package(dnnl DNNL CONFIG REQUIRED)
target_link_libraries(project_app DNNL::dnnl)

# Or direct sub-project inclusion
add_subdirectory(${DNNL_DIR} DNNL)
include_directories(${DNNL_DIR}/include)
target_link_libraries(project_app dnnl)
~~~

@anchor dg_mtdt_s2
## Broken Compatibility with Intel MKL-DNN

Unfortunately, full compatibility after renaming is not implemented.
DNNL is **not** compatible with Intel MKL-DNN in the following things:
- ABI: An application or a library built with Intel MKL-DNN cannot switch on
  using DNNL without recompiling.
- Microsoft\* Visual Studio Solution files that are generated by cmake will
  be based on DNNL name only
  (e.g. `MKLDNN.sln` becomes `DNNL.sln`, and the former is no more generated).

## Information for Developers

The implementation of renaming (several patches and scripts that rename the
library) can be found
[here](https://github.com/emfomenk/intel-mkldnn-rebranding).