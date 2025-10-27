NOTE:

1 Arm or X86_64 Deiban 7 ~ 12 Compile passed;

2 You don't need mod_persistentperl to run it,nginx APACHE lighttpd etc all webserver；

3 Equivalent in performance to FastCGI；

4 Outside of the header, there is no need to modify the Perl script；


注意：

1、Arm 或 X86_64 Deiban 7 ~ 12编译通过；

2、不需要增加mod_persistentperl模块，即可在apache、nginx、lighttpd等所有web服务器上运行；

3、性能和fastcgi相当；

4、除了#!/usr/bin/perperl头部外，不需要修改严谨的perl脚本即可运行；






==================================================================================

NAME
    PersistentPerl - Speed up perl scripts by running them persistently.

SYNOPSIS
     #!/usr/bin/perperl

     ### Your Script Here.  For example:
     print "Content-type: text/html\n\nHello World!\n";

     ##
     ## Optionally, use the PersistentPerl module for various things
     ##

     # Create a PersistentPerl object
     use PersistentPerl;
     my $pp = PersistentPerl->new;

     # See if we are running under PersistentPerl or not.
     print "Running under perperl=", $pp->i_am_perperl ? 'yes' : 'no', "\n";

     # Register a shutdown handler
     $pp->add_shutdown_handler(sub { do something here });

     # Register a cleanup handler
     $pp->register_cleanup(sub { do something here });

     # Set/get some PersistentPerl options
     $pp->setopt('timeout', 30);
     print "maxruns=", $pp->getopt('maxruns'), "\n";

DESCRIPTION
    PersistentPerl is a way to run perl scripts persistently, which can make
    them run much more quickly. A script can be made to to run persistently
    by changing the interpreter line at the top of the script from:

        #!/usr/bin/perl

    to

        #!/usr/bin/perperl

    After the script is initially run, instead of exiting, the perl
    interpreter is kept running. During subsequent runs, this interpreter is
    used to handle new executions instead of starting a new perl interpreter
    each time. A very fast frontend program, written in C, is executed for
    each request. This fast frontend then contacts the persistent Perl
    process, which is usually already running, to do the work and return the
    results.

    By default each perl script runs in its own Unix process, so one perl
    script can't interfere with another. Command line options can also be
    used to deal with programs that have memory leaks or other problems that
    might keep them from otherwise running persistently.

    PersistentPerl can be used to speed up perl CGI scripts. It conforms to
    the CGI specification, and does not run perl code inside the web server.
    Since the perl interpreter runs outside the web server, it can't cause
    problems for the web server itself.

    PersistentPerl also provides an Apache module so that under the Apache
    web server, scripts can be run without the overhead of doing a fork/exec
    for each request. With this module a small amount of frontend code is
    run within the web server - the perl interpreters still run outside the
    server.

    SpeedyCGI and PersistentPerl are currently both names for the same code.
    SpeedyCGI was the original name, but because people weren't sure what it
    did, the name PersistentPerl was picked as an alias. At some point
    SpeedyCGI will be replaced by PersistentPerl, or become a sub-class of
    PersistentPerl to avoid always having two distributions.

