# NAME

App::Followme::Guide - How to install, configure, and run followme

# SYNOPSIS

    followme [directory]
    

# DESCRIPTION

Updates a static website after changes. Constant portions of each page are
updated to match, text files are converted to html, and indexes are created
for files in the archive.

The followme script is run on the directory or file passed as its argument. If
no argument is given, it is run on the current directory.

If a file is passed, the script is run on the directory the file is in. In
addition, the script is run in quick mode, meaning that only the directory
the file is in is checked for changes. Otherwise not only that directory, but
all directories below it are checked.

# INSTALLATION

First, install the App::Followme module from CPAN. It will copy the
followme script to /usr/local/bin, so it will be on your search path.

    sudo cpanm App::Followme

Then create a folder to contain the new website. Run followme with the
init option in that directory

    mkdir website
    cd website
    followme --init

It will install the initial templates and configuration files. The initial
setup is configured to update pages to maintain a consistent look for the site
and to create a weblog from files placed in the archive directory. If you do not
want a weblog, just delete the archive directory and its contents.

To start creating your site, create the index page as a Markdown file, run
followme again, and edit the resulting page:

    vi index.md
    followme
    vi index.html
    

The first page will serve as a prototype for the rest of your site. When you
look at the html page, you will see that it contains comments looking like

    <!-- section content -->
    <!-- endsection content -->

These comments mark the parts of the prototype that will change from page to
page from the parts that are constant across the entire site. Everything
outside the comments is the constant portion of the page. When you have
more than one html page in the folder, you can edit any page, run followme,
and the other pages will be updated to match it.

So you should edit your first page and add any other files you need to create
the look of your site, such as the style sheet.

You can also use followme on an existing site. Run the command

    followme --init
    

in the top directory of your site. The init option will not overwrite any
existing files in your site. Then look at the page template it has
created:

    cat templates/page.htm

Edit the existing pages in your site to have all the section comments in this
template.

# CONFIGURATION

The configuration file for followme is followme.cfg in the top directory of
your site. Configuration file lines are organized as lines containing

    NAME = VALUE

pairs. Configuration files may also contain blank lines or comment lines
starting with a `#`. Subdirectories of the top directory may also contain
configuration files. Values in these configuration files are combined with those
set in the configuration files in directories above it, If it has a parameter of
the same name as a configuration file in a higher directory, it overrides it for
that directory and its subdirectories.

Configuration files contain the names of the Perl modules to be run by followme
in the parameters named run\_before and run\_after

    run_before = App::Followme::FormatPages
    run_before = App::Followme::ConvertPages
    run_after = App::Followme::CreateSitemap

Perl modules are run in the order they appear in the configuration file. If they
are named run\_before then they are run before modules in any configuration files
contained in subdirectories. If they are named run\_after, they are run after
modules which are named in the configuration files in subdirectories. Other
parameters in the configuration files are written to a hash. This hash is passed
to the new method of each module as it loaded, overriding the default values of
the parameters when creating the new object.

These modules are distributed with followme:

