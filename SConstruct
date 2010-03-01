# -*- coding: utf-8 -*-

vars = Variables('.scons.vars', ARGUMENTS)

vars.AddVariables(
    PathVariable("prefix", "Installation prefix", "/usr",
                 PathVariable.PathAccept),
    ('AR', "The static archiver to use"),
    ('RANLIB', "The archive indexer to use"),
    ('CC', "The C compiler to use"),
    ('LINK', "The linker to use"),
    ('SHLINK', "The shared library linker to use"),
    ('CCFLAGS', "Any additional options to pass in to the C compiler"),
    ('CPPFLAGS', "Any additional options to pass in to the C preprocessor"),
    ('LINKFLAGS', "Any additional options to pass in to the linker"),
    ('SHLINKFLAGS', "Any additional options to pass in to the shared library linker"),
    )


root_env = Environment(tools=['default', 'packaging', 'scanreplace'],
                       package="libpush",
                       pkg_version="2.4.2",
                       CCFLAGS="-O2",
                       BINDIR = "$prefix/bin",
                       DOCDIR = "$prefix/share/doc/$package",
                       LIBDIR = "$prefix/lib",
                       INCLUDEDIR = "$prefix/include")

vars.Update(root_env)
vars.Save(".scons.vars", root_env)

root_env.MergeFlags("-g")

# An action that can clean up the scons temporary files.

if 'sdist' not in COMMAND_LINE_TARGETS:
    root_env.Clean("distclean",
                   [
                    ".sconf_temp",
                    ".sconsign.dblite",
                    ".scons.vars",
                    "config.log",
                   ])

# Only run the configuration steps if we're actually going to build
# something.

if not GetOption('clean') and not GetOption('help'):
    conf = root_env.Configure()

    if not conf.CheckCC():
        print "!! Your compiler and/or environment is not properly configured."
        Exit(0)


    SIZEOF_VOID_P = conf.CheckTypeSize("void *")
    if SIZEOF_VOID_P == 0:
        print "!! Cannot determine size of void *."
        Exit(0)

    SIZEOF_INT = conf.CheckTypeSize("int")
    if SIZEOF_INT == 0:
        print "!! Cannot determine size of void *."
        Exit(0)

    SIZEOF_LONG = conf.CheckTypeSize("long")
    if SIZEOF_LONG == 0:
        print "!! Cannot determine size of void *."
        Exit(0)

    root_env = conf.Finish()

    root_env.Append(CPPDEFINES={
                        'SIZEOF_VOID_P': SIZEOF_VOID_P,
                        'SIZEOF_INT': SIZEOF_INT,
                        'SIZEOF_LONG': SIZEOF_LONG,
                    })


# Include the subdirectory SConscripts

root_env.Prepend(CPPPATH=["#/cudd", "#/epd", "#/mtr", "#/st", "#/util"])

shared_files = []
static_files = []

Export('root_env shared_files static_files')
SConscript([
            'cudd/SConscript',
            'dddmp/SConscript',
            'epd/SConscript',
            'mtr/SConscript',
            'st/SConscript',
            'util/SConscript',
           ])


libcudd_shared = root_env.SharedLibrary("libcudd", shared_files)
root_env.Alias("shared", libcudd_shared)
root_env.Alias("install", root_env.Install("$LIBDIR", libcudd_shared))
Default(libcudd_shared)

libcudd_static = root_env.StaticLibrary("libcudd", static_files)
root_env.Alias("static", libcudd_static)
root_env.Alias("install", root_env.Install("$LIBDIR", libcudd_static))
Default(libcudd_static)

pc_file = root_env.ScanReplace('cudd.pc.in')
Default(pc_file)
root_env.Alias("install", root_env.Install("$LIBDIR/pkgconfig", pc_file))
