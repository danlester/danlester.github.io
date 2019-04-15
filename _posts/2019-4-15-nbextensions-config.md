---
layout: post
title: How config and installation works for Jupyter Notebook Extensions
---

If you're working with Jupyter Notebook extensions (nbextensions) you may have used commands such as `jupyter nbextension enable <extension name>` to make them work or to stop them from loading temporarily.

For a simple Jupyter installation you can't go too wrong, but if you have multiple virtual environments things can becoming confusing if you aren't familiar with the way Jupyter configuration works.

If you are debugging your Jupyter installation or developing your own nbextension widgets, you might need to know a bit more. Since Notebook 5.3, pip-installing nbextensions can now allow the installation and activation to happen automatically - which is great but you want to know more about the config behind the magic.

This post describes only the steps and files involved in the installation and enabling of a Notebook extension, allowing you to understand a bit more about the config that happens behind the scenes from the point of view of the extension itself. In the examples we will install the [Jupyter Innotater](https://github.com/ideonate/jupyter-innotater) widget extension. 

## Jupyter script commands

First of all, what exactly runs when you execute commands such as `jupyter notebook` or `jupyter nbextension`? This is helpful to know if you are debugging Jupyter Notebook itself.

You'll find the `jupyter` command in one of your 'bin' folders, e.g. /usr/local/bin depending on how you installed Jupyter. If you look at the script itself (e.g. `cat /usr/local/bin/jupyter`) you'll see that all it really does is run the function main in the jupyter_core.command Python module. All _that_ does is run the first subcommand supplied in the command line as a separate new command in an expected format. For example, if you type `jupyter notebook` on the command line, the jupyter script simply attempts to run a file called `jupyter-notebook`, which will hopefully be found in the right location for it to run.

 How does jupyter-notebook appear in the bin folder in the first place (or the jupyter script itself for that matter)?
 
 When you `pip install jupyter`, the setup.py file for the jupyter_notebook module automatically creates the script(s). Take a look at the `entry_points` section in [setup.py](https://github.com/jupyter/notebook/blob/master/setup.py):
 
```python
entry_points = {
        'console_scripts': [
            'jupyter-notebook = notebook.notebookapp:main',
            'jupyter-nbextension = notebook.nbextensions:main',
            'jupyter-serverextension = notebook.serverextensions:main',
            'jupyter-bundlerextension = notebook.bundler.bundlerextensions:main',
        ]
    }
    ...
```

Something in the `setuptools` module (used by `setup.py` at install time) understands this configuration as saying to generate a script called `jupyter-notebook` (and the others) which does little more than act as an entry point to the `main` function in the Python module called `notebook.notebookapp`. It just wraps its invocation up in some supporting bash or other shell script code.

## Jupyter Config files

Run the command `jupyter --paths` and you'll see something like:
```shell
config:
    /Users/dan/.jupyter
    /usr/local/etc/jupyter
    /etc/jupyter
data:
    /Users/dan/Library/Jupyter
    /usr/local/share/jupyter
    /usr/share/jupyter
runtime:
    /Users/dan/Library/Jupyter/runtime
```

Info from [Jupyter docs is here](https://jupyter.readthedocs.io/en/latest/projects/jupyter-directories.html).

Simplifying things a bit, basically the config section shows a series of paths that will be searched, in reverse order, for JSON files containing configuration values. They will be read in reverse order but values will be overridden if found later on in the search - so the folders listed first are ultimately given priority.

The first path (/Users/dan/.jupyter) is user-specific, and the second path of /usr/local/etc/jupyter is known as the 'sys-prefix' location. This is because it is located within the value of the [sys.prefix](https://docs.python.org/3/library/sys.html#sys.prefix) constant of the relevant Python installation.

Generally, the sys-prefix configuration works out best for development since it is specific to any virtualenv you may have active, whereas the user-specific config will be read by Jupyters in all virtualenvs (unless you remove it from the Jupyter path variables of course).

 ### Nbextensions manual install and enable

Beyond installing the Python module of Jupyter Innotater, there are two main nbextension-specific steps required to get the widget working. Of course you will install before enabling, but I'll cover these steps in reverse order:
 
 #### Enable
 
 If you're installing jupyter_innotater via pip (`pip install jupyter_innotater`), the install and enable steps should be carried out automatically (see later), but let's imagine your config has gone wrong and only the jupyter_innotater Python module has been installed correctly (i.e. exists in your `site-packages` folder) but the widget isn't working in the Notebook. Then you might run:
 
```shell
jupyter nbextension enable --py --sys-prefix jupyter_innotater
```

The --py argument means that the extension is available as a Python module (called jupyter_innotater in this case). And --sys-prefix means we want to enable at the sys-prefix config level (not user-specific).

This command should make the following addition to /usr/local/etc/jupyter/nbconfig/notebook.json:
```json
{
     "load_extensions": {
       "jupyter-innotater/extension": true
     }
   }
```
which instructs the Notebook to load the relevant Javascript files etc when it runs, presuming this configuration isn't overridden at the user level. Maybe it is, in which case running `jupyter nbextension list` will show you:

```shell
Known nbextensions:
  config dir: /Users/dan/.jupyter/nbconfig
    notebook section
      jupyter-innotater/extension disabled
  config dir: /local/etc/jupyter/nbconfig
    notebook section
      jupyter-innotater/extension  enabled 
      - Validating: OK
      jupyter-js-widgets/extension  enabled 
      - Validating: OK
```

The net result here is that Innotater will not be available in the Notebook. You would run `jupyter nbextension enable --py jupyter_innotater` (without --sys-prefix) to fix this.

 #### Install
 
 The Python module jupyter_innotater might already be installed and available to Python, so if your notebook starts with `from jupyter_innotater import Innotater`, cell execution might seem to go smoothly, but the widget doesn't show later on when you try to instantiate it. In the Javascript console you might find the file /static/jupyter-innotater.js is unavailable.
 
 The following step would hopefully fix this:
 ```shell
 jupyter nbextension install --sys-prefix --py jupyter_innotater
```

These arguments are the same as for 'enable', but instead of changing the config, install copies relevant Javascript and other media resources such as CSS files to a location where they can be found easily by Notebook. In particular, it will copy Javscript files to /usr/local/share/jupyter/nbextensions/jupyter-innotater - which is the sys-prefix level 'data path' (as opposed to 'config path') as displayed when we ran `jupyter --paths` earlier.

But how did the `jupyter nbextension install` command know which file(s) to copy? Simply by importing and running a function called `_jupyter_nbextension_paths` from the jupyter_innotater Python module. You can see it in the `__init__.py` file of the [jupyter_innotater Python module](https://github.com/ideonate/jupyter-innotater/blob/master/jupyter-innotater/jupyter_innotater/__init__.py):

```python
def _jupyter_nbextension_paths():
       return [{
           'section': 'notebook',
           'src': 'static',
           'dest': 'jupyter-innotater',
           'require': 'jupyter-innotater/extension'
       }]
```

The src and dest entries above show where to copy these media files from (the static folder of the jupyter_innotater Python module in this case) and to (a new folder called jupyter-innotater).

You can now also see how the `jupyter nbextension enable` command knew to add exactly the value: `"jupyter-innotater/extension": true` to the `load_extensions` entry of the config file when we enabled the extension earlier. It took this value from the require value in _jupyter_nbextension_paths.

### Nbextensions automatic install and enable via pip

So how do these install and enable steps happen automatically when you `pip install jupyter_innotater`?

Both steps are handled through setuptools when [setup.py](https://github.com/ideonate/jupyter-innotater/blob/master/jupyter-innotater/setup.py) is run at install time. And in both cases it is a simple matter of copying files. See the `data_files` section of setup.py:

```python
data_files = [
       ('share/jupyter/nbextensions/jupyter-innotater', [
           'jupyter_innotater/static/extension.js',
           'jupyter_innotater/static/extension.js.map',
           'jupyter_innotater/static/index.js',
           'jupyter_innotater/static/index.js.map',
       ]),
       ('etc/jupyter/nbconfig/notebook.d', [
           'enable_jupyter_innotater.json'
       ])
   
   ]
```

The data_files section is understood by [setuptools (more details here)](https://setuptools.readthedocs.io/en/latest/) as an instruction to copy the specified files from the package.

In the first entry above, the media files (crucially the js files) are asked to be copied to the relevant subfolder inside the sys-prefix level data folder (share/jupyter/nbextensions/jupyter-innotater). This is exactly what happens when we run the `jupyter nbextension install` command.

In the second entry, a json file is copied to a subfolder of the sys-prefix level config folder. Let's look at the [enable_jupyter_innotater.json file](https://github.com/ideonate/jupyter-innotater/blob/master/jupyter-innotater/enable_jupyter_innotater.json) contents:
```json
{
    "load_extensions": {
      "jupyter-innotater/extension": true
    }
  }
```

This is what we find within the notebook.json file when we run `jupyter nbextension enable` command, but in this case is found within an arbitrarily-named file within a config folder called `notebook.d`.

When we covered the config files earlier, we didn't mention the ".d" folder, so let's cover it now. At startup time, when Jupyter Notebook comes to look for the 'notebook' section in the sys-prefix level config, it will look for notebook.json as described earlier. But just before that it will look in the `notebook.d` folder (.d stands for 'directory') for _any_ json files that might want to contribute configuration. These are read before so that values in the `notebook.json` file may overwrite the .d values and thus take precedence.

So these file copying steps carried out during `pip install jupyter_notebook` explain why we don't also need to run the jupyter nbextension install and enable commands.



 


