ACLOCAL_AMFLAGS = -I m4 ${ACLOCAL_FLAGS}

# Installation directories.
pkgconfigdir     = $(libdir)/pkgconfig
GEXIV2_publicdir = $(includedir)/gexiv2
girdir           = $(datadir)/gir-1.0
typelibdir       = $(datadir)/gir-1.0
py2gidir         = $(PYTHON2_GIDIR)
py3gidir         = $(PYTHON3_GIDIR)
vapidir          = $(datadir)/vala/vapi

DISTCHECK_CONFIGURE_FLAGS = \
	--enable-gtk-doc \
	--with-python2-girdir='$${libdir}/python2/dist-packages/gi/overrides' \
	--with-python3-girdir='$${libdir}/python3/dist-packages/gi/overrides' \
	--enable-introspection

# Extra package files. #
EXTRA_DIST = \
	@PACKAGE_NAME@.pc.in	\
	GExiv2.py				\
	@PACKAGE_NAME@.vapi		\
	gexiv2/gexiv2-version.m4 \
	$(TYPELIB_FILES) \
	test/sample-author-badencoding.jpg \
	test/gexiv2-dump.vala \
	test/python/gexiv2.py


# Installed Files #
pkgconfig_DATA  = @PACKAGE_NAME@.pc
lib_LTLIBRARIES = lib@PACKAGE_NAME@.la
if HAVE_INTROSPECTION
gir_DATA        = GExiv2-$(GIR_VERSION).gir
typelib_DATA    = GExiv2-$(GIR_VERSION).typelib
# No introspection, no .gi file for the Python module to call into
if HAS_PYTHON2
py2gi_DATA      = GExiv2.py
endif
if HAS_PYTHON3
py3gi_DATA      = GExiv2.py
endif
endif

vapi: gexiv2.vapi

if HAVE_INTROSPECTION
gexiv2.vapi: gexiv2_vapi.stamp
	@true

gexiv2_vapi.stamp: $(top_builddir)/GExiv2-0.10.gir $(abs_top_srcdir)/vapi/GExiv2-0.10.metadata $(abs_top_srcdir)/vapi/gexiv2-custom.vala
	rm -f $(@F)
	$(AM_V_GEN)$(VAPIGEN) --metadatadir=$(abs_top_srcdir)/vapi \
			      --library=gexiv2 \
			      $(top_builddir)/GExiv2-0.10.gir $(abs_top_srcdir)/vapi/gexiv2-custom.vala
	echo timestmap > $(@F)

dist_vapi_DATA = gexiv2.vapi
EXTRA_DIST += $(top_srcdir)/vapi/GExiv2-0.10.metadata gexiv2.vapi gexiv2_vapi.stamp $(top_srcdir)/vapi/gexiv2-custom.vala
DISTCLEANFILES = gexiv2.vapi gexiv2_vapi.stamp
endif

GEXIV2_GIRHEADERS = \
	$(srcdir)/gexiv2/gexiv2.h \
	$(srcdir)/gexiv2/gexiv2-metadata.h \
	$(srcdir)/gexiv2/gexiv2-preview-properties.h \
	$(srcdir)/gexiv2/gexiv2-preview-image.h \
	$(srcdir)/gexiv2/gexiv2-log.h \
	$(srcdir)/gexiv2/gexiv2-startup.h \
	gexiv2/gexiv2-version.h

GEXIV2_public_HEADERS = \
	$(srcdir)/gexiv2/gexiv2-managed-stream.h \
	$(GEXIV2_GIRHEADERS)

# Non-installed source files. #
GEXIV2_sources = \
	gexiv2/gexiv2-metadata.cpp \
	gexiv2/gexiv2-metadata-exif.cpp \
	gexiv2/gexiv2-metadata-gps.cpp \
	gexiv2/gexiv2-metadata-iptc.cpp \
	gexiv2/gexiv2-metadata-xmp.cpp \
	gexiv2/gexiv2-stream-io.cpp \
	gexiv2/gexiv2-preview-properties.cpp \
	gexiv2/gexiv2-preview-image.cpp \
	gexiv2/gexiv2-log.cpp \
	gexiv2/gexiv2-startup.cpp \
	gexiv2/gexiv2-log-private.h \
	gexiv2/gexiv2-metadata-private.h \
	gexiv2/gexiv2-stream-io.h \
	gexiv2/gexiv2-preview-properties-private.h \
	gexiv2/gexiv2-preview-image-private.h

# compilation setup. #

if PLATFORM_WIN32
no_undefined = -no-undefined
endif

# REQUIRED_CFLAGS absolutely get appended to CFLAGS, whatever the
# the value of CFLAGS in the environment.
REQUIRED_CXXFLAGS=

# Because -fPIC can interfere with compilation on 32-bit platforms but
# is absolutely necessary for AMD64, we check what the target machine's
# word size is, and set our required flags based upon that.
if WORD_64
REQUIRED_CXXFLAGS += -fPIC -DPIC
endif

AM_CXXFLAGS = $(REQUIRED_CXXFLAGS) -I$(top_builddir)/gexiv2 $(WARN_CXXFLAGS)

$(GEXIV2_sources): gexiv2/gexiv2-version.h

gexiv2/gexiv2-version.h: gexiv2/gexiv2-version.m4
	m4 '--define=_VERSION_MAJOR_=$(GEXIV2_MAJOR_VERSION)' '--define=_VERSION_MINOR_=$(GEXIV2_MINOR_VERSION)' '--define=_VERSION_MICRO_=$(GEXIV2_MICRO_VERSION)' $< > $@

lib@PACKAGE_NAME@_la_SOURCES  = $(GEXIV2_sources) $(GEXIV2_public_HEADERS)
lib@PACKAGE_NAME@_la_LIBADD   = $(EXIV2_LIBS) $(GLIB_LIBS)
lib@PACKAGE_NAME@_la_CPPFLAGS = $(EXIV2_CFLAGS) $(GLIB_CFLAGS)

lib@PACKAGE_NAME@_la_LDFLAGS  = \
	$(no_undefined) -export-dynamic -version-info $(GEXIV2_VERSION_INFO) \
	$(WARN_LDFLAGS)

clean-local:
	rm -f gexiv2/gexiv2-version.h
	rm -f gexiv2.pc
	rm -f test/gexiv2-dump
	rm -rf test/python/gi test/python/__pycache__

# Tests and utilities #

if ENABLE_TESTS

tests: test/gexiv2-dump test/python2 test/python3

test/gexiv2-dump: test/gexiv2-dump.vala
	valac -g --enable-checking --vapidir=. --pkg gexiv2 $< -o $@

test/python/gi/overrides/GExiv2.py:
	[ -d $(dir $@) ] || mkdir -p $(dir $@)
	ln -sf ../../../../GExiv2.py $@

test/python%: $(TYPELIB_FILES) test/python/gi/overrides/GExiv2.py
	LD_LIBRARY_PATH=.libs GI_TYPELIB_PATH=. PYTHONPATH=test/python python$* -m unittest gexiv2

endif

# Optional Introspection #

-include $(INTROSPECTION_MAKEFILE)
if HAVE_INTROSPECTION
INTROSPECTION_GIRS =
INTROSPECTION_SCANNER_ARGS = \
			     --symbol-prefix=@PACKAGE_NAME@		\
			     --add-include-path=$(BUILD_DIR) \
			     --pkg-export=gexiv2 \
			     $(WARN_SCANNERFLAGS)
INTROSPECTION_COMPILER_ARGS = --includedir=$(BUILD_DIR)

GExiv2-@GIR_VERSION@.gir: lib@PACKAGE_NAME@.la
GExiv2_@GIR_VERSION_U@_gir_INCLUDES = GObject-2.0
GExiv2_@GIR_VERSION_U@_gir_CFLAGS = -I$(srcdir) -I@PACKAGE_NAME@
GExiv2_@GIR_VERSION_U@_gir_LIBS = lib@PACKAGE_NAME@.la
GExiv2_@GIR_VERSION_U@_gir_FILES = $(GEXIV2_GIRHEADERS)
GExiv2_@GIR_VERSION_U@_gir_NAMESPACE = GExiv2
GExiv2_@GIR_VERSION_U@_gir_VERSION = $(GIR_VERSION)
INTROSPECTION_GIRS += GExiv2-$(GIR_VERSION).gir

CLEANFILES = $(gir_DATA) $(typelib_DATA)
endif
