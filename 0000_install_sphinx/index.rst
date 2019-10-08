Install and use Sphinx
===========================

Requirements
--------------

1. Python installed, in my example Python 3.7 on Windows
2. pip from python

Update pip
............

see: https://datatofish.com/upgrade-pip/

steps:

1. start cmd as ADMIN [Win] -> search
2. cd C:\Program Files\Python37
   Bash: also try which python
3. python -m pip install --upgrade pip

-->done


install sphinx
------------------------

see: https://www.sphinx-doc.org/en/1.5/tutorial.html

steps:

1. start cmd as ADMIN [Win] -> search
2. cd C:\Program Files\Python37
3. pip install Sphinx

Now without ADMIN


setup sphinx
------------------

1. start a shell and goto location
2. sphinx-quickstart

additional for Windows:

3. create a simple batch file: "build_sphinx.bat" with this code

@ECHO ON

REM build sphinx documentation as html

make.bat html

4. optional: add "PAUSE" to the end of make.bat
5. run this batch file, this will create the basic folder structure
6. create a link to _build/html/index.html to the basis folder

--> done

Now to create the documentation just run "build_sphinx.bat" and open the link to index.html




