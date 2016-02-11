
SublimeClang2 uses gtk2

Disable Menu accelerators 
-------------------------------
gt2k by default intercepts Atl+V, Alt+X, etc. keyboard presses
and translates them into menu actions using the menu accelerators. This overlaps with keybindings in sublemacspro (such as Alt+V to move up a page). http://www.sublimetext.com/forum/viewtopic.php?f=2&t=8816

```
$ echo "gtk-enable-mnemonics=0" >> ~/.gtkrc-2.0
```

Install package manager
-------------------------------
Open python console
Copy paste the following into it.

```
import urllib2,os,hashlib; h = '2915d1851351e5ee549c20394736b442' + '8bc59f460fa1548d1514676163dafc88'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); os.makedirs( ipp ) if not os.path.exists(ipp) else None; urllib2.install_opener( urllib2.build_opener( urllib2.ProxyHandler()) ); by = urllib2.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); open( os.path.join( ipp, pf), 'wb' ).write(by) if dh == h else None; print('Error validating download (got %s instead of %s), please try manual install' % (dh, h) if dh != h else 'Please restart Sublime Text to finish installation')
```

https://packagecontrol.io/installation#st2

SublimeClang
-------------------------------

This is clang based plugin that provides an "intellisense", i.e. auto completion etc. for C++ code. https://github.com/quarnster/SublimeClang

The problem is that this only works with Python2.6 that can import libcache, which is not available in the python2.6 bundled with SublimeClang.

If your system has Python 2.6

```
$ cd ~/your/sublime/lib
$ ln -s /usr/lib/python2.6 python2.6
```

NOTE: Archlinux (as of 10th Feb 2016 still has Python2.6 in AUR)

Then install the plugin through package manager.

Finally it might complain that libclang.so cannot be loaded. 

```
$ cd .config/sublime-text-2/Packages/SublimeClang/src/
$ mkdir build
$ cd build
$ cmake ..
$ make
```
Now it's likely that the above will also fail because your /usr/bin/python points to python3. Fix that by temporarily pointing
the link to python2.6. Then try the build again, if it succeeds then fix the python link. 

```
$ su 
$ cd /usr/bin/
$ unlink python
$ ln -s python2.6 python
$ DO THE BUILD AGAIN HERE
$ unlink python
$ ln -s python3 python
```


----------------------

The clang on which sublimeclang is based has some obscure bug regarding gcc stdlib
and it complains about max_align_t in cstddef (pulls it into std:: from global scope)
However this problem is only affecting sublimeclang in sublimetext.

https://bugs.archlinux.org/task/40229
http://reviews.llvm.org/rL201729

As a workaround we define a build flag in the sublimeclang settings
and then add a workaround definition inside #if #end scope in a project config.h

```
#  if defined(SUBLIME_CLANG_WORKAROUNDS)
    // clang has a problem with gcc 4.9.0 stdlib and it complains
    // about max_align_t in cstddef (pulls it into std:: from global scope)
    // however the problem is only affecting sublimeclang in SublimeText
    // 
    // https://bugs.archlinux.org/task/40229
    // http://reviews.llvm.org/rL201729
    //
    // As a workaround we define some type with a matching typename
    // in the global namespace. 
    // the macro is enabled in 'pime.sublime-project'    
    typedef int max_align_t;
#  endif
```

I think this bug only happens when using gcc's c++ lib. Looks like adding -stdlib=libc++ 
in sublimeclang options makes this go away. 

NOTE: Make sure that you actually have clang headers (libc++-dev in Ubuntu 14.04) installed!

----------------------

Ubuntu 14.04 LTS appears to have some problems with clang packages (at least on this machine the clang installation was messed up).
clang 3.4 appears to work. Also remember to install the libc++-dev separately for clang stdlib headers!


```
$ sudo apt-get intall clang-3.4 libc++dev
```

