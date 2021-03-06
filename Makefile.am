NULL =

bin_PROGRAMS = $(NULL)
dist_installed_test_extra_scripts = $(NULL)
noinst_PROGRAMS = $(NULL)
noinst_LTLIBRARIES = $(NULL)
libexec_PROGRAMS = $(NULL)
CLEANFILES = $(NULL)
MAINTAINERCLEANFILES = $(NULL)
DISTCLEANFILES= $(NULL)
BUILT_SOURCES = $(NULL)
EXTRA_DIST =

include $(top_srcdir)/buildutil/glib-tap.mk

SUBDIRS =


if BUILD_DOCUMENTATION
SUBDIRS += . doc
endif

SUBDIRS += po

%.service: %.service.in config.log
	$(AM_V_GEN) $(SED) -e "s|\@libexecdir\@|$(libexecdir)|" \
		-e "s|\@localstatedir\@|$(localstatedir)|" \
		-e "s|\@extraargs\@||" $< > $@

dbus_servicedir = $(DBUS_SERVICE_DIR)
service_in_files = $(NULL)
dbus_service_DATA = $(NULL)
systemduserunit_DATA = $(NULL)
CLEANFILES += $(dbus_service_DATA)
CLEANFILES += $(systemduserunit_DATA)
EXTRA_DIST += $(service_in_files)

FLATPAK_BINDIR=$(bindir)

ACLOCAL_AMFLAGS = -I m4 -I libglnx ${ACLOCAL_FLAGS}
AM_CPPFLAGS =							\
	-DFLATPAK_BINDIR=\"$(FLATPAK_BINDIR)\"			\
	-DFLATPAK_SYSTEMDIR=\"$(SYSTEM_INSTALL_DIR)\"		\
	-DFLATPAK_CONFIGDIR=\"$(sysconfdir)/flatpak\"		\
	-DFLATPAK_BASEDIR=\"$(pkgdatadir)\"			\
	-DFLATPAK_TRIGGERDIR=\"$(pkgdatadir)/triggers\"		\
	-DSYSTEM_FONTS_DIR=\"$(SYSTEM_FONTS_DIR)\"		\
	-DSYSTEM_FONT_CACHE_DIRS=\"$(SYSTEM_FONT_CACHE_DIRS)\"		\
	-DG_LOG_DOMAIN=\"flatpak\"				\
	-I$(srcdir)/libglnx					\
	-I$(srcdir)/common					\
	-I$(builddir)/common					\
	-include "config.h"					\
	$(NULL)

AM_CFLAGS = $(WARN_CFLAGS)

if WITH_SYSTEM_BWRAP
AM_CPPFLAGS += -DHELPER=\"$(BWRAP)\"
else
AM_CPPFLAGS += -DHELPER=\"$(libexecdir)/flatpak-bwrap\"
endif

if WITH_SYSTEM_DBUS_PROXY
AM_CPPFLAGS += -DDBUSPROXY=\"$(DBUS_PROXY)\"
else
AM_CPPFLAGS += -DDBUSPROXY=\"$(libexecdir)/flatpak-dbus-proxy\"
endif

triggersdir = $(pkgdatadir)/triggers
dist_triggers_SCRIPTS = \
	triggers/gtk-icon-cache.trigger \
	triggers/mime-database.trigger \
	triggers/desktop-database.trigger \
	$(NULL)

# This canonicalizes the PKG_CHECK_MODULES or AM_PATH_GPGME results
INTERNAL_GPGME_CFLAGS = $(DEP_GPGME_CFLAGS) $(GPGME_PTHREAD_CFLAGS)
INTERNAL_GPGME_LIBS = $(DEP_GPGME_LIBS) $(GPGME_PTHREAD_LIBS)

lib_LTLIBRARIES =
noinst_LTLIBRARIES += libglnx.la
libglnx_srcpath := $(srcdir)/libglnx
libglnx_cflags := \
	$(BASE_CFLAGS) \
	"-I$(libglnx_srcpath)" \
	-std=gnu99 \
	$(HIDDEN_VISIBILITY_CFLAGS) \
	$(NULL)
libglnx_libs := $(BASE_LIBS)
include libglnx/Makefile-libglnx.am.inc

include common/Makefile.am.inc
include data/Makefile.am.inc
include app/Makefile.am.inc
include session-helper/Makefile.am.inc
include portal/Makefile.am.inc
include system-helper/Makefile.am.inc
include tests/Makefile.am.inc

