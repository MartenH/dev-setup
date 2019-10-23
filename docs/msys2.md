# MSYS2





## ssh 

openssh will always use /home an not $HOME or anything else.

```bash
echo "C:\Users /home" >> /etc/fstab
```

See answer from Alexx83 [here](https://sourceforge.net/p/msys2/tickets/111/) .

# msys, mingw, msys2, cygwin, etc??

First have a look at the [wiki](https://github.com/msys2/msys2/wiki/MSYS2-introduction)

A more detailed answer is from Matthieu Vachon [here] (https://sourceforge.net/p/msys2/discussion/general/thread/dcf8f4d3/). 

If that thread is ever lost it's here (Sorry for pure copy here):

First of all, it's good to understand distinctions between MSYS2 and MINGW.

### MSYS2
MSYS2 is based on Cygwin and knows how to understand POSIX conventions like paths (`/usr/bin/`, `/etc`) as well as special devices like `/dev/null`, `/dev/clipboard`, etc and many other things. MSYS2 comes with a GCC toolchain who will produce native binaries that are always tied to `msys-2.0.dll`. It's that DLL that does all the magic of interoperability between POSIX conventions and Windows. It's because of it that it's possible to use a Unix like path /home/joe/ and pass it to a Windows and it still works.

Indeed, (I'm not sure of the exact details) but when when `msys-2.0.dll`
sees you are calling a native Windows executables (one that does not
depends on `msys-2.0.dll`), it will convert the path `/home/joe/` to a
Windows path like C:\Users\joe that the native Windows program receives
and knows how to understand. Without this translation, the Windows program
would receive `/home/joe/` and wouldn't understand how to treat it
correctly. All this paths translations applies also to environment
variables.

This process (and all the POSIX emulation layer) incurs a performance
penalty that can be significant for heavy file-centric software. Git is a
good example and that's why the MSYS2 git package (msys/git) is slower
than a native git client like the one provided by Git For Windows
(git-for-windows/mingw-w64-x86_64-git). That's why a native program (read
a MINGW one) is usually "better" than an MSYS2 one. Don't be afraid of
using pure MSYS2 program, in most cases (I would say 90%), the difference
is not significant because bottleneck is not the filesystem (CPU, network,
other).

That's why you will often read that MSYS2 is only a way to make it easier
to build and deploy MINGW native executables. In my personal opinion and
the way I use it, it's much much more than that as I use and build little
MINGW tools. MSYS2 is a full subsystem dealing with way more than just a
toolchain as it deals with filesystem, users, permissions (to an extent)
and much more. I use it mainly because it has the same toolset as the one
I'm used on Linux (console-based of course :)).

Because MSYS2 understands POSIX and has a pretty good emulation layer, it
is often very easy to port a package originally built on Linux to it,
because almost no code work is needed.

### MINGW
MINGW (either 32 or 64) is a set of toolchain to build native Windows
program. Those program do not depends on `msys-2.0.dll` and depends rather
on MSVC runtime. Those don't understand POSIX paths, so if you pass
`/home/joe` outside of an MSYS2 shells (from pure cmd for example), it will
not understand it. Indeed, in pure cmd (and via many interactions from
other programs), no translation of paths is performed.

MINGW toolchains are based on mingw-w64 which
provides the ground work to compile them. They are available in pacman for
installation and they also contains GCC et related tools (binutils, ld).
When invoked, GCC produces pure native executable. So, when working with a
MINGW program, remember that it's like a native Window program that knows
nothing about MSYS2.

MSYS2 happens to actual provides those toolchains directly to you
pre-compiled via pacman. Note however, that it's possible to download the
mingw-w64 toolchains in standalone, put them somewhere and invoke them
afterward in MSYS2 (by proprely configuring PATH and other variables
yourself). They can also be invoked via cmd directly because they do not
depend on MSYS2 at all.

Shells
When you start any of the MSYS2 shells, they all start an MSYS2 bash
program. Since it's a MSYS2 bash, it understands Unix paths and will do
translation when calling native Windows executables.

That being said, what is the difference between all the shells? It all
about the tweaking some environment variables to change default programs
picked when calling them. It's mainly about the PATH environment variable
but it also involves MSYSTEM PKG_CONFIG_PATH ACLOCAL_PATH MANPATH like Ray
said earlier. Checkout /etc/profile around line 28 to see the conditional
that sets all this.

Let's assume you have installed three toolchains:
* MSYS2 (package group msys2-devel) to compile MSYS2 tools which
installs gcc in `/usr/bin`.
* MINGW32 (package group mingw-w64-i686-toolchain) to compile Windows
native 32 bits executable which installs gcc in /mingw32/bin.
* MINGW64 (package group mingw-w64-x86_64-toolchain) to compile
Windows native 64 bits executable which installs gcc in /mingw64/bin.

For msys2_shell, PATH is roughly only `/usr/local/bin:/usr/bin:/bin`. This
means that when running gcc, you will get /usr/bin/gcc.

For mingw32_shell, PATH is roughly
`/mingw32/bin:/usr/local/bin:/usr/bin:/bin`. This means that now, running
gcc you will get instead `/mingw32/bin/gcc`.

For mingw64_shell, PATH is roughly
`/mingw64/bin:/usr/local/bin:/usr/bin:/bin` and as you have guessed,
running gcc you will get instead `/mingw64/bin/gcc`.

This means that all three shells have the same capabilities, but because of
the different orders, tools that will be picked are different. So, if you
want to compile something, use the appropriate shell for what you want to
do (MSYS2 program, Windows native 32 bits or Windows native 64 bits).

If you are not compiling anything, then it depends. Usually, msys2_shell is
recommended. However, on a bare installation, /mingw64/bin is not added
anywhere when using msys2_shell, problem it can causes is that if you had
installed mingw64 packages, they are not available in the shell. In my
case, I have tweaked by .bash_profile to correctly add /mingw64/bin
after PATH (like `/usr/local/bin:/usr/bin:/bin:/mingw64/bin`). This
always favor MSYS2 tools but will pick /mingw64/bin when they don't exist
in MSYS2 `/usr/bin`.

Dual Packages
Then, it all comes to the problem that sometimes, programs exist in both
variant, i.e. an MSYS2 version and a MINGW version. That the case with
curl (and others). There is two packages for it (three if you include the
32 bits version):

MSYS2 package is `msys/curl` while mingw64 package is
mingw64/mingw-w64-x86_64-curl.
When running pacman curl in any of the shells, it will always pick the
msys one by default. Now, let's say you installed both of them:

`pacman -S msys/curl # Executable at /usr/bin/curl`
`pacman -S mingw64/mingw-w64-x86_64-curl # Executable at
/mingw64/bin/curl`
Now, depending on the shell you start with, the curl that will be picked
will be different. And here comes the advise to use msys2 shell for
pacman tasks. If a package does a post-install and uses curl, it might
get the wrong one depending on the shell you used. It can be used with
mingw64_shell, just need more care on some (I think rare) cases. Indeed,
even using the mingw64 could yield the correct results, thanks to path
conversion, so, it's hard to come with a sound real existing example.

I think sed was a wrong example as I see only one package for it
pacsearch sed. I might be wrong but potential problems still stand for
other packages.

### Packages
Whether you want to package a MSYS2 program or MINGW one, both uses
PKGBUILD. It's just the toolchain used to do the steps that are different.
I think (someone would need to confirm), that makepkg-mingw takes cares
of setuping the PATH and all other variables. So even in msys2 shell,
it would pick the correct toolchain. I really don't know the pros and cons
of using either the shells or the makepkg-mingw tool however.

It's a pretty long post now. Don't hesitate to break it down and ask
specific clarifications about portion of it. I will try my best to help you
out.

Sorry if I explain stuff you already knew.

Hope if helped you clarify all this. MSYS2 and related tooling like MINGW.
It's a big beast but understanding every parts will make you usage more
pleasant ... and will help you further understand other problems you might
have and also debug problems you got with tooling interacting with MSYS2
subsystem and MINGW programs.

Regard,
Matt












