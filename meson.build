project('notcurses', ['c', 'cpp'],
        default_options:
          ['c_std=gnu11',
           'cpp_std=gnu++17',
           'default_library=both',
           'c_args=-Wall -Wextra -W -Wshadow -Wformat -fexceptions',
           'cpp_args=-Wall -Wextra -W -Wshadow -Wformat -fexceptions'
          ],
        version: '2.2.4',
        license: 'Apache-2.0',
        meson_version: '>=0.57.1')

fs = import('fs')

cc = meson.get_compiler('c')
cxx = meson.get_compiler('cpp')

deps = []

add_global_arguments('-Og', language: 'cpp')
add_global_arguments('-Og', language: 'c')
add_global_arguments('-D_FORTIFY_SOURCE=2', language: 'cpp')
add_global_arguments('-D_FORTIFY_SOURCE=2', language: 'c')

multimedia_backend = get_option('use_multimedia')

message('Requested multimedia backend: ' + multimedia_backend)
message('Requested build type: ' + get_option('buildtype'))

# Because everything and it's mother requires threads...
threads_dep = dependency('threads')

deps += threads_dep
deps += dependency('readline')

# Due to some distros (*cough cough* alpine *cough cough*), terminfo sometimes
# gets sucked into curses. Accept either, and have mercy on us all.

terminfo = dependency('tinfo', required : false)

if not terminfo.found()
    deps += dependency('ncursesw')
else
    deps += terminfo
endif

if multimedia_backend == 'ffmpeg'
  deps += dependency('libavcodec')
  deps += dependency('libavformat')
  deps += dependency('libavutil')
  deps += dependency('libswscale')
elif multimedia_backend == 'oiio'
  deps += dependency('OpenImageIO')
endif

deps += cc.find_library('m', required : false)

if get_option('use_doctest')
  deps += dependency('doctest')
endif

if not cc.has_header('uniwbrk.h')
  error('Could not find uniwbrk.h from GNU libunistring. STOP.')
else
  deps += cc.find_library('unistring', required : true)
endif

if get_option('use_qrcodegen')
  deps += cc.find_library('libqrcodegen', required : true)
endif

deps += cc.find_library('rt')

compatsrc = files(['src/compat/compat.c'])

################################################################################
# libnotcurses-core

nccore_src = files(['src/lib/blit.c',
                    'src/lib/debug.c',
                    'src/lib/fade.c',
                    'src/lib/fd.c',
                    'src/lib/fill.c',
                    'src/lib/input.c',
                    'src/lib/kitty.c',
                    'src/lib/layout.c',
                    'src/lib/linux.c',
                    'src/lib/menu.c',
                    'src/lib/metric.c',
                    'src/lib/notcurses.c',
                    'src/lib/plot.cpp',
                    'src/lib/progbar.c',
                    'src/lib/reader.c',
                    'src/lib/reel.c',
                    'src/lib/render.c',
                    'src/lib/selector.c',
                    'src/lib/signal.c',
                    'src/lib/sixel.c',
                    'src/lib/sprite.c',
                    'src/lib/stats.c',
                    'src/lib/terminfo.c',
                    'src/lib/tree.c',
                    'src/lib/tabbed.c',
                    'src/lib/visual.c',
                    'src/lib/direct.c'])

nccore_include = include_directories('include', 'src')

nccore_compdefs = []

if host_machine.system() == 'linux'
  nccore_compdefs += ['-D_GNU_SOURCE', '-D_DEFAULT_SOURCE']
endif

nccore_lib = library('notcurses-core',
                     nccore_src + compatsrc,
                     c_args: nccore_compdefs,
                     include_directories : nccore_include,
                     dependencies : deps,
                     install : true)


notcurses_core_dep = declare_dependency(include_directories : 'include',
                                        link_with : nccore_lib)


################################################################################
# libnotcurses

nc_src = files(['src/media/ffmpeg.cpp',
                'src/media/none.cpp',
                'src/media/oiio-indep.c',
                'src/media/oiio.cpp',
                'src/media/shim.c'])

nc_include = [ nccore_include , include_directories('src/lib') ]

nc_lib = library('notcurses',
                 nc_src + compatsrc,
                 include_directories : nc_include,
                 dependencies : deps + [notcurses_core_dep],
                 install : true)

notcurses_dep = declare_dependency(include_directories : 'include',
                                   link_with : [ nc_lib , nccore_lib ])


################################################################################
# libnotcurses++ (C++ wrappers)

ncpp_src = files(['src/libcpp/FDPlane.cc',
                  'src/libcpp/Menu.cc',
                  'src/libcpp/MultiSelector.cc',
                  'src/libcpp/NotCurses.cc',
                  'src/libcpp/Plane.cc',
                  'src/libcpp/Plot.cc',
                  'src/libcpp/Reel.cc',
                  'src/libcpp/Root.cc',
                  'src/libcpp/Selector.cc',
                  'src/libcpp/Subproc.cc',
                  'src/libcpp/Tablet.cc',
                  'src/libcpp/Utilities.cc'])

ncpp_lib = library('notcurses++',
                   ncpp_src,
                   cpp_args : ['-Werror=format-security',
                               '-Wnull-dereference',
                               '-Wunused',
                               '-Wno-c99-extensions',
                               '-fno-strict-aliasing',
                               '-ffunction-sections',
                               '-fstack-protector',
                               '-fno-rtti'],
                   pic : true,
                   include_directories : 'include',
                   dependencies : deps + [ notcurses_core_dep , notcurses_dep ],
                   install : true)

