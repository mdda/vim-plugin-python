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



