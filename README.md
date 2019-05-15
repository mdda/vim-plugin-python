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

Now, let’s make sure this actually works. Let’s add following content to `sampleplugin.vim`:

```
echo "It worked!"
```

And start new Vim instance where we will test the plugin. 
Upon startup you should see “It worked!” printed out in the terminal. 

If at this point it doesn’t work, try to load the plugin manually. 
For this, execute following command from Vim: `:source ~/.vim/bundle/sampleplugin/plugin/sampleplugin.vim`. 
Now, if this finally works, 
it means that your plugin manager doesn’t load the plugin automatically on Vim startup - 
refer to your plugin manager documentation to find out how to configure it correctly. 
If however this doesn't work either - Vim should normally print out an error message, 
which should give you a better idea. 
Most likely you need to check that file actually exists and symbolic link works as expected, 
and that file content (syntax) is correct.

All set! Let’s write some Python!