OPTIONS
  Setting Option Values

    PersistentPerl options can be set in several ways:

    Command Line
        The perperl command line is the same as for regular perl, with the
        exception that PersistentPerl specific options can be passed in
        after a "--".

        For example the line:

                #!/usr/bin/perperl -w -- -t300

        at the top of your script will set the perl option "`-w'" and will
        pass the "`-t'" option to PersistentPerl, setting the Timeout value
        to 300 seconds.

    Environment
        Environment variables can be used to pass in options. This can only
        be done before the initial execution, not from within the script
        itself. The name of the environment variable is always PERPERL_
        followed by the option name in upper-case. For example to set the
        perperl Timeout option, use the environment variable named
        PERPERL_TIMEOUT.

    Module
        The PersistentPerl module provides the setopt method to set options
        from within the perl script at runtime. There is also a getopt
        method to retrieve the current options. See the section on "METHODS"
        below.

    Apache
        If you are using the optional Apache module, PersistentPerl options
        can be set in the httpd.conf file. The name of the apache directive
        will always be Persistent followed by the option name. For example
        to set the Timeout option, use the apache directive
        PersistentTimeout.

  Context

    Not all options below are available in all contexts. The context for
    which each option is valid is listed on the "Context" line in the
    section below. There are three contexts:

    perperl
        The command-line "perperl" program, used normally with #! at the top
        of your script or from a shell prompt.

    mod_persistentperl
        The optional Apache mod_persistentperl module.

    module
        During perl execution via the PersistentPerl module's getopt/setopt
        methods.

  Options Available

    BackendProg
            Command Line    : -p<string>
            Default Value   : "/usr/bin/perperl_backend"
            Context         : mod_persistentperl, perperl

            Description:

                Path to the perperl backend program.

    BufsizGet
            Command Line    : -B<number>
            Default Value   : 131072
            Context         : perperl

            Description:

                Use <number> bytes as the maximum size for the buffer that
                receives data from the perl backend.

    BufsizPost
            Command Line    : -b<number>
            Default Value   : 131072
            Context         : perperl

            Description:

                Use <number> bytes as the maximum size for the buffer that
                sends data to the perl backend.

    Group
            Command Line    : -g<string>
            Default Value   : "none"
            Context         : mod_persistentperl, perperl

            Description:

                Allow a single perl interpreter to run multiple scripts.
                All scripts that are run with the same group name and by
                the same user will be run by the same group of perl
                interpreters. If the group name is "none" then grouping is
                disabled and each interpreter will run one script.
                Different group names allow scripts to be separated into
                different groups. Name is case-sensitive, and only the
                first 12-characters are significant. Specifying an empty
                group name is the same as specifying the group name
                "default" - this allows just specifying "-g" on the command
                line to turn on grouping.

    MaxBackends
            Command Line    : -M<number>
            Default Value   : 0 (no max)
            Context         : mod_persistentperl, perperl

            Description:

                If non-zero, limits the number of perperl backends running
                for this perl script to <number>.

    MaxRuns
            Command Line    : -r<number>
            Default Value   : 500
            Context         : mod_persistentperl, module, perperl

            Description:

                Once the perl interpreter has run <number> times, re-exec
                the backend process.  Zero indicates no maximum.  This
                option is useful for processes that tend to consume
                resources over time.

    PerlArgs
            Command Line    : N/A
            Default Value   : ""
            Context         : mod_persistentperl

            Description:

                Command-line options to pass to the perl interpreter.

    Timeout
            Command Line    : -t<number>
            Default Value   : 3600 (one hour)
            Context         : mod_persistentperl, module, perperl

            Description:

                If no new requests have been received after <number>
                seconds, exit the persistent perl interpreter.  Zero
                indicates no timeout.

    TmpBase
            Command Line    : -T<string>
            Default Value   : "/tmp/perperl"
            Context         : mod_persistentperl, perperl

            Description:

                Use the given prefix for creating temporary files.  This
                must be a filename prefix, not a directory name.

    Version
            Command Line    : -v
            Context         : perperl

            Description:

                Print the PersistentPerl version and exit.

