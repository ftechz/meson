# Copyright © 2017 Dylan Baker
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

project('C and C++ static link test', ['c', 'cpp'])

# Verify that adding link arguments works.
add_global_link_arguments('', language : 'c')
add_project_link_arguments('', language : 'c')

libc = static_library('cfoo', ['foo.c', 'foo.h'])

# Test that linking C libs to external static C++ libs uses the C++ linker
# Since we can't depend on the test system to provide this, we create one
# ourselves at configure time and then 'find' it with cxx.find_library().
cxx = meson.get_compiler('cpp')

if cxx.get_id() == 'msvc'
  compile_cmd = ['/c', '@INPUT@', '/Fo@OUTPUT@']
  stlib_cmd = ['lib', '/OUT:@OUTPUT@', '@INPUT@']
else
  compile_cmd = ['-c', '-fPIC', '@INPUT@', '-o', '@OUTPUT@']
  stlib_cmd = ['ar', 'csr', '@OUTPUT@', '@INPUT@']
endif

foo_cpp_o = configure_file(
  input : 'foo.cpp',
  output : 'foo.cpp.o',
  command : cxx.cmd_array() + compile_cmd)

configure_file(
  input : foo_cpp_o,
  output : 'libstcppext.a',
  command : stlib_cmd)

libstcppext = cxx.find_library('stcppext', dirs : meson.current_build_dir())
lib_type_name = libstcppext.type_name()
assert(lib_type_name == 'library', 'type name is ' + lib_type_name)

libfooext = shared_library(
  'fooext',
  ['foobar.c', 'foobar.h'],
  link_with : libc,
  dependencies : libstcppext,
)

# Test that linking C libs to internal static C++ libs uses the C++ linker
libcpp = static_library('cppfoo', ['foo.cpp', 'foo.hpp'])

libfoo = shared_library(
  'foo',
  ['foobar.c', 'foobar.h'],
  link_with : [libc, libcpp],
)

# Test that link_whole is also honored
#
# VS2010 lacks the /WHOLEARCHIVE option that later versions of MSVC support, so
# don't run this tests on that backend.
if not (cxx.get_id() == 'msvc' and cxx.version().version_compare('<19'))
  libfoowhole = shared_library(
    'foowhole',
    ['foobar.c', 'foobar.h'],
    link_whole : [libc, libcpp],
  )
endif

# Test sublinking (linking C and C++, then linking that to C)
libfoo_static = static_library(
  'foo_static',
  ['foobar.c', 'foobar.h'],
  link_with : [libc, libcpp],
)

libsub = shared_library(
  'sub',
  ['sub.c', 'sub.h'],
  link_with : libfoo_static,
)

if not (cxx.get_id() == 'msvc' and cxx.version().version_compare('<19'))
  libsubwhole = shared_library(
    'subwhole',
    ['sub.c', 'sub.h'],
    link_whole : libfoo_static,
  )
endif

# Test that it really is recursive
libsub_static = static_library(
  'sub_static',
  ['sub.c', 'sub.h'],
  link_with : libfoo_static,
)

libsubsub = shared_library(
  'subsub',
  ['dummy.c'],
  link_with : libsub_static,
)
