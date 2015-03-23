---
title: How i wrote a dissertation in MARKDOWN
layout: post
category : posts
tags : [markdown, python, pdf, pandoc]
---

##Oh yes i love MARKDOWN
A couple of months ago i started to write a dissertation about a web project for a Bachelor of Computer Science; Surfing the web i saw the best way to write a thesis is using the latex markdown syntax, very powerful for technical writing with support for math formulas and tons of extensions.


Ok, powerful, flexible, standard, but the syntax could be like this `$latex e^{\i \pi} + 1 = 0&bg=00ff00&fg=ff0000$`. Uhm, not so simple! I just need to write about a technical project with some images, tables and references. I don't want to die drowning in the complexity of latex so the choice it was quite obvious, **i will not use latex!!**.

The reasons that led me to ***markdown*** syntax were the semplicity and the readability, they allowed me to write in plain text with multiple devices and keep the text synchronized by the dropbox cloud service.

This is the story of a dissertation about "*Booking platform with REST apis*"

## Lazy developer
I'm a software developer, all my live is under the version control system and i've never started a new project without the `git init` command so  **git** is first tool that helped me to manage this dissertation-project.
Ok markdown and git, this is all i need to start writing.

* Create the directory project `mkdir dissertation && cd dissertation`
* Initialize the version control `git init`
* Start writing...

... *some hours late* ...

Oh the first chapter is almost complete, now i need to convert from markdown to pdf, maybe the best tool is [Pandoc](http://johnmacfarlane.net/pandoc/)
, a very powerful document format converter which allows the automatic table of contents generation, bibliography integration and tons of others customization.

Well done, i've all i need: a comphrensive syntax supporting tables, table of contents, images, links, bibliography and a document converter tool.

Let's write!

##The flow
So the write loop now is :

1. Write the text ;
2. Convert from `*.md` to `dissertation.pdf` with `pandoc dissertation.md--toc -s thesis.pdf`;
3. Commit the changes with git ;
4. Send to the professor.
5. goto 1

##Chapters
Within a couple of hours i wrote a huge file with tables, code snippets and images. The markdown file was hard to read and poorly maintainable so i decided to create a virtual chapters structure and split the dissertation in multiple files. The structure looks like this

    /dissertation
        /chapters
            /01-intro.md
            /02-the-project.md
            /03-implementation.md
            /04-infrastructure.md
            /05-performance.md
            /06-conclusions.md
        /dist
            dissertation.pdf

##I still have too much mechanical steps
The writing flow is still too much expensive for a lazy developer like me so
i wrote a small python script to help the conversion process, it's composed by a `PandocConverter` and a `compile.py` script

#####PandocConverter
{% highlight python %}
    from subprocess import call
    import os
     
        class PandocConverter:
     
            def __init__(self, destination = 'dist', enableLogger = True):
                self.commandBase = ['pandoc']
                self.destination = destination
                self.isLoggerEnabled = enableLogger
         
            def log(self, msg):
                if (self.isLoggerEnabled):
                    print msg;
         
            def list_files_in_folder(self, folder):
                basePath = os.path.abspath(folder)
                files    = os.listdir(basePath)
                files.sort()
                return map(lambda f: os.path.join(basePath,f), files)
         
            def ensure_dir(self, directory):
                if not os.path.exists(directory):
                    os.makedirs(directory)
         
            def compile_doc(self, files, formats, outputBaseName, options = []):
         
                for format in formats:
                    self.ensure_dir(self.destination)
                    compiledDoc = 
                        "%s/%s.%s" % (
                            self.destination, 
                            outputBaseName, 
                            format
                            ) 
                    self.log("Generating %s" % (compiledDoc))
                    commandOut  = ['-o', compiledDoc]
                    call(self.commandBase + files + options + commandOut)
                    
                    return compiledDoc
{% endhighlight %}

#####compile.py
{% highlight python %}
    #!/usr/bin/env python
    import sys

    from PandocConverter import PandocConverter

    formats = sys.argv[1:]
    converter = PandocConverter('dist')

    chaptersFiles = converter.list_files_in_folder('chapters')

    content  = converter.compile_doc(
        chaptersFiles, 
        formats, 
        'thesis', 
        ['--toc', '-s']
    )
{% endhighlight %}    

Ok now i've the right toolbox to write.

The repository structure looks like this

    /dissertation
        PandocConverter.py
        compile.py
        /chapters
            /01-intro.md
            /02-the-project.md
            /03-implementation.md
            /04-infrastructure.md
            /05-performance.md
            /06-conclusions.md
        /dist
            dissertation.pdf


If you'are using a live editing tool such as SublimeText + [Markdown Editing](https://sublime.wbond.net/packages/MarkdownEditing) i suggest you to hook the `compile.py pdf` command within the [pre-commit event](https://github.com/git/git/blob/master/templates/hooks--pre-commit.sample)

