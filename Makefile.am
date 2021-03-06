## Makefile.am
##
## Copyright (C) 2010-2020 Free Software Foundation, Inc.
##
## This file is part of LibreDWG.
##
## LibreDWG is free software: you can redistribute it and/or modify it
## under the terms of the GNU General Public License as published by
## the Free Software Foundation, either version 3 of the License, or
## (at your option) any later version.
##
## LibreDWG is distributed in the hope that it will be useful,
## but WITHOUT ANY WARRANTY; without even the implied warranty
## of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
## See the GNU General Public License for more details.
##
## You should have received a copy of the GNU General Public License
## along with this program.  If not, see <http://www.gnu.org/licenses/>.

ACLOCAL_AMFLAGS = -I m4

SUBDIRS = src programs examples test doc
if HAVE_SWIG_PYTHON
if !DISABLE_BINDINGS
SUBDIRS += bindings
endif
endif

include_HEADERS = \
	include/dwg.h \
	include/dwg_api.h

pcdatadir = $(libdir)/pkgconfig
pcdata_DATA = libredwg.pc
BUILT_SOURCES = $(top_srcdir)/.version
LTEXEC = @LTEXEC@

VALGRIND_SUPPRESSIONS = @VALGRIND_SUPPRESSIONS@
VALGRIND_SUPPRESSIONS_FILE = @VALGRIND_SUPPRESSIONS_FILE@
VALGRIND_OPTS = --leak-check=full --show-reachable=no --num-callers=30
JSON_DIST = jsmn/jsmn.h jsmn/LICENSE jsmn/README.md
doc_DIST = README HACKING NEWS TODO
EXTRA_DIST = $(top_srcdir)/.version libredwg.spec \
	     dwg.in dxf.in dxf-allcvt.lsp dxf-allcvt.sh test-dxf.sh \
	     log.in logs-all-parallel.sh.in logs-all-serial.sh.in unknown.sh \
	     build-aux/swig_python.patch \
             $(JSON_DIST) \
	     $(VALGRIND_SUPPRESSIONS_FILE)

.PHONY: check-dwg check-dxf check-json check-dwg-valgrind \
        regen-man man manual refman scan-build gcov \
	unknown regen-unknown

UNKNOWN_LOG = unknown-`git describe --long --tags --dirty --always`.log
UNKNOWN_SKIP = examples/alldwg.skip
TIMEOUT_30 = @TIMEOUT_30@

unknown:
	-mv examples/alldwg.skip examples/alldwg.skip~
	$(MAKE) -C src bits.lo && \
	  $(MAKE) -C examples unknown$(EXEEXT) bd$(EXEEXT) bits$(EXEEXT) && \
	  $(LTEXEC) examples/unknown$(EXEEXT) >$(UNKNOWN_LOG) 2>$(UNKNOWN_SKIP)
	if test -s examples/alldwg.skip; then true; \
	else mv examples/alldwg.skip~ examples/alldwg.skip; fi
if HAVE_PICAT
if HAVE_PARALLEL
	-rm examples/*.pi.tmplog 2>/dev/null
	echo $(PARALLEL) $(TIMEOUT_30) $(PICAT) "{} \\>{}.tmplog ::: examples/[A-Z]*.pi"
	-$(PARALLEL) $(TIMEOUT_30) $(PICAT) {} \>{}.tmplog ::: examples/[A-Z]*.pi
	-for f in examples/*.pi.tmplog; do \
	   if test -n "`grep 'Most probable result:' $$f`"; then \
	     mv $$f examples/`basename $$f .tmplog`.log; fi; done
	-rm examples/*.pi.tmplog 2>/dev/null
else
	-for f in examples/[A-Z]*.pi; do \
	  echo $(TIMEOUT_30) picat $$f; \
	  $(TIMEOUT_30) $(PICAT) $$f >$$f.tmplog; \
	  if test -n "`grep 'Most probable result:' $$f.tmplog`"; then \
	    mv $$f.tmplog $$f.log; \
	  fi; \
	done; rm examples/[A-Z]*.pi.tmplog 2>/dev/null
endif
endif

regen-dynapi: $(srcdir)/src/gen-dynapi.pl $(srcdir)/include/dwg.h
	-$(PERL) $(srcdir)/src/gen-dynapi.pl
regen-unknown:
	$(MAKE) -C examples $@
regen-man:
	$(MAKE) -C programs $@
man:
	$(MAKE) -C programs $@
manual:
	$(MAKE) -C doc $@
refman:
	$(MAKE) -C doc $@
regen-swig-patch:
	$(MAKE) -C bindings/python
	-difforig bindings/python/swig_python.c >xxxx
	if test -s xxxx; then mv xxxx build-aux/swig_python.patch; fi

check-dwg: all
	-mv check-dwg.log check-dwg.log~ 2>/dev/null
	for d in `find $(srcdir)/test/test-data -iname \*.dwg -size -40M`; do \
	  echo programs/dwgread -v3 $$d | tee -a check-dwg.log; \
	  $(LTEXEC) programs/dwgread -v3 $$d >> check-dwg.log 2>&1; \
	done
check-dwg-valgrind: all
	-mv check-dwg-valgrind.log check-dwg-valgrind.log~ 2>/dev/null
	for d in `find $(srcdir)/test/test-data -iname \*.dwg -size -10M`; do \
	  echo $(VALGRIND) $(VALGRIND_OPTS) $(VALGRIND_SUPPRESSIONS) programs/dwgread $$d | tee -a check-dwg-valgrind.log; \
	  $(VALGRIND) $(VALGRIND_OPTS) $(VALGRIND_SUPPRESSIONS) $(LTEXEC) programs/dwgread $$d >> check-dwg-valgrind.log 2>&1; \
	done
check-dxf: all
	-for f in $(srcdir)/test/test-data/Drawing_*.dwg \
                  $(srcdir)/test/test-data/sample_*.dwg \
                  $(srcdir)/test/test-data/example_*.dwg; \
          do \
            env MAKEFLAGS= $(srcdir)/dxf-check $$f; \
          done
	-for d in r14 2000 2004 2007 2010 2013 2018; do \
          for f in $(srcdir)/test/test-data/$$d/*.dwg; do \
            env MAKEFLAGS= $(srcdir)/dxf-check $$f; \
	  done; \
	done
check-json: all
	-for f in $(srcdir)/test/test-data/Drawing_*.dwg \
                  $(srcdir)/test/test-data/sample_*.dwg \
                  $(srcdir)/test/test-data/example_*.dwg; \
          do \
            env MAKEFLAGS= $(srcdir)/json-check $$f; \
          done
	-for d in r14 2000 2004 2007 2010 2013 2018; do \
          for f in $(srcdir)/test/test-data/$$d/*.dwg; do \
            env MAKEFLAGS= $(srcdir)/json-check $$f; \
	  done; \
	done

# clang-analyzer.llvm.org, debian: clang-tools-{6.0,7}, redhat: clang-analyzer
SCAN_BUILD = scan-build
scan-build: clean
	$(SCAN_BUILD) -V -o .analysis $(MAKE) -j4 &

if ENABLE_GCOV
gcov: check
	$(MAKE) -C test gcov
	$(MAKE) -C src gcov
	$(MAKE) -C programs gcov
	$(MAKE) -C examples gcov
	gcov2perl test/*/*.gcov src/*.gcov programs/*.gcov examples/*.gcov
	cover -no-gcov
endif

$(BUILT_SOURCES):
	echo $(VERSION) > $@-t && mv $@-t $@
dist-hook:
	echo $(VERSION) > $(distdir)/.tarball-version

CLEANFILES = check-dwg.log check-dwg-valgrind.log check-dxf.log \
             check-dwg.log~ check-dwg-valgrind.log~ check-dxf.log~ \
	     *_20*.dxf *_r1*.dxf example_*.dxf.log
DISTCLEANFILES  = logs-all.sh dynapi_test_all.sh log json dwg dxf libredwg.pc
MAINTAINERCLEANFILES  = *~ *.log *.dxf *.json *.dwg cover_db .analysis

maintainer-clean-local:
	-rm -rf cover_db
LIBTOOL_DISTCLEAN_FILES = \
	m4/libtool.m4 m4/ltoptions.m4 m4/ltsugar.m4 m4/ltversion.m4 m4/lt~obsolete.m4 \
	build-aux/compile \
	build-aux/config.guess \
	build-aux/config.sub \
	build-aux/depcomp \
	build-aux/install-sh \
	build-aux/ltmain.sh \
	build-aux/missing \
	build-aux/py-compile \
	build-aux/textinfo.tex

# cannot uninstall the generated perllocal.pod and share/info/dir
distuninstallcheck:
	@:
