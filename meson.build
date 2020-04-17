project('libjpeg', 'c', version : '2.0.4', license : 'bsd-like')

conf_data = configuration_data()
conf_data.set('SIZEOF_SIZE_T', size_t_size)

system_name  = build_machine.system()
is_windows   = system_name == 'windows'
is_cygwin    = system_name == 'cygwin'
is_dragonfly = system_name == 'dragonfly'
is_freebsd   = system_name == 'freebsd'
is_netbsd    = system_name == 'netbsd'
is_openbsd   = system_name == 'openbsd'
is_linux     = system_name == 'linux'
is_macos     = system_name == 'darwin'
is_ios       = system_name == 'ios'
is_android   = system_name == 'android'
is_unix      = is_cygwin or is_dragonfly or is_freebsd or is_netbsd or is_openbsd or is_linux or is_macos or is_ios or is_android
is_apple     = is_macos or is_ios

cc = meson.get_compiler('c')

require_simd    = get_option('require_simd')
with_12bit      = get_option('with_12bit') 
with_arith_dec  = get_option('with_arith_dec')
with_arith_enc  = get_option('with_arith_enc')
with_java       = get_option('with_java')
with_jpeg       = get_option('with_jpeg')
with_jpeg7      = get_option('with_jpeg7')
with_jpeg8      = get_option('with_jpeg8')
with_mem_srcdst = get_option('with_mem_srcdst')
with_simd       = get_option('with_simd')
with_turbojpeg  = get_option('with_turbojpeg')
force_inline    = get_option('force_inline')

version                    = '2.0.5'
version_number             = 2000004
bits_in_samples            = 8
d_arith_coding_supported   = false
c_arith_coding_supported   = false
mem_srcdst_supported       = false
mem_srcdst_functions       = ''
so_age                     = 2
jpeg_lib_version           = 62
force_so_version           = false
turbojpeg_so_major_version = 0
turbojpeg_so_version       = '0.2.0'
if with_12bit
    with_arith_dec = false
    with_arith_enc = false
    with_java      = false
    with_simd      = false
    with_turbojpeg = false

    bits_in_samples = 12
endif

conf_data.set('WITH_SIMD', 1)
conf_data.set('BITS_IN_JSAMPLE', bits_in_samples)

if get_option('default_library') == 'static'
    with_java = false
endif

if with_jpeg8 or with_jpeg7
    with_arith_enc = true
    with_arith_dec = true

    if with_jpeg8
        jpeg_lib_version = 80
    elif with_jpeg7
        jpeg_lib_version = 70
    endif
endif 

if with_arith_dec
    conf_data.set('D_ARITH_CODING_SUPPORTED', 1)
endif

if with_arith_enc
    conf_data.set('C_ARITH_CODING_SUPPORTED', 1)
endif

if with_mem_srcdst
    conf_data.set('MEM_SRCDST_SUPPORTED', 1)
    conf_data.set('MEM_SRCDST_FUNCTIONS', 'global: jpeg_mem_dest; jpeg_mem_src;')
    so_age               = 3
endif

so_major_version = jpeg_lib_version
so_minor_version = 0
if jpeg_lib_version == 62
    so_major_version = jpeg_lib_version / 10
elif jpeg_lib_version == 80
    so_minor_version = 2
endif

conf_data.set('JPEG_LIB_VERSION', jpeg_lib_version)
conf_data.set('JPEG_LIB_VERSION_DECIMAL', jpeg_lib_version / 10)
conf_data.set('LIBJPEG_TURBO_VERSION', turbojpeg_so_major_version)
conf_data.set('LIBJPEG_TURBO_VERSION_NUMBER', turbojpeg_so_version)

size_t_size = cc.sizeof('size_t')
ul_size     = cc.sizeof('unsigned long')

if size_t_size == ul_size
    src = 'int main(int argc, char **argv) { unsigned long a = argc;  return __builtin_ctzl(a); }'

    if cc.compiles(src)
        conf_data.set('HAVE_BUILTIN_CTZL', 1)
    endif
endif

conf_data.set('SIZEOF_SIZE_T', size_t_size)

if cc.get_id() == 'msvc' and cc.has_header('intrin.h')
    conf_data.set('HAVE_INTRIN_H', 1)
endif

