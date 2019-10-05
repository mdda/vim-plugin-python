## vim plugin template (in python)

NB: This repo is solely trying to recreate the code built in [this excellent blog post](http://candidtim.github.io/vim/2017/08/11/write-vim-plugin-in-python.html)

The original author can be contacted via [their GitHub repo](https://github.com/candidtim)

*The text on this page has been fixed-up a little from the original, and the code has been confirmed to work (on my Fedora 30 machine).*


### Writing a Vim plugin in Python

There are many ways to create a Vim plugin. 

Classic one - use `VimL`. Or you can also use `Lua`. Or `Python`. 
This particular guide uses `Python` (specifically `python3`), 
and `Python` may be a great language to write a plugin for Vim because:

*  it is "natively" supported by Vim
*  you most likely already know it, in contrast to `VimL`
*  and it simple; you know, in contract to `VimL`

Certainly, a plugin written in Python will only run in Vim compiled with Python support. 
Vim's default distribution is compiled with Python support, and nowadays finding the opposite is actually harder. 
There is also a number of widely used Vim plugins written in Python and you shouldn’t worry about Python support - it is not going anywhere.

To make sure that your Vim has Python support, run `vim --version`, 
and look for a line marked `+python` or `+python3`. 


### Principles and minimal template

Vim plugins actually have to be written in `VimL` and not in Python. 

The good news is that Vim plugins can execute arbitrary Python scripts from within `VimL` code. 
With this in mind, the basic idea of the plugin is to:

*  create a wrapper script in `VimL`
*  which will declare Vim commands
*  and import and run Python code
*  while latter implements those commands

Before going into Python code, let's prepare the basic project structure, 
development environment, and ensure that our plugin is ready for plugin managers.


### Plugin structure

If we want our plugin to work with Vim plugin managers, 
like `pathogen`, `Vundle` and many others, 
it needs to follow some basic structure:

```
vim-plugin-python/
├── doc/
│   └── vim-plugin-python.doc
└── plugin/
    └── vim-plugin-python.vim
```

This is self-explanatory.  *And is how this repo is laid out.*
But, for sanity's sake, it's probably best to rename the files in this template from `vim-plugin-python` to `yourpluginname` 
(or whatever, but *be consistent*).

```
yourpluginname/
├── doc/
│   └── yourpluginname.doc
└── plugin/
    └── yourpluginname.vim
```

It is a good idea to provide a integrated documentation for your plugin, 
and we will address this later on (maybe). 

If we are to publish the plugin, say, on GitHub, 
it makes sense to also add two more files:

```
yourpluginname/
├── ...
├── LICENSE
└── README.md
```

Once our project structure is ready, let’s try and install it.


### Development process and our first Vim command 

Let's configure the development environment as a first step, 
so that we can test and run the plugin in a Vim instance regularly. 
How this set-up is done depends largely on the plugin manager you use with Vim.

Some plugin managers require all plugins to be installed under same root directory, 
which for most users is `~/.vim/bundle`. 
If you are concerned, and don't want to change your plugins root directory, 
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

Now, let's make sure this actually works : Add following content to `sampleplugin.vim` 
(this is written in Vim's custom built-in language `VimL`) :

```
echo "It worked!"
```

And start a new Vim instance (in a new terminal window) where we will test the plugin. 
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
let's instead 'boot' Vim with Python3 (it seems to load in the first python version mentioned to it, 
and can't load both python2 and python3 at the same time).  
The opinionated choice made in this repo is to use Python3 for the plugin template, so put (in `plugin/yourpluginname.vim`) :

```
python3 << EOF
print("Hello from Vim's Python3!")
EOF
```

### Refining the loading process


Now, writing few simple commands inline like this should be fine, 
however our actual goal is to make a clean plugin system, 
where Python code that lives in Python source files, and `VimL` code in `.vim` source files. 
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

Now, let's add some commands to the Plugin.

As an example, let's implement a simple command which would print out the country you are in, 
based on your local IP. 

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
It can be tested and debugged outside Vim with whatever tools you typically use. 
And you can write Python unit tests and execute code from Python REPL:

```
$ cd ~/your-src-directory/yourpluginname/python/
$ python
>>> import plugin
No vim module available outside vim
>>> plugin.print_country()
You seem to be in Singapore
```


### Calling Python from Vim

Now, if we want to call the python commands from Vim, 
some more `VimL` is necessary. 
Let's declare a Vim function which will call our Python function : 
Add this to the end of `yourpluginname.vim` file:

```
function! PrintCountry()
  python3 print_country()
endfunction
```

Restart a test Vim instance, and type: `:call PrintCountry()` 


### Calling Python from Vim (streamlined)

However, it is not very convenient to use the `:call` syntax. 
Typically, Vim plugins provide commands instead.  
To do this, add the following after the function declaration:

```
command! -nargs=0 PrintCountry call PrintCountry()
```

Launching Vim again will enable you to type `:PrintCountry` 
and have it print the same country. 


### Accessing Vim functionality from Python plugin

The plugin (as presented above) is quite limited so far: 
it only spits some text to Vim message area, but doesn’t do a lot otherwise. 
If we want to do more interesting thins - we need to import the `vim` module, 
which provides a Python interface to a lot of Vim functinality.

For a start, the `vim` module can simply evaluate expressions writtern in `VimL` 
(This is what we previously did to extract a value of a variable declared in `VimL`) :

```
plugin_root_dir = vim.eval('s:plugin_root_dir')
```

The `eval` function can evalaute any `VimL` expression and is certainly 
not limited to accessing `VimL` variables. 
However, it is often more convenient to use other interfaces within the `vim` module instead of `eval`.

For example, you can access and modify text in current buffer :

```
vim.current.buffer.append('I was added by a Python plugin!')
```

Continuing from our example above, let’s implement another command, `InsertCountry`, 
which inserts the name of the country your machine is in at current cursor position. 
Here is the Python code to add:

```
def insert_country():
  row, col = vim.current.window.cursor
  current_line = vim.current.buffer[row-1]
  new_line = current_line[:col] + _get_country() + current_line[col:]
  vim.current.buffer[row-1] = new_line
```

And, just as before, let's add the corresponding `VimL` function and command:

```
function! InsertCountry()
  python3 plugin.insert_country()
endfunction

command! -nargs=0 InsertCountry call InsertCountry()
```

Try it out in a new Vim instance : Position a cursor somewhere in a buffer and run `:InsertCountry`


### Binding function calls to key combinations

To map a key combination for this, run :

```
:map <Leader>c :InsertCountry<CR>
```

and press `<Leader> c` to run the command 
(check out [](https://stackoverflow.com/questions/1764263/what-is-the-leader-in-a-vimrc-file) to find out what `<Leader>` means).

This is a significant upgrade to the functionality available :  
Our users can add the mapping to `~/.vimrc` and their country name is just two key presses away!

Vim plugins can do a lot more interesting things. 
What is possible and how to use vim module is well documented in Vim itself. 
Check out help: `:help python-vim`.  Note that this (like Vim itself) has quite a learning curve.


### Configuration

This section is simple, since we already saw everything we need to provide a configuration for our plugin. 
Typically, users will configure the plugin in their `~/.vimrc` file and that will set some global variables, 
which we will later access in a plugin and use to adjust its behaviour. 
Say, we want to configure our plugin to provide either country names, or ISO codes. 
Add the following to your `~/.vimrc`:

```
let g:SamplePluginUseCountryCodes = 1
```

And then, access it in Python code:

```
vim.eval('g:SamplePluginUseCountryCodes')
```

Heads up: `eval` will only return a string, list or a dict, 
depending on type of data used in `VimL`. 
In this case, it is a string, so normally you would actually use it like so:

```
use_codes = vim.eval('g:SamplePluginUseCountryCodes').strip() != '0'
```

You can technically ask users to use ‘true’ and ‘false’ in this case for example, 
but it is good idea to stick to the behaviour users are already used to with the majority of other plugins, 
which is using `0` and `1` for this.



### Getting a bit more sophisticated

We are almost done. Let's just finalize our `VimL` wrapper. 
It makes sense to add two more features to it:

*  ensure that our plugin is only started when Python is actually available in Vim (this prevents Vim from spitting too many errors to the user when Python is not available)
*  ensure that the plugin is initialized once and only once.

The following does precisely that:

```
if !has("python3")
  echo "vim has to be compiled with +python3 to run this"
  finish
endif

if exists('g:sample_python_plugin_loaded')
    finish
endif

; the rest of plugin VimL code goes here

let g:sample_python_plugin_loaded = 1
```

Now, for example, if our user does something like `:source ~/.vimrc`, we can be sure that our plugin:

*  won't try to run the initialization code again
*  won't change `sys.path again`, 
*  won't import python modules or execute mode-level code. 


### Provide documentation

TODO: Describe documentation process


### Publish a plugin

TODO: Show how to add this to vim plugin central


### That's it!

This repository is its own final source code... 

Certainly check out :help python which contanins a lot of important details.

If you know of other important tricks, or have a good advice - please, leave a comment below. 
I'm very interested in further improvments of my Vim plugin development workflow and implementation.

Hope it was useful. Have fun with Vim!