ncpp_dep = declare_dependency(include_directories : 'include',
                              link_with : [ nc_lib , nccore_lib , ncpp_lib ])

################################################################################
# notcurses-demo

ncdemo_src = files(['src/demo/animate.c',
                    'src/demo/boxdemo.c',
                    'src/demo/chunli.c',
                    'src/demo/demo.c',
                    'src/demo/dragon.c',
                    'src/demo/eagle.c',
                    'src/demo/fallin.c',
                    'src/demo/grid.c',
                    'src/demo/highcon.c',
                    'src/demo/hud.c',
                    'src/demo/input.c',
                    'src/demo/intro.c',
                    'src/demo/jungle.c',
                    'src/demo/keller.c',
                    'src/demo/luigi.c',
                    'src/demo/mojibake.c',
                    'src/demo/normal.c',
                    'src/demo/outro.c',
                    'src/demo/qrcode.c',
                    'src/demo/reel.c',
                    'src/demo/sliding.c',
                    'src/demo/trans.c',
                    'src/demo/unicodeblocks.c',
                    'src/demo/view.c',
                    'src/demo/whiteout.c',
                    'src/demo/xray.c',
                    'src/demo/yield.c',
                    'src/demo/zoo.c']) + compatsrc

ncdemo_include = include_directories('src', 'include')

ncdemo_deps = [notcurses_dep, cc.find_library('m', required : false), dependency('threads')]

ncdemo = executable('notcurses-demo',
                    ncdemo_src,
                    dependencies : ncdemo_deps,
                    include_directories : ncdemo_include,
                    c_args : '-D_GNU_SOURCE',
                    install : true)

################################################################################
# Configure files

configure_file(input : 'tools/version.h.in',
               output : 'version.h',
               configuration : {'notcurses_VERNUM_MAJOR': 2,
                                'notcurses_VERNUM_MINOR': 2,
                                'notcurses_VERNUM_PATCH': 4,
                                'notcurses_VERNUM_TWEAK': '',
                                'notcurses_VERSION_MAJOR': 2,
                                'notcurses_VERSION_MINOR': 2,
                                'notcurses_VERSION_PATCH': 4,
                                'notcurses_VERSION_TWEAK': ''})

builddefs = configuration_data({
                 'DFSG_BUILD': get_option('dfsg'),
                 'USE_QRCODEGEN': get_option('use_qrcodegen')})

builddefs.set_quoted('NOTCURSES_SHARE', get_option('datadir'))

if multimedia_backend == 'ffmpeg'
  builddefs.set('USE_FFMPEG', true)
  builddefs.set('NOTCURSES_USE_MULTIMEDIA', true)
elif multimedia_backend == 'oiio'
  builddefs.set('USE_OIIO', true)
  builddefs.set('NOTCURSES_USE_MULTIMEDIA', true)
endif

configure_file(
  output : 'builddef.h',
  configuration : builddefs,
)


################################################################################
# testing

if get_option('use_doctest')

  nctest_src = files(['src/tests/blit.cpp',
                      'src/tests/cell.cpp',
                      'src/tests/channel.cpp',
                      'src/tests/direct.cpp',
                      'src/tests/egcpool.cpp',
                      'src/tests/Exceptions.cpp',
                      'src/tests/fade.cpp',
                      'src/tests/fds.cpp',
                      'src/tests/fills.cpp',
                      'src/tests/geom.cpp',
                      'src/tests/input.cpp',
                      'src/tests/internal.cpp',
                      'src/tests/layout.cpp',
                      'src/tests/libunistring.cpp',
                      'src/tests/main.cpp',
                      'src/tests/menu.cpp',
                      'src/tests/metric.cpp',
                      'src/tests/Ncpp.cpp',
                      'src/tests/notcurses.cpp',
                      'src/tests/palette.cpp',
                      'src/tests/piles.cpp',
                      'src/tests/pixel.cpp',
                      'src/tests/plane.cpp',
                      'src/tests/plot.cpp',
                      'src/tests/progbar.cpp',
                      'src/tests/reader.cpp',
                      'src/tests/reel.cpp',
                      'src/tests/reelgaps.cpp',
                      'src/tests/resize.cpp',
                      'src/tests/rotate.cpp',
                      'src/tests/scrolling.cpp',
                      'src/tests/selector.cpp',
                      'src/tests/stacking.cpp',
                      'src/tests/tabbed.cpp',
                      'src/tests/tree.cpp',
                      'src/tests/version.cpp',
                      'src/tests/visual.cpp',
                      'src/tests/wide.cpp',
                      'src/tests/zaxis.cpp'])

  nctest_include = include_directories('include', 'src', 'src/lib')

  nctest = executable('notcurses-tester',
                      nctest_src,
                      dependencies :
                        [ ncpp_dep,
                          terminfo,
                          cc.find_library('libunistring',
                                          required : true) ],
                      include_directories : nctest_include)

  # While the notcurses-tester *does* compile, and it *does* run if run
  # manually, it does *not* play nicely with meson's test setup, in comparsion
  # to ctest. However, for the purpose of completeness, it is here,
  # uncommentable if requested.

  # test('notcurses tester',
  #      nctest,
  #      args : ['-p', meson.project_build_root() / 'data'],
  #      is_parallel : false)

endif
