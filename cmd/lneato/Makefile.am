# $Id$ $Revision$
## Process this file with automake to produce Makefile.in

pdfdir = $(pkgdatadir)/doc/pdf

if WITH_X
if WITH_WIN32
bin_PROGRAMS = lneato
else
bin_SCRIPTS = lneato
endif
endif

man_MANS = lneato.1
if ENABLE_MAN_PDFS
pdf_DATA = lneato.1.pdf

if HAVE_PS2PDF
pdf = $(pdf_DATA)
else
pdf = ERROR_NO_PS2PDF
endif
else
pdf =
endif

AM_CFLAGS = -DLNEATO
if WITH_WIN32
lneato_SOURCES = ${top_srcdir}/cmd/dotty/mswin32/dotty.c
lneato_LDADD = -luser32 -lgdi32 -lwinspool -lcomdlg32 -ladvapi32 -lshell32
else
lneato: ${top_srcdir}/cmd/lneato/lneato.sh
	cp ${top_srcdir}/cmd/lneato/lneato.sh lneato
	chmod +x lneato
endif

if ENABLE_MAN_PDFS
if HAVE_PS2PDF
lneato.1.pdf: lneato.1.ps
	@PS2PDF@ $<

lneato.1.ps: $(srcdir)/lneato.1
	@GROFF@ -Tps -man $< >$@
endif
endif

EXTRA_DIST = $(man_MANS) $(pdf) lneato.bsh lneato.ksh lneato.sh \
	mswin32 lneato.vcxproj*

DISTCLEANFILES = lneato $(pdf) lneato.1.ps
