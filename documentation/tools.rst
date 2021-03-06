=======
 Tools
=======

These command line tools can be started with ``python -m <toolname>``.


launcher_tool
=============
This is the main tool to create customized launcher executables.
It combines the user application with the executable (``launcher27.exe`` or
``launcher3.exe``) that runs a local Python distribution .

A ZIP file is appended to the launcher executable, containing:

- A script with the name ``__main__.py``. This is the entry point. This file
  is autogenerated, based on the command line options used.
- ``launcher.py``, a helper module for the boot script is also appended.
- Any other contents specified on the command line. If this contains a
  ``__main__.py`` then that file will be renamed (dropping the final two
  underscores) so that it does not interfere with the autogenerated file.

The options under "extra customization" invoke the ``resource_editor`` module
and are here for convenience. See below for restrictions.


Here is the output of ``python -m launcher_tool -h``::

    usage: __main__.py [-h] (-o FILE | -a FILE)                                                           
                      [-e MODULE:FUNC | --run-path FILE | -m MODULE | --main FILE]                       
                      [--add-file FILE] [--add-zip ZIPFILE] [--launcher EXE]                             
                      [--wait] [--wait-on-error] [-p PATTERN] [--32 | --64]                              
                      [-2 | -3]                                                                          
                                                                                                          
    create customized launcher.exe                                                                        
                                                                                                          
    optional arguments:                                                                                   
      -h, --help            show this help message and exit                                               
                                                                                                          
    output options:                                                                                       
      -o FILE, --output FILE                                                                              
                            filename to write the result to                                               
      -a FILE, --append-only FILE                                                                         
                            append to this file instead of creating a new one                             
                                                                                                          
    entry point options:                                                                                  
      -e MODULE:FUNC, --entry-point MODULE:FUNC                                                           
                            import given module and call function                                         
      --run-path FILE       execute given file (e.g. .py, .zip)                                           
      -m MODULE, --run-module MODULE                                                                      
                            execute module (similar to python -m)                                         
      --main FILE           use this as __main__.py instead of built-in code                              
                                                                                                          
    customization:                                                                                        
      --launcher EXE        launcher executable to use instead of built-in one                            
      --wait                do not close console window automatically                                     
      --wait-on-error       wait if there is an exception                                                 
      -p PATTERN, --extend-sys-path PATTERN                                                               
                            add search pattern for files added to sys.path                                
                                                                                                          
    extra customization:
      --icon ICON           filename of icon to use
      --python-minimal DIR  change the location of the python-minimal distribution
      --bin-dir             put binaries in subdirectory /bin

    launcher architecture:                                                                                
      default value is based on sys.executable                                                            
                                                                                                          
      --32                  force copy of 32 bit version                                                  
      --64                  force copy of 64 bit version                                                  
                                                                                                          
    launcher Python version:                                                                              
      default value is based on sys.executable                                                            
                                                                                                          
      -2                    force use of Python 2.7 launcher                                              
      -3                    force use of Python 3.x launcher                                              

    additional ZIP contents:                              
      --add-file FILE       add additional file(s) to zip 
      --add-zip ZIPFILE     add contents of zip file(s)   
                                                        

launcher_tool.create_python27_minimal
=====================================
There is no official "embedded" distribution for Python 2.x so this tool
packages a local copy of Python to create a python27-minimal distribution.

There must be an unmodified copy of Python 2.7 installed (e.g. no changes to
Pythons library, installed modules in ``site-packages`` are OK, they are not
copied). Ensure to update to get the latest security fixes.

It does not package tkinter and tests (comparable to the Python 3 embedded
distribution).

Here is the output of ``python -m launcher_tool.create_python27_minimal -h``::

    usage: create_python27_minimal.py [-h] [-d DIR] [-n NAME]

    extract a copy of python27-minimal

    optional arguments:
      -h, --help            show this help message and exit

    output options:
      -d DIR, --directory DIR
                            set a destination directory, a subdirectory DIR will
                            be creted [default: .]
      -n NAME, --name NAME  Set a directory name [default: python27-minimal]


launcher_tool.download_python3_minimal
======================================
Unpack a Python 3 embedded distribution. The data is downloaded from
https://www.python.org/downloads/windows/
and cached locally (so that for repeated runs, it does not need to use
the Internet again). Command line options can be used to select the
desired Python version and architecture.

Here is the output of ``python -m launcher_tool.download_python3_minimal -h``::

    usage: download_python3_minimal.py [-h] [-d DIR] [-n NAME] [--32 | --64]
                                      [--this-version] [-p PYTHON_VERSION]
                                      [--url URL] [-f]

    download/extract for python3-minimal

    optional arguments:
      -h, --help            show this help message and exit

    output options:
      -d DIR, --directory DIR
                            set a destination directory, a subdirectory DIR will
                            be created [default: .]
      -n NAME, --name NAME  set a directory name [default: python3-minimal]

    download options:
      --32                  force download of 32 bit version
      --64                  force download of 64 bit version
      --this-version        choose this Python version that is running now
      -p PYTHON_VERSION, --python-version PYTHON_VERSION
                            choose Python version (major.minor, default=3.6.0)
      --url URL             override download URL
      -f, --force-download  force download (ignore/overwrite cached file)


launcher_tool.copy_launcher
===========================
Copy the ``launcher.exe`` to a file. Used e.g. for customizations using
``launcher_tool.resource_editor``. Launcher executables are bundled as package
data and this tool can be used to extract them, selecting the variant, as
multiple exist (four, one for each combination of Python 2.x/3.x and 32/64
bits).

Here is the output of ``python -m launcher_tool.copy_launcher -h``::

    usage: copy_launcher.py [-h] [-o FILE] [--32 | --64] [-2 | -3]

    copy the launcher.exe

    optional arguments:
      -h, --help            show this help message and exit

    output options:
      -o FILE, --output FILE
                            write to this file

    architecture:
      default value is based on sys.executable

      --32                  force copy of 32 bit version
      --64                  force copy of 64 bit version

    launcher Python version:
      default value is based on sys.executable

      -2                    force use of Python 2.7 launcher
      -3                    force use of Python 3.x launcher


launcher_tool.resource_editor
=============================
This is a small Windows resource editor that can modify resources in exe files
so for example in the launcher executables.

It uses Windows API functions to read and write the data (and therefore can
only be run under Windows).

- adding and editing strings
- retrieving and writing icons
- export resources as (binary) blob
- removing any resource type
- adding any resource type is supported partially (currently limited by
  data input possibilities)
- dump resources
- dump decoded string table

.. attention::

    It will strip debug data and remove the attached ZIP file! So this tool
    must be used before the application is appended to the launcher.

Here is the output of ``python -m launcher_tool.resource_editor -h``::

    usage: resource_editor.py [-h]
                              FILE
                              {dump,list,export,export_icon,write_icon,edit,dump_strings,edit_strings}
                              ...

    Windows Resource Editor

    positional arguments:
      FILE                  file containing the resources (.exe, .dll)
      {dump,list,export,export_icon,write_icon,edit,dump_strings,edit_strings}
                            sub-command help
        dump                read and output resources.
        list                read and output resources identifiers.
        export              export one entry to a file.
        export_icon         export icon to a file.
        write_icon          write icon to a resource file.
        edit                edit resources.
        dump_strings        read and output string table resource.
        edit_strings        edit resources.

    optional arguments:
      -h, --help            show this help message and exit