METHODS
    The following methods are available in the PersistentPerl module.

    new Create a new PersistentPerl object.

            my $pp = PersistentPerl->new;

    register_cleanup($function_ref)
        Register a function that will be called at the end of each request,
        after your script finishes running, but before STDOUT and STDERR are
        closed. Multiple functions can be added by calling the method more
        than once. At the end of the request, each function will be called
        in the order in which it was registered.

            $pp->register_cleanup(\&cleanup_func);

    add_shutdown_handler($function_ref)
        Add a function to the list of functions that will be called right
        before the perl interpreter exits. This is not at the end of each
        request, it is when the perl interpreter decides to exit completely
        due to a Timeout or reaching MaxRuns.

            $pp->add_shutdown_handler(sub {$dbh->logout});

    set_shutdown_handler($function_ref)
        Deprecated. Similar to `add_shutdown_handler', but only allows for a
        single function to be registered.

            $pp->set_shutdown_handler(sub {$dbh->logout});

    i_am_perperl
        Returns a boolean telling whether this script is running under
        PersistentPerl or not. A perl script can run under regular perl, or
        under PersistentPerl. This method allows the script to tell which
        environment it is in.

            $pp->i_am_perperl;

        To make your script as portable as possible, you can use the
        following test to make sure both the PersistentPerl module is
        available and you are running under PersistentPerl:

            if (eval {require PersistentPerl} && PersistentPerl->i_am_perperl) {
                Do something PersistentPerl specific here...

        To increase the speed of this check you can also test whether the
        following variable is defined instead of going through the object
        interface:

            $PersistentPerl::i_am_perperl

    setopt($optname, $value)
        Set one of the PersistentPerl options given in the section on
        "Options Available". Returns the option's previous value. $optname
        is case-insensitive.

            $pp->setopt('TIMEOUT', 300);

    getopt($optname)
        Return the current value of one of the PersistentPerl options.
        $optname is case-insensitive.

            $pp->getopt('TIMEOUT');

    shutdown_now
        Shut down the perl interpreter right away. This function does not
        return.

            $pp->shutdown_now

    shutdown_next_time
        Shut down the perl interpreter as soon as this request is done.

            $pp->shutdown_next_time

INSTALLATION
    To install PersistentPerl you will need to either download a binary
    package for your OS, or compile PersistentPerl from source code. See the
    section on "DOWNLOADING" for information on where to obtain the source
    code and binaries.

  Binary Installation

    Once you have downloaded the binary package for your OS, you'll need to
    install it using the normal package tools for your OS. The commands to
    do that are:

    Linux
         rpm -i <filename>

    Solaris
         gunzip <filename>.gz
         pkgadd -d <filename>

    BSD pkg_add <filename>

    If you are also installing the apache module you will have to configure
    Apache as documented in the section on "Apache Configuration".

  Source Code Installation

    To compile PersistentPerl you will need perl 5.005_03 or later, and a C
    compiler, preferably the same one that your perl distribution was
    compiled with. PersistentPerl is known to work under Solaris, Redhat
    Linux, FreeBSD and OpenBSD. There may be problems with other OSes or
    earlier versions of Perl. PersistentPerl may not work with threaded perl
    -- as of release 2.10, Linux and Solaris seem to work OK with threaded
    perl, but FreeBSD does not.

  Standard Install

    To do a standard install from source code, execute the following:

        perl Makefile.PL
        make
        make test
        make install

    This will install the perperl and perperl_backend binaries in the same
    directory where perl was installed, and the PersistentPerl.pm module in
    the standard perl lib directory. It will also attempt to install the
    mod_persistentperl module if you have the command apxs in your path.

  Install in a Different Directory

    If you don't have permission to install into the standard perl
    directory, or if you want to install elsewhere, the easiest way is to
    compile and install your own copy of perl in another location, then use
    your new version of perl when you run "perl Makefile.PL". The
    PersistentPerl binaries and module will then be installed in the same
    location as the new version of perl.

    If you can't install your own perl, you can take the following steps:

    *   Edit src/optdefs and change the default value for BackendProg to the
        location where perperl_backend will be installed.

    *   Compile as above, then manually copy the perperl and perperl_backend
        binaries to where you want to install them.

    *   If you want to use the PersistentPerl module in your code (it's not
        required), you will have to use "use lib" so it can be located.

  Setuid Install

    PersistentPerl has limited support for running setuid - installing this
    way may compromise the security of your system. To install setuid do the
    following:

    *   Run "perl Makefile.PL"

    *   Edit perperl/Makefile and add "-DIAMSUID" to the end of the "DEFINE = "
        line.

    *   Run make

    *   Take the resulting "perperl" binary and install it suid-root as
        /usr/bin/perperl_suid

    *   Change your setuid scripts to use /usr/bin/perperl_suid as the
        interpreter.

    This has been know to work in Linux and FreeBSD. Solaris will work as
    long as the Group option is set to "none".

  Apache Installation

    To compile the optional apache mod_persistentperl module you must have
    the apxs command in your path. Redhat includes this command with the
    "apache-devel" RPM, though it may not work properly for installation.

    If the apache installation fails:

    *   Copy the mod_persistentperl.so from the mod_persistentperl directory, or
        from the mod_persistentperl2/.libs directory, to wherever your
        apache modules are stored (try /usr/lib/apache)

    *   Edit your httpd.conf (try /etc/httpd/conf/httpd.conf) and add the
        following lines. The path at the end of the LoadModule directive may
        be different in your installation -- look at other LoadModules to
        see.

            LoadModule persistentperl_module modules/mod_persistentperl.so

        If you are using Apache-1, also add:

            AddModule mod_persistentperl.c

  Apache Configuration

    Once mod_persistentperl is installed, it has to be configured to be used
    for your perl scripts. There are two methods.

    Warning! The instructions below may compromise the security of your web
    site. The security risks associated with PersistentPerl are similar to
    those of regular CGI. If you don't understand the security implications
    of the changes below then don't make them.

    1. Path Configuration
        This is similar to the way /cgi-bin works - everything under this
        path is handled by PersistentPerl. Add the following lines near the
        top of your httpd.conf - this will cause all scripts in your cgi-bin
        directory to be handled by PersistentPerl when they are accessed as
        /perperl/script-name.

            Alias /perperl/ /home/httpd/cgi-bin/
            <Location /perperl>
                SetHandler persistentperl-script
                Options ExecCGI
                allow from all
            </Location>

    2. File Extension Configuration
        This will make PersistentPerl handle all files with a certain
        extension, similar to the way .cgi files work. Add the following
        lines near the top of your httpd.conf file - this will set up the
        file extension ".perperl" to be handled by PersistentPerl.

            AddHandler persistentperl-script .perperl
            <Location />
                Options ExecCGI
            </Location>

FREQUENTLY ASKED QUESTIONS
    How does the perperl front end connect to the back end process?
        Via a Unix socket in /tmp. A queue is kept in a shared file in /tmp
        that holds an entry for each process. In that queue are the pids of
        the perl processes waiting for connections. The frontend pulls a
        process out of this queue, connects to its socket, sends over the
        environment and argv, and then uses this socket for stdin/stdout to
        the perl process.

    If another request comes in while PersistentPerl script is running, does the client
    have to wait or is another process started?  Is there a way to set a limit
    on how many processes get started?
        If another request comes while all the perl processes are busy, then
        another perl process is started. Just like in regular perl there is
        normally no limit on how many processes get started. But, the
        processes are only started when the load is so high that they're
        necessary. If the load goes down, the processes will die off due to
        inactivity, unless you disable the timeout.

        Starting in version 1.8.3 an option was added to limit the number of
        perl backends running. See MaxBackends in the section on "Options
        Available" above.

    How much of perl's state is kept when perperl starts another request?
    Do globals keep their values?  Are destructors run after the request?
        Globals keep their values. Nothing is destroyed after the request.
        STDIN/STDOUT/STDERR are closed -- other files are not. `%ENV' and
        `@ARGV' are the only globals changed between requests.

    How can I make sure perperl restarts when I edit a perl library used
    by the CGI?
        Do a touch on the main cgi file that is executed. The mtime on the
        main file is checked each time the front-end runs.

    Do I need to be root to install and/or run PersistentPerl?
        No, root is not required.

    How can I determine if my perl app needs to be changed to work with
    perperl?  Or is there no modification necessary?
        You may have to make modifications.

        Globals retain their values between runs, which can be good for
        keeping persistent database handles for example, or bad if your code
        assumes they're undefined.

        Also, if you create global variables with "my", you shouldn't try to
        reference those variables from within a subroutine - you should pass
        them into the subroutine instead. Or better yet just declare global
        variables with "use vars" instead of "my" to avoid the problem
        altogether.

        Here's a good explanation of the problem - it's for mod_perl, but
        the same thing applies to persistentperl:

            http://perl.apache.org/docs/general/perl_reference/perl_reference.html#my___Scoped_Variable_in_Nested_Subroutines

        If all else fails you can disable persistence by setting MaxRuns to
        1. The only benefit of this over normal perl is that perperl will
        pre-compile your script.

    How do I keep a persistent connection to a database?
        Since globals retain their values between runs, the best way to do
        this is to store the connection in a global variable, then check on
        each run to see if that variable is already defined.

        For example, if your code has an "open_db_connection" subroutine
        that returns a database connection handle, you can use the code
        below to keep a persistent connection:

            use vars qw($dbh);
            unless (defined($dbh)) {
                $dbh = &open_db_connection;
            }

        This code will store a persistent database connection handle in the
        global variable "$dbh" and only initialize it the first time the
        code is run. During subsequent runs, the existing connection is re-
        used.

        You may also want to check the connection each time before using it,
        in case it is not working for some reason. So, assuming you have a
        subroutine named "db_connection_ok" that returns true if the db
        connection is working, you can use code like this:

            use vars qw($dbh);
            unless (defined($dbh) && &db_connection_ok($dbh)) {
                $dbh = &open_db_connection;
            }

    Why do scripts with persistent Oracle database connections hang?
        When using an IPC connection to Oracle, an oracle process is fork'ed
        and exec'ed and keeps the stdout connection open, so that the web
        server never gets an EOF. To fix the problem, either switch to using
        a TCP connection to the database, or add the following perl code
        somewhere before the DBI->connect statement:

            use Fcntl;
            fcntl(STDOUT, F_SETFD, FD_CLOEXEC);

        This will set the close-on-exec flag on standard out so it is closed
        when oracle is exec'ed.

USING GROUPS
    The group feature in PersistentPerl can be used to help reduce the
    amount of memory used by the perl interpreters. By default groups are
    not used (group name is "none"), and each perl script is given its own
    set of perl interpreters. Each perl interpreter is also a separate
    system process.

    When grouping is used, perl interpreters and perl scripts are put in a
    group. All perl interpreters in a group can run perl scripts in the same
    group. What this means is that by putting all your scripts into one
    group, there could be one perl interpreter running all the perl scripts
    on your system. This can greatly reduce your memory needs when running
    lots of different perl scripts.

    PersistentPerl group names are entities unto themselves. They are not
    associated with Unix groups, or with the Group directive in Apache.
    Expect for the two special group names "none" and "default", all group
    names are created by the user of PersistentPerl using the Group option
    described in the section on "OPTIONS"

    If you want the maximum amount of grouping possible then you should run
    all scripts with the group option set to "default". This the group name
    used if you just specify "-g" on the command line without an explicit
    group name. When you do this, you will get the fewest number of perl
    interpreters possible - any perl interpreter will be able to run any of
    your perl scripts.

    Although using group "default" for all scripts results in the most
    efficient use of resources, it's not always possible or desirable to do
    this. You may want to use other group names for the following reasons:

    * To isolate misbehaving scripts, or scripts that don't work in groups.
        Some scripts won't work in groups. When perl scripts are grouped
        together they are each given their own unique package name - they
        are not run out of the "main" package as they normally would be. So,
        for example, a script that explicitly uses "main" somewhere in its
        code to find its subroutines or variables probably won't work in
        groups. In this case, it's probably best to run such a script with
        group "none", so it's compiled and run out of package main, and
        always given its own interpreter.

        In other cases, scripts may make changes to included packages, etc,
        that may break other scripts running in the same interpreter. In
        this case such scripts can be given their own group name (like
        "pariah") to keep them away from scripts they are incompatible with.
        The rest of your scripts can then run out of group "default". This
        will ensure that the "pariah" scripts won't run within the same
        interpreter as the other scripts.

    * To pass different perl or PersistentPerl parameters to different scripts.
        You may want to use separate groups to create different policies for
        different scripts.

        For example, you may have an email application that contains ten
        perl scripts, and since the common perl code used in this
        application has a bad memory leak, you want to use a MaxRuns setting
        of 5 for all of these scripts. You want to run all your other
        scripts with a normal MaxRuns setting. To accomplish this you can
        edit the ten email application scripts, and at the top use the line:

            #!/usr/bin/perperl -- -gmail -r5

        In the rest of your perl scripts you can use:

            #!/usr/bin/perperl -- -g

        What this will do is put the ten email scripts into a group of their
        own (named "mail") and give them all the default MaxRuns value of 5.
        All other scripts will be put into the group named "default", and
        this group will have a normal MaxRuns setting.

DOWNLOADING
  Binaries

    Binaries for many OSes can be found at:

        http://daemoninc.com/PersistentPerl/download.html

  Source Code

    The standard source code distribution can be retrieved from any CPAN
    mirror or from:

        http://daemoninc.com/PersistentPerl/download.html
        http://www.cpan.org/modules/by-authors/id/H/HO/HORROCKS/

AUTHOR
        Sam Horrocks
        http://daemoninc.com
        sam@daemoninc.com

  Contributors

    A lot of people have helped out with code, patches, ideas, resources,
    etc. I'm sure I'm missing someone here - if so, please drop me an email.

    *   Gunther Birznieks

    *   Diana Eichert

    *   Takanori Kawai

    *   Robert Klep

    *   Marc Lehmann

    *   James McGregor

    *   Josh Rabinowitz

    *   Dave Parker

    *   Craig Sanders

    *   Joseph Wang

SEE ALSO
    perl(1), httpd(8), apxs(8).

MORE INFORMATION
  PersistentPerl Home Page

    http://daemoninc.com/PersistentPerl/

  Mailing List

    *   PersistentPerl users mailing list - persistentperl-
        users@lists.sourceforge.net. Archives and subscription information
        are at http://lists.sourceforge.net/lists/listinfo/persistentperl-
        users

    *   PersistentPerl announcements mailing list - persistentperl-
        announce@lists.sourceforge.net. Archives and subscription
        information are at
        http://lists.sourceforge.net/lists/listinfo/persistentperl-announce

  Bugs and Todo List

    Please report any bugs or requests for changes to the mailing list.

COPYRIGHT
    Copyright (C) 2003 Sam Horrocks

    This program is free software; you can redistribute it and/or modify it
    under the terms of the GNU General Public License as published by the
    Free Software Foundation; either version 2 of the License, or (at your
    option) any later version.

    This program is distributed in the hope that it will be useful, but
    WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General
    Public License for more details.

    You should have received a copy of the GNU General Public License along
    with this program; if not, write to the Free Software Foundation, Inc.,
    59 Temple Place - Suite 330, Boston, MA 02111-1307, USA.

    This product includes software developed by the Apache Software
    Foundation (http://www.apache.org/).