if is_unix
    if cc.has_header('locale.h')
        conf_data.set('HAVE_LOCALE_H', 1)
    endif 
    if cc.has_header('stddef.h')
        conf_data.set('HAVE_STDDEF_H', 1)
    endif 
    if cc.has_header('stdlib.h')
        conf_data.set('HAVE_STDLIB_H', 1)
    endif 
    if cc.has_header('sys/types.h')
        conf_data.set('NEED_SYS_TYPES_H', 1)
    endif 

    have_memset = cc.has_header_symbol('string.h', 'memset')
    have_memcpy = cc.has_header_symbol('string.h', 'memcpy')
    if have_memset
        conf_data.set('HAVE_MEMSET', 1)
    endif
    if have_memcpy
        conf_data.set('HAVE_MEMCPY', 1)
    endif

    if not have_memset and not have_memcpy
        conf_data.set('NEED_BSD_STRINGS', 1)
    endif

    uc_size = cc.sizeof('unsigned char')
    us_size = cc.sizeof('unsigned short')

    if not cc.compiles('int main(void) { typedef struct undefined_structure *undef_struct_ptr;  undef_struct_ptr ptr = 0;  return ptr != 0; }')
        conf_data.set('INCOMPLETE_TYPES_BROKEN', 1)
    endif

    right_shift_is_signed = false
    __char_unsigned__     = false
    if not meson.is_cross_build()
        check_right_shift_is_unsigned_src = ' \
            #include <stdio.h> \
            #include <stdlib.h> \
            int is_shifting_signed (long arg) { \
                long res = arg >> 4; \
                if (res == -0x7F7E80CL) \
                    return 1; /* right shift is signed */ \
                /* see if unsigned-shift hack will fix it. */ \
                /* we can\'t just test exact value since it depends on width of long... */ \
                res |= (~0L) << (32-4); \
                if (res == -0x7F7E80CL) \
                    return 0; /* right shift is unsigned */ \
                printf(\"Right shift isn\'t acting as I expect it to.\\\\n\"); \
                printf(\"I fear the JPEG software will not work at all.\\\\n\\\\n\"); \
                return 0; /* try it with unsigned anyway */ \
            } \
            int main (void) { \
                exit(is_shifting_signed(-0x7F7E80B1L)); \
            } \
        '
        right_shift_is_signed = cc.run(check_right_shift_is_unsigned_src).returncode() == 1
        __char_unsigned__     = cc.run('int main(void) { return ((char) -1 < 0); }').returncode() == 0
    endif

    if right_shift_is_signed
        conf_data.set('RIGHT_SHIFT_IS_UNSIGNED', 1)
    endif
    if __char_unsigned__
        conf_data.set('__CHAR_UNSIGNED__', 1)
    endif
endif

inline_keyword       = ''
force_inline_keyword = ''

if cc.compile('inline static foo(void) { return 0; }  int main(void) { return foo(); }')
    inline = 'inline'
elif cc.compile('__inline static foo(void) { return 0; }  int main(void) { return foo(); }')
    inline_keyword  = '__inline'
elif cc.compile('__inline__ static foo(void) { return 0; }  int main(void) { return foo(); }')
    inline_keyword = '__inline__'
else
    error('Could not determine how to inline functions.')
endif

if force_inline
    if cc.get_id() == 'msvc'
        force_inline_keyword = ' __forceinline'
    else 
        if cc.has_function_attribute('always_inline')
            force_inline_keyword = ' __attribute__((always_inline))'
        else
            error('Could not determine how to force inline functions.')
        endif
    endif
endif

inline = inline_keyword + ' ' + force_inline_keyword
conf_data.set('INLINE', inline)


if is_windows
    configure_file(input : 'win/jconfig.h.meson',
             output : 'jconfig.h',
             configuration : conf_data)
else
    configure_file(input : 'jconfig.h.meson',
             output : 'jconfig.h',
             configuration : conf_data)
endif

configure_file(input : 'jconfigint.h.meson',
            output : 'jconfigint.h',
            configuration : conf_data)
if is_unix
    configure_file(input : 'libjpeg.map.meson',
             output : 'libjpeg.map',
             configuration : conf_data)
endif

jpeg_sources = files([
    'jcapimin.c', 'jcapistd.c', 'jccoefct.c', 'jccolor.c', 'jcdctmgr.c', 'jchuff.c',
    'jcicc.c', 'jcinit.c', 'jcmainct.c', 'jcmarker.c', 'jcmaster.c', 'jcomapi.c', 'jcparam.c',
    'jcphuff.c', 'jcprepct.c', 'jcsample.c', 'jctrans.c', 'jdapimin.c', 'jdapistd.c', 'jdatadst.c',
    'jdatasrc.c', 'jdcoefct.c', 'jdcolor.c', 'jddctmgr.c', 'jdhuff.c', 'jdicc.c', 'jdinput.c',
    'jdmainct.c', 'jdmarker.c', 'jdmaster.c', 'jdmerge.c', 'jdphuff.c', 'jdpostct.c', 'jdsample.c',
    'jdtrans.c', 'jerror.c', 'jfdctflt.c', 'jfdctfst.c', 'jfdctint.c', 'jidctflt.c', 'jidctfst.c',
    'jidctint.c', 'jidctred.c', 'jquant1.c', 'jquant2.c', 'jutils.c', 'jmemmgr.c', 'jmemnobs.c'
])

if with_arith_dec or with_arith_enc
    jpeg_sources += files('jaricom.c')
    if with_arith_enc
        jpeg_sources += files('jcarith.c')
    endif

    if with_arith_dec
        jpeg_sources += files('jdarith.c')
    endif
endif

if with_simd

endif

if with_java 

endif

if 

suffixe = ''
if get_option('default_library') == 'static'
    suffixe = '-static'
endif

jpeg_inc = include_directories(['.'])
jpeg_lib = library(
    'jpeg' + suffixe,
    jpeg_sources,
    include_directories: jpeg_inc
)

jpeg_dep = declare_dependency(
    link_with: jpeg_lib, 
    include_directories: jpeg_inc
)