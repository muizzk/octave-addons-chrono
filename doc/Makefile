# Makefile for Chrono documentation
#
# This setup uses the convention that "*.texi.in" files are the human-editable
# input files, and corresponding "*.texi" files are processed intermediate
# files.
#
# This Makefile is generic enough that it could be dropped without modification
# in the doc/ directory of any package that follows Octave Forge conventions.
# Just include its dependencies: mkdoc.pl, mkqhp.pl, mktexi.pl
# And make sure that your texinfo doc strings follow this' conventions:
#   - each @node must be followed immediately by a @chapter/@section/@subsection/@subsubsection

## Some basic tools (can be overriden using environment variables)
SED ?= sed
TAR ?= tar
GREP ?= grep
CUT ?= cut

## Note the use of ':=' (immediate set) and not just '=' (lazy set).
## http://stackoverflow.com/a/448939/1609556
package := $(shell $(GREP) "^Name: " ../DESCRIPTION | $(CUT) -f2 -d" ")
version := $(shell $(GREP) "^Version: " ../DESCRIPTION | $(CUT) -f2 -d" ")

DVIPS = dvips
LN_S = ln -s
OCTAVE = octave
PERL = perl

# On Mac, prefer Homebrew-installed Texinfo, because the system one is
# outdated. We need to explicitly pull it in because Homebrew does not 
# put it on the path by default.
BREW=$(shell which brew 2>/dev/null)
ifeq ($(BREW),)
	MAKEINFO = makeinfo
	TEXI2DVI = texi2dvi
	TEXI2HTML = makeinfo --html
	TEXI2PDF = texi2pdf
	# Ubuntu: qtchooser seems broken for qhelpgenerator, so /usr/bin/qhelpgenerator
	# doesn't work. Locate it directly. Prefer qcollectiongenerator because that separate
	# program is needed on Qt prior to Qt 5.12.
	ifneq ($(wildcard /usr/lib/x86_64-linux-gnu/qt5/bin/qcollectiongenerator),)
		QHELPGENERATOR = /usr/lib/x86_64-linux-gnu/qt5/bin/qcollectiongenerator
	else ifneq ($(wildcard /usr/lib/x86_64-linux-gnu/qt5/bin/qhelpgenerator),)
		QHELPGENERATOR = /usr/lib/x86_64-linux-gnu/qt5/bin/qhelpgenerator
	# The trailing space here is significant on Ubuntu Xenial! (GNU Make 4.1)
	else ifneq ($(shell which qcollectiongenerator 2>/dev/null ))
		QHELPGENERATOR = qcollectiongenerator
    else
		QHELPGENERATOR = qhelpgenerator
	endif
else
	BREW_TEXINFO = $(shell brew --prefix texinfo)
	MAKEINFO = $(BREW_TEXINFO)/bin/makeinfo
	TEXI2DVI = $(BREW_TEXINFO)/bin/texi2dvi
	TEXI2HTML = $(BREW_TEXINFO)/bin/makeinfo --html
	TEXI2PDF = $(BREW_TEXINFO)/bin/texi2pdf
	BREW_QT = $(shell brew --prefix qt)
	ifeq ($(wildcard $(BREW_QT)/bin/qhelpgenerator),)
		QHELPGENERATOR = $(BREW_QT)/bin/qcollectiongenerator
	else
		QHELPGENERATOR = $(BREW_QT)/bin/qhelpgenerator
	endif
endif

INFODOC = $(package).info
DVIDOC = $(patsubst %.info,%.dvi,$(INFODOC))
PSDOC = $(patsubst %.info,%.ps,$(INFODOC))
PDFDOC = $(patsubst %.info,%.pdf,$(INFODOC))
#HTMLDOC = $(patsubst %.info,%.html,$(INFODOC))
TEXIDOC = $(patsubst %.info,%.texi,$(INFODOC))
DOCSTRINGS = DOCSTRINGS.texi.tmp
INDEX = ../INDEX

default: all
all: info html pdf qhelp
info: $(INFODOC)
dvi: $(DVIDOC)
#html: $(HTMLDOC) $(package)-single.html
pdf: $(PDFDOC)
ps: $(PSDOC)
qhelp: $(package).qhc

%.dvi: %.texi
	$(TEXI2DVI) --clean -o $@ $<

%.info: %.texi
	$(MAKEINFO) --no-split -o $@ $<

%.pdf: %.texi
	$(TEXI2PDF) --clean -o $@ $<

%.ps: %.dvi
	$(DVIPS) -o $@ $<

$(package).qhp: $(package).texi mkqhp.pl
	$(PERL) mkqhp.pl $(package).texi $(package).qhp

$(package).qhc: $(package).qhcp $(package).qhp
	$(QHELPGENERATOR) $(package).qhcp -o $(package).qhc

$(package).html: $(package).texi
	rm -f $(package).html
	$(TEXI2HTML) --no-split -o $(package).html $<

html: $(package).texi $(package).html
	rm -rf html
	$(TEXI2HTML) -o html $<

.PRECIOUS: %.texi
%.texi : %.texi.in mkdoc.pl mktexi.pl
	$(RM) -f $(DOCSTRINGS); \
	$(PERL) ./mkdoc.pl ../ > $(DOCSTRINGS); \
	$(PERL) ./mktexi.pl $< $(DOCSTRINGS) $(INDEX) > $@ ; \
	$(RM) -f $(DOCSTRINGS);

# Auxiliary make file defines build rules for generated images for the manual
-include images.mk
images.mk: images.sh
	$(SHELL) $< > $@

$(DVIDOC): $(IMAGES_EPS)
$(PDFDOC): $(IMAGES_PDF)

HTMLDIR_IMAGES = $(addprefix $(HTMLDOC)/,$(IMAGES_PNG))
$(HTMLDIR_IMAGES): $(IMAGES_PNG) | $(HTMLDOC)
	cp $(@F) $@

html: $(HTMLDIR_IMAGES)

# The images included in the HTML manual must be present before the makeinfo
# command is invoked or it will fall back on incorrect file names.
$(HTMLDOC): $(IMAGES_PNG)

# The texi2dvi script (used to create both PDF and DVI output formats)
# uses some fixed temporary file names.  In order to avoid a race condition
# the DVI and PDF builds are forced to run serially through a Makefile rule.
$(PDFDOC): $(DVIDOC)

clean:
	rm -rf *.tmp 

maintainer-clean: clean
	rm -rf *.dvi *.eps *.html *.info *.pdf *.ps *.png *.texi *.qhp *.qch *.qhc images.mk

.PHONY: default all clean maintainer-clean dvi html info pdf ps qhelp