- [App::Followme::FormatPages](https://metacpan.org/pod/App::Followme::FormatPages)

    This module updates the web pages in a folder to match the most recently
    modified page. Each web page has sections that are different from other pages
    and other sections that are the same. The sections that differ are enclosed in
    html comments that look like

        <!-- section name-->
        <!-- endsection name -->

    and indicate where the section begins and ends. When a page is changed, this
    module checks the text outside of these comments. If that text has changed. the
    other pages on the site are also changed to match the page that has changed.
    Each page updated by substituting all its named blocks into corresponding block
    in the changed page. The effect is that all the text outside the named blocks
    are updated to be the same across all the web pages.

    In addition to normal section blocks, there are per folder section blocks.
    The contents of these blocks is kept constant across all files in a folder and
    all subfolders of it. If the block is changed in one file in the folder, it will
    be updated in all the other files. Per folder section blocks look like

        <!-- section name in folder_name -->
        <!-- endsection name -->

    where folder\_name is the the folder the content is kept constant across. The
    folder name is not a full path, it is the last folder in the path.

- [App::Followme::ConvertPages](https://metacpan.org/pod/App::Followme::ConvertPages)

    This module changes Markdown files to html files. Markdown format is described
    at:

        http://daringfireball.net/projects/markdown/

    It builds several variables and substitutes them into the page template. The
    most significant variable is body, which is the contents of the text file
    after it has been converted by Markdown. The title is built from the title of
    the Markdown file if one is put at the top of the file. If the file has no
    title, it is built from the file name, replacing dashes with blanks and
    capitalizing each word, The url and absolute\_url are built from the html file
    name. A number of time variables are built from the modification date of the
    text file: weekday, month, monthnum, day, year, hour24, hour, ampm, minute, and
    second. To change the look of the html page, edit the page template. Only blocks
    inside the section comments will be in the resulting page, editing the text
    outside it will have no effect on the resulting page.

- [App::Followme::CreateIndex](https://metacpan.org/pod/App::Followme::CreateIndex)

    This module builds an index for a directory containing links to all the files
    with the specified extension contained in it. The same variables mentioned above
    are calculated for each file, with the exception of body. Comments that look like

        <!-- for @loop -->
        <!-- endfor -->

    indicate the section of the template that is repeated for each file contained
    in the index. 

- [App::Followme::CreateNews](https://metacpan.org/pod/App::Followme::CreateNews)

    This module generates an html file from the most recently updated files in the
    news directory. It also creates index files in each directory and
    subdirectory in the news directory. The same variables mentioned under
    [App::Followme::ConvertPages](https://metacpan.org/pod/App::Followme::ConvertPages) are calculated for each file included in the
    indexes.

- [App::Followme::CreateSitemap](https://metacpan.org/pod/App::Followme::CreateSitemap)

    This module creates a sitemap file, which is a text file containing the url of
    every page on the site, one per line. It is also intended as a simple example of
    how to write a module that can be run by followme.

- [App::Followme::UploadSite](https://metacpan.org/pod/App::Followme::UploadSite)

    This module uploads changed files to a remote site. The default method to do the
    uploads is ftp, but that can be changed by changing the parameter upload\_pkg.
    This package computes a checksum for every file in the site. If the checksum has
    changed since the last time it was run, the file is uploaded to the remote site.
    If there is a checksum, but no local file, the file is deleted from the remote
    site. If followme is run in quick mode, only files whose modification date is
    later then the last time it was run are checked.

# RUNNING

The followme script is run on the directory or file passed as its argument. If
no argument is given, it is run on the current directory. If a file is passed,
the script is run on the directory the file is in and followme is run in
quick mode. Quick mode is an implicit promise that only the named file has
been changed since last time. Each module can make up this assumption what it
will, but it is supposed to shorten the list of files examined.

Followme looks for its configuration files in all the directories above the
directory it is run from and runs all the modules it finds in them. But they are
are only run on the folder it is run from and subfolders of it. Followme only
looks at the folder it is run from to determine if other files in the folder
need to be updated. So after changing a file, followme should be run from the
directory containing the file.

# TEMPLATES

Templates are read either from the same directory as the configuration file
containing the name of the module being run or from the templates subdirectory
of the top directory of the site.

Templates support the control structures in Perl: "for" and "while" loops,
"if-else" blocks, and some others. Creating output is a two step process. First
followme generates a subroutine from one or more templates, then you call the
subroutine with your data to generate the output.

The template format is line oriented. Commands are enclosed in an html comment
(<!-- -->) on its own line. A command may be preceded or followed by white
space. If a command is a block command, it is terminated by the word "end"
followed by the command name. For example, the "for" command is terminated by an
"endfor" command and the "if" command by an "endif" command.

All lines may contain variables. As in Perl, variables are a sigil character
('$,' '@,' or '%') followed by one or more word characters. For example,
`$name` or `@names`. To indicate a literal character instead of a variable,
precede the sigil with a backslash. When followme runs the subroutine that is
generated, it is passed a reference to a hash generated from the file or files
examined by the module. By convention if more than one file is examined, the
data is in an array referenced by a hash field named 'loop'. The subroutine
replaces variables in the template with the value in the field of the same name
in the hash. If the types of the two disagree, the code will coerce the data to
the type of the sigil.

If the first non-white characters on a line are the comment start string, the
line is interpreted as a command. The command name continues up to the first
white space character. The text following the initial span of white space is the
command argument. The argument continues up to the comment end string.

Variables in the template have the same format as ordinary Perl variables,
a string of word characters starting with a sigil character. for example,

    $summary @loop %dictionary

are examples of variables. The subroutine this module generates will substitute
values in the data it is passed for the variables in the template. New variables
can be added with the "set" command.

Arrays and hashes are rendered as unordered lists and definition lists when
interpolating them. This is done recursively, so arbitrary structures can be
rendered. This is mostly intended for debugging, as it does not provide fine
control over how the structures are rendered. For finer control, use the
commands described below so that the scalar fields in the structures can be
accessed. Undefined fields are replaced with the empty string when rendering. If
the type of data passed to the subroutine differs from the sigil on the variable
the variable is coerced to the type of the sigil. This works the same as an
assignment. If an array is referenced as a scalar, the length of the array is
output.

The following commands are supported in templates:

- do

    The remainder of the line is interpreted as Perl code. For assignments, use
    the set command.

- if

    The text until the matching `endif` is included only if the expression in the
    "if" command is true. If false, the text is skipped. The "if" command can contain
    an `else`, in which case the text before the "else" is included if the
    expression in the "if" command is true and the text after the "else" is included
    if it is false. You can also place an "elsif" command in the "if" block, which
    includes the following text if its expression is true.

        <!-- if $highlight eq 'y' -->
        <em>$text</em>
        <!-- else -->
        $text
        <!-- endif -->

- for

    Expand the text between the "for" and "endfor" commands several times. The
    "for" command takes a name of a field in a hash as its argument. The value of this
    name should be a reference to a list. It will expand the text in the for block
    once for each element in the list. Within the "for" block, any element of the list
    is accessible. This is especially useful for displaying lists of hashes. For
    example, suppose the data field name phonelist points to an array. This array is
    a list of hashes, and each hash has two entries, name and phone. Then the code

        <!-- for @phonelist -->
        <p>$name<br>
        $phone</p>
        <!-- endfor -->

    displays the entire phone list.

- section

    If a template contains a section, the text until the endsection command will be
    replaced by the section block with the same name in one the subtemplates. For
    example, if the main template has the code

        <!-- section footer -->
        <div></div>
        <!-- endsection -->

    and the subtemplate has the lines

        <!-- section footer -->
        <div>This template is copyright with a Creative Commons License.</div>
        <!-- endsection -->

    The text will be copied from a section in the subtemplate into a section of the
    same name in the template. If there is no block with the same name in the
    subtemplate, the text is used unchanged.

- set

    Adds a new variable or updates the value of an existing variable. The argument
    following the command name looks like any Perl assignment statement minus the
    trailing semicolon. For example,

        <!-- set $link = "<a href=\"$url\">$title</a>" -->

- while

    Expand the text between the `while` and `endwhile` as long as the
    expression following the `while` is true.

        <!-- set $i = 10 -->
        <p>Countdown ...<br>
        <!-- while $i >= 0 -->
        $i<br>
        <!-- set $i = $i - 1 -->
        <!-- endwhile -->

- with

    Lists within a hash can be accessed using the "for" command. Hashes within a
    hash are accessed using the "with" command. For example:

        <!-- with %address -->
        <p><i>$street<br />
        $city, $state $zip</i></p.
        <!-- endwith -->

More information on the syntax of template is in the documentation of
the [App::Followme::Template](https://metacpan.org/pod/App::Followme::Template) module.

# MODULES

New modules can be written and then invoked via the configuration file, exactly
like the modules that have been distributed with App::Followme. Each module to
be run must have new and run methods. An object of the module's class is created
by calling the new method with the a reference to a hash containing the
configuration parameters. The run method is then called with the directory as
its argument.

The signature of the new method is

    $obj = $module_name->new($configuration);
    

where $configuration is a reference to a hash containing the configuration
parameters. $module name is the same as the name in the configuration file.

All the modules distributed with App::Followme subclass
App::Followme::Module to access its methods, which provide consistent
behavior, such as looping over files and template handling. It also supplies a
new method, so if you subclass it, you will not need to supply a new method in
your class.

The signature of the run method is

    $obj->run($directory);

where $obj is the object created by the new method and $directory is the name
of the directory the module is being run on. All modules included in
App::Followme use [App::Followme::Module](https://metacpan.org/pod/App::Followme::Module) as a base class, so they can use its
methods, such as visiting all files in a directory and compiling a template. If
you wish to write your own module, you can use [App::Followme::Sitemap](https://metacpan.org/pod/App::Followme::Sitemap) as a
guide. If you use App::Followme::Module as a base class, you should not supply
your own new method, but rely on the new method in
[App::Followme::ConfiguredObject](https://metacpan.org/pod/App::Followme::ConfiguredObject), which you will inherit.

# LICENSE

Copyright (C) Bernie Simon.

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# AUTHOR

Bernie Simon <bernie.simon@gmail.com>
