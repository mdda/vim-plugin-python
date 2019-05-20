## vim plugin template (in python)

NB: This repo is solely trying to recreate the code built in [this excellent blog post](http://candidtim.github.io/vim/2017/08/11/write-vim-plugin-in-python.html)

### Writing Vim plugin in Python


There are many ways to create a Vim plugin. 

Classic one - use VimL. Or you can also use Lua. Or Python. This particular guide uses Python, and Python may be a great language to write a plugin for Vim because:

*  it is “natively” supported by Vim
*  you most likely already know it, in contrast to VimL
*  and it simple; you know, in contract to VimL


Certainly, a plugin written in Python will only run in Vim compiled with Python support. 
Vim’s default distribution is compiled with Python support, and nowadays finding the opposite is actually harder. 
There is also a number of widely used Vim plugins written in Python and you shouldn’t worry about Python support - it is not going anywhere.

To make sure that your Vim has Python support, run `vim --version`, 
and look for a line marked `+python` or `+python3`. 
Note that all code below is designed for Python 2 (`+python`) which is how Vim is distributed by default. 
If your Vim uses Python 3 (`+python3`) - you will need to update the source code accordingly.



### Principles and minimal template

Vim plugins actually have to be written in VimL and not in Python. 
Good news is that Vim plugin can execute arbitrary Python scripts from withing VimL code. 
With this knowledge, the basic idea of the plugin is to:

*  create a wrapper script in VimL
*  which will declare Vim commands
*  and import and run Python code
*  while latter implements those commands

Before going into Python code, let’s prepare the basic project structure, 
development environment, and ensure that our plugin is ready for plugin managers.


### Plugin structure

If we want our plugin to work with Vim plugin managers, like 
pathogen, Vundle and many others, it needs to follow some basic structure:

```
vim-plugin-python/
├── doc/
│   └── vim-plugin-python.doc
└── plugin/
    └── vim-plugin-python.vim
```

This is self-explanatory.  *And is how this repo is laid out.*

But, for sanity's sake, it's probably best to rename the files in this template from `vim-plugin-python` to `yourpluginname`.

```
yourpluginname/
├── doc/
│   └── yourpluginname.doc
└── plugin/
    └── yourpluginname.vim
```


It is a good idea to provide an integrated documentation for a plugin, 
and we will address this later on. 
If we are to publish the plugin, say, on GitHub, 
it makes sense to also add two more files:

```
yourpluginname/
├── ...
├── LICENSE
└── README
```

Once our project structure is ready, let’s try and install it.


### Development process and our first Vim command 

Let's configure the development environment at once, 
so that we can test and run the plugin in a Vim instance regularly. 
How this set-up is made depends largely on the plugin manager you use with Vim.

Some plugin managers require all plugins to be installed under same root directory, 
which for most users is `~/.vim/bundle`. 
If you are concerned, and don’t want to change your plugins root directory, 
you can create a symbolic link from your source code (which is also convenient during development):

```
$ cd ~/.vim/bundle
$ ln -s ~/your-src-directory/yourpluginname yourpluginname
```

Check you Vim's plugin manager documentation on how to declare and load the plugin. 
For example, I use `Vundle`, my plugin source code is in `~/src/sampleplugin`, 
and thus I have following in my `~/.vimrc`:

```
Plugin 'file:///home/candidtim/src/sampleplugin'
```

Of course, with `Vim 8.0` and after, there's a built-in package manager, so if you 
create the necessary directory (here `devel` is just an arbitrary choice), you can symlink from there :

```
pushd . 
mkdir -p ~/.vim/pack/devel/start
cd  ~/.vim/pack/devel/start
ln -s ~/your-src-directory/yourpluginname .
popd
```

Now, let's make sure this actually works. Let’s add following content to `sampleplugin.vim` 
(this is written in Vim's custom built-in language `VimL`) :

```
echo "It worked!"
```

And start new Vim instance where we will test the plugin. 
Upon startup you should see "It worked!" printed out in the terminal. 

>   If at this point it doesn't work, try to load the plugin manually. 
>   For this, execute following command from Vim: `:source ~/.vim/.vim/pack/devel/start/yourpluginname/plugin/yourpluginname.vim`. 
>   Now, if this finally works, 
>   it means that your plugin manager doesn’t load the plugin automatically on Vim startup - 
>   refer to your plugin manager documentation to find out how to configure it correctly. 
>   If however this doesn't work either - Vim should normally print out an error message, 
>   which should give you a better idea. 
>   Most likely you need to check that file actually exists and symbolic link works as expected, 
>   and that file content (syntax) is correct.

All set! Let's write some Python!


### Use Python in Vim plugin

As noted above, the idea now is to execute Python code from `VimL`. 
`VimL` exposes specific syntax for this. 

Let's change our plugin source (in `plugin/yourpluginname.vim`) to the following:

```
python << EOF
print "Hello from Vim's Python!"
EOF
```

(Re-)start test Vim instance and you should see the new message.


### Actually, Python3 ...

So that we can remain sane (and, of course, providing that your locally installed vim supports it),
let's instead 'boot' vim with Python3 (it seems to load in the first python version mentioned to it, 
and can't load both python2 and python3 at the same time).  
My opinionated choice here is Python3 for the plugin template, so put (in `plugin/yourpluginname.vim`) :

```
python3 << EOF
print("Hello from Vim's Python3!")
EOF
```

### Refining the loading process


Now, I don’t mind writing few simple commands inline like this, 
but our actual goal is to make Python code to live in Python source files, 
and `VimL` code in `.vim` source files. 
So, let’s actually make Vim "import" our code from Python source files. 
Change the code to:

```
let s:plugin_root_dir = fnamemodify(resolve(expand('<sfile>:p')), ':h')

python3 << EOF
import sys
from os.path import normpath, join
import vim
plugin_root_dir = vim.eval('s:plugin_root_dir')
python_root_dir = normpath(join(plugin_root_dir, '..', 'python'))
sys.path.insert(0, python_root_dir)
import plugin
EOF
```

Vim doesn’t know where your Python plugin code lives, so if we are to import it, 
we need to add its root directory to sys.path in the interpreter running inside Vim. 
For this:

*  We first save plugin’s directory path into a local variable in plugin’s Vim script
*  then acces its value from within Python script
*  use it to build the path to the directory where our Python code lives
*  and finally add it to sys.path
*  so that we can now import our Python module

To extract value from Vim’s `plugin_root_dir` variable we use the `vim` Python module. 
This is available inside Vim and provides an interface to the Vim environment. 
We will revist this in detail later.

Now, we'll actually add this Python code we talk about. 
Let's add into a file `./python/plugin.py`):

```
print("Hello from Python source code in plugin.py")
```

Restart test Vim instance, see the new message, all done!

(This is already done in this template repo).


### Declare `vim` commands and implement them in Python

Now, you likely want to add some commands to the Plugin, 
or it risks to not to be very useful. 
Let's implement a simple command which would print out the country you are in, based on your IP. 

Add this to your 'plain python file' `./python/plugin.py` :

```
import urllib, urllib.request
import json

try:
  import vim
except:
  print("No vim module available outside vim")
  pass

def _get(url):
  return urllib.request.urlopen(url, None, 5).read().strip().decode()

def _get_country():
  try:
    ip = _get('http://ipinfo.io/ip')
    json_location_data = _get('http://api.ip2country.info/ip?%s' % ip)
    location_data = json.loads(json_location_data)
    return location_data['countryName']
  except Exception as e:
    print('Error in sample plugin (%s)' % (e.msg,))

def print_country():
  print('You seem to be in %s' % (_get_country(),))
```

Now, the beauty of this implementation is in that it is plain Python code. 
You can test and debug it outside Vim with whatever tools you typically use. 
You can write Python unit tests and execute code from Python REPL, for example:

```
$ cd ~/your-src-directory/yourpluginname/python/
$ python
>>> import plugin
No vim module available outside vim
>>> plugin.print_country()
You seem to be in Singapore
```


### Calling Python from Vim

Now, if we want to call it from Vim, some `VimL` is necessary again. 
Let's declare a Vim function which will call our Python function. 
Add this to the end of `yourpluginname.vim` file:

```
function! PrintCountry()
  python3 print_country()
endfunction
```

Restart a test Vim instance, and type: `:call PrintCountry()` 