if !WITH_SYSTEM_DBUS_PROXY
libexec_PROGRAMS+=flatpak-dbus-proxy
flatpak_dbus_proxy_SOURCES =		\
	dbus-proxy/flatpak-proxy.c	\
	dbus-proxy/flatpak-proxy.h	\
	dbus-proxy/dbus-proxy.c		\
	dbus-proxy/backport-autoptr.h	\
	$(NULL)
flatpak_dbus_proxy_LDADD = $(AM_LDADD) $(BASE_LIBS)
flatpak_dbus_proxy_CFLAGS = $(AM_CFLAGS) $(BASE_CFLAGS)
endif

if !WITH_SYSTEM_BWRAP

bwrap_PROGRAMS = flatpak-bwrap
flatpak_bwrap_SOURCES = $(bwrap_SOURCES)
flatpak_bwrap_CFLAGS = $(AM_CFLAGS) $(bwrap_CFLAGS)
flatpak_bwrap_LDADD = $(AM_LDADD) $(bwrap_LDADD) $(CAP_LIB)
bwrapdir = $(libexecdir)
include bubblewrap/Makefile-bwrap.am.inc

endif # !WITH_SYSTEM_BWRAP

# NOTE: bwrap is install-bwrapPROGS which is run from install-data, not install-exec, this data-hook is used
install-data-hook:
	$(MAKE) $(AM_MAKEFLAGS) install-test-data-hook
if !WITH_SYSTEM_BWRAP
if PRIV_MODE_SETUID
	$(SUDO_BIN) chown root $(DESTDIR)$(libexecdir)/flatpak-bwrap
	$(SUDO_BIN) chmod u+s $(DESTDIR)$(libexecdir)/flatpak-bwrap
endif # !PRIV_MODE_SETUID
endif # !WITH_SYSTEM_BWRAP

bashcompletiondir = $(datadir)/bash-completion/completions
bashcompletion_DATA = completion/flatpak
EXTRA_DIST += $(bashcompletion_DATA)

zshcompletiondir = $(datadir)/zsh/site-functions
zshcompletion_DATA = completion/_flatpak
EXTRA_DIST += $(zshcompletion_DATA)

profiledir = $(PROFILE_DIR)
profile_DATA = flatpak.sh
EXTRA_DIST += \
	profile/flatpak.sh.in \
	$(NULL)
DISTCLEANFILES += flatpak.sh

flatpak.sh: profile/flatpak.sh.in
	$(AM_V_GEN) $(SED) -e "s|\@localstatedir\@|$(localstatedir)|" \
		-e "s|\@sysconfdir\@|$(sysconfdir)|" $< > $@

envdir = $(datadir)/gdm/env.d
env_DATA = flatpak.env
EXTRA_DIST += env.d/flatpak.env.in
DISTCLEANFILES += flatpak.env

flatpak.env: env.d/flatpak.env.in
	$(AM_V_GEN) $(SED) -e "s|\@localstatedir\@|$(localstatedir)|" \
		-e "s|\@sysconfdir\@|$(sysconfdir)|" $< > $@

dbussnippetdir = $(systemduserunitdir)/dbus.service.d
dbussnippet_DATA = flatpak.conf
EXTRA_DIST += dbus.service.d/flatpak.conf.in
DISTCLEANFILES += flatpak.conf

flatpak.conf: dbus.service.d/flatpak.conf.in
	$(AM_V_GEN) $(SED) -e "s|\@localstatedir\@|$(localstatedir)|" \
		-e "s|\@sysconfdir\@|$(sysconfdir)|" $< > $@

pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = flatpak.pc
EXTRA_DIST += flatpak.pc.in

scriptsdir = $(bindir)
scripts_SCRIPTS = scripts/flatpak-bisect scripts/flatpak-coredumpctl
EXTRA_DIST += scripts/flatpak-bisect scripts/flatpak-coredumpctl

EXTRA_DIST += README.md

EXTRA_DIST += flatpak.png

AM_DISTCHECK_CONFIGURE_FLAGS =		\
	--enable-documentation		\
	--disable-maintainer-mode	\
	--enable-introspection

.PHONY: coverage lcov-clean genlcov

coverage:
	$(AM_V_GEN) $(MAKE) $(AM_MAKEFLAGS) lcov-clean
	$(AM_V_GEN) $(MAKE) check
	$(AM_V_GEN) $(MAKE) $(AM_MAKEFLAGS) genlcov

lcov-clean:
	$(AM_V_GEN) $(LCOV) --directory $(top_builddir) --zerocounters

genlcov:
	$(AM_V_GEN) $(LCOV) --directory $(top_builddir) --capture --output-file coverage.info
	$(AM_V_GEN) $(GENHTML) --prefix $(top_builddir) --output-directory coverage coverage.info
	$(AM_V_GEN)
