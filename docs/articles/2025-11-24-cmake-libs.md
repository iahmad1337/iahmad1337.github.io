## Key-concepts

- *install prefix* (or just *prefix*) - the base directory for installation of
  your software package. The most notable examples are `/`, `/usr`, `/usr/local` and `~/.local`
- Env variables:
    - *PATH* - paths for binary lookups. Example: `PATH=/usr/sbin:/usr/local/bin:/usr/bin:/bin`
    - *LD_LIBRARY_PATH* - paths for static library lookup during linkage.
      Example: if we want to use `-lcublas` linker flag, we have to do something
      like this `LD_LIBRARY_PATH=/usr/local/cuda-12.9/lib64:`
    - [and others](https://gcc.gnu.org/onlinedocs/gcc/Environment-Variables.html)
- [Filesystem Hierarchy
  Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard) -
  how your typical UNIX filesystem is organized

## General workflow

1. Download source code
2. Install the dependencies and build tools (make, cmake, autotools, etc.)
3. Configure step: most of the software needs to find the toolchain and prepare
   the ground work before any building/installation. Examples: `./configure` in
   autoconf, `cmake $SOURCE` in cmake-based projects, and `./bootstrap.sh` in
   boost
4. Build step: build whatever the targets you need. Examples: `make all`, `ninja all`
5. [Optional] Testing: mostly used by the project developers themselves, but we
   can do it too. Example: `make test` in build dir
6. Install step: after build, we need to place libraries/headers/etc. into
   a common place. The most common way the installation is done is by you
   providing an install prefix (`$prefix` from now on) during configure step and
   then building target `install` - this results in
   `$prefix/{include,lib,bin,...}` being populated with the library artifacts

## Using the installed libraries
In order to build our binary, we have to tell the compiler where to search for
library headers using `-I` flag, and where to search for static/dynamic libs via
`-L` flag or LD_LIBRARY_PATH env variable[^3]

!!! note
    `-I` flags are also crucial for proper IDE functioning: if the IDE doesn't
    know where to look for includes, it won't provide autocompletion, go-to
    declaration, static analysis etc.

Now, how do we determine the compile flags?

### Method #1: brute-force
Suppose we used `/usr/local` as build prefix for boost[^0]. Then, we can
search all of the containing folders for any boost-related libs and pass them to
linker via `-l` flags in hopes that

### Method #2: pkg-config
pkg-config[^2]-enabled libraries basically provide all of the flags needed for
compilation in `$prefix/libs/pkg-config`. All you have to do is call
`pkg-config --libs --cflags <library name>`.

### Method #3: $prefix/lib/cmake/&ast;.cmake
Cmake-based libraries provide these files so you could just do this in your
CMakeLists:
```CMakeLists.txt
find_package(GTest)
...
target_link_libraries(test GTest::gtest_main)
```

But, we can also use them to infer flags for cmake-less compilation

## Uninstalling the libraries
Some popular methods[^1]:

#### Method #1: make uninstall
If we're lucky, the Makefile/ninja.build will have the uninstall target that
reverts all the installation step.

#### Method #2: checkinstall
We can generate a `.deb`/`.rpm` package by calling `checkinstall <install
command>` in build root. We then can just use `dpkg -i package.deb`/`dpkg -t package.deb`.
It would basically work the same as `apt install`/`apt purge`

Random examples on github:

- [install.sh](https://github.com/erichs/bootnukem/blob/f5221f946f9f9306d46a2a6b1e5a340ad82d3ee6/build_deb.sh#L2C35-L4C144)
- [make install](https://github.com/scaryguy/jwthumbs/blob/0ca7b8384716435eecbc1499702770e66278cc94/ffmpeg.sh#L14)
- [make install install-doc](https://github.com/efficient/catbench/blob/4f66541efd8318109c4ac150898d60f023e7aba5/setup/package#L17)


#### Method #3: install_manifest.txt (cmake-based projects)
cmake lists all of the installed files in `$BUILD_ROOT/install_manifest.txt`,
and so uninstalling can be simply done with `xargs rm <install_manifest.txt`[^4]

## Real-world examples

### Case-study #1: abseil-cpp & boost

Default prefix for user installed dirs is `/usr/local`. Installing
[abseil-cpp](https://abseil.io/):
```bash
git clone git@github.com:abseil/abseil-cpp.git
cd abseil-cpp
mkdir build
cd build

cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local

make -j8
sudo make install
```
And boost:
```
git clone git@github.com:boostorg/boost.git
cd boost/
git tag -l
git co boost-1.89.0
git submodule update --init
./bootstrap.sh
sudo ./b2 install -j 10 --prefix=/usr/local
```

Now, let's compile the following code
```c++
#include <absl/strings/str_format.h>
#include <boost/format.hpp>
#include <boost/url/parse.hpp>
#include <iostream>


int main(int argc, char** argv) {
    std::cout << boost::format("Hello from %1%") % argv[0] << std::endl;
    std::cout << absl::StrFormat("Hi from absl %s", argv[0]) << std::endl;
    auto result = boost::urls::parse_uri("http://example.com");
    std::cout << result->scheme() << std::endl;
}
```
the most straightforward way is `g++ -std=c++2b -g main.cc -o a.out`, but we get
this:
```
$ g++ -std=c++2b -g main.cc -o a.out
/usr/bin/ld: /tmp/ccqvfnzh.o: in function `main':
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.cc:10: undefined reference to `boost::urls::parse_uri(boost::core::basic_string_view<char>)'
/usr/bin/ld: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.cc:11: undefined reference to `boost::urls::url_view_base::scheme() const'
/usr/bin/ld: /tmp/ccqvfnzh.o: in function `std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > absl::StrFormat<char*>(absl::str_format_internal::FormatSpecTemplate<(ArgumentToConv<ch
ar*>)()> const&, char* const&)':
/usr/local/include/absl/strings/str_format.h:366: undefined reference to `absl::str_format_internal::FormatPack[abi:cxx11](absl::str_format_internal::UntypedFormatSpecImpl, absl::Span<absl::str_format_internal:
:FormatArgImpl const>)'
/usr/bin/ld: /tmp/ccqvfnzh.o: in function `void absl::str_format_internal::FormatArgImpl::Init<char const*>(char const* const&)':
/usr/local/include/absl/strings/internal/str_format/arg.h:555: undefined reference to `bool absl::str_format_internal::FormatArgImpl::Dispatch<char const*>(absl::str_format_internal::FormatArgImpl::Data, absl::
str_format_internal::FormatConversionSpecImpl, void*)'
collect2: error: ld returned 1 exit status
```

As we can see, only the linker failed; compilation succeeded, because
`/usr/local/include` is looked up by default and headers were found just fine:
```txt
$ echo | gcc -E -Wp,-v -
...
#include <...> search starts here:
 /usr/lib/gcc/x86_64-linux-gnu/12/include
 /usr/local/include                            <-- *****HERE*****
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
...
```

also, there are no `boost::format` link errors, because it's a header-only library.

Let's try and use `pkg-config` autocomplete to see what libraries it can see:
```bash
$ pkg-config <double tab>
Display all 334 possibilities? (y or n)
...
absl_base_internal                           absl_log_globals                             absl_str_format                              ncurses++w
...
```
There's no boost in the output, but we can surely see `absl_str_format`, let's
add its flags to our compile command:
```
$ g++ -std=c++2b -g poly_division.cc `pkg-config --libs absl_str_format` -o a.out
/usr/bin/ld: /tmp/ccNGZ7Vm.o: in function `main':
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.cc:10: undefined reference to `boost::urls::parse_uri(boost::core::basic_string_view<char>)'
/usr/bin/ld: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX.cc:11: undefined reference to `boost::urls::url_view_base::scheme() const'
collect2: error: ld returned 1 exit status
```

Great! Now we need to find the boost libraries to link against. Although boost
is not pkg-config-enabled, we can try to manually infer the needed library:
```
$ find /usr/local/lib -name 'libboost*.a' -and -name '*url*'
/usr/local/lib/libboost_url.a
```
and finally the linker succeeds:
```
$ g++ -std=c++2b -g poly_division.cc `pkg-config --libs absl_str_format` -lboost_url -o a.out
```

We could also try to utilize the
`/usr/local/lib/cmake/boost_url-1.89.0/libboost_url-variant-static.cmake` file
to help us find list of required libs.

### Case-study #2: python3.12 (make + autotools)

[article](../articles/2024-12-04-python.md)

### Case-study #3: musl libc
TODO: a target that uses musl libc - more than just -L/-I

## Miscellany
1. Alternative install prefixes can be useful when installing the dependencies
   (e.g. openssl[^5]), that may conflict with existing libraries on the system
1. You can avoid sudo in `make install`, if you have write rights to prefix.
   For example I often use `~/.local` as a writable install prefix
1. Sometimes between reinstallations a hidden state persists in places not under
   the install prefix. Examples:
    - `~/.cache` and analogues
    - `~/.local/state/nvim`
    - `/etc/config`/`~/.config`
1. A cool trick I used to intercept compiler launches:
   `strace --follow-forks -e trace=process --string-limit=256 make`.
   Although, same could be accomplished with just `make -n`

### Resources
[^3]: [using libraries in gcc](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/developer_guide/gcc-using-libraries#gcc-using-libraries_using-library-gcc)
[^0]: [boost](https://www.boost.org/)
[^1]: [undoing make install](https://stackoverflow.com/a/50898839)
[^2]: [pkg-config main page](https://www.freedesktop.org/wiki/Software/pkg-config/) & [guide](https://people.freedesktop.org/~dbn/pkg-config-guide.html)
[^4]: [install_manifest.txt](https://gitlab.kitware.com/cmake/community/-/wikis/FAQ#can-i-do-make-uninstall-with-cmake)
[^5]: [using alternative install prefix](../articles/2024-12-04-python.md#openssl)
