# Created by: David O'Brien <obrien@cs.ucdavis.edu>

PORTNAME=	vim
PORTVERSION=	8.2.3570
DISTVERSIONPREFIX=	v
CATEGORIES=	editors

MAINTAINER=	adamw@FreeBSD.org
COMMENT=	Improved version of the vi editor (${FLAVOR:U} flavor)

LICENSE=	VIM
LICENSE_NAME=	VIM License
LICENSE_FILE=	${WRKSRC}/runtime/doc/uganda.txt
LICENSE_PERMS=	dist-mirror dist-sell pkg-mirror pkg-sell auto-accept

USES=		cpe iconv ncurses pkgconfig shebangfix
USE_GITHUB=	yes

# Vim will try to link against libcanberra if it's installed
CONFIGURE_ARGS=	--disable-canberra \
		--enable-cscope \
		--enable-gui=${GUI} \
		--enable-multibyte \
		--with-tlib=ncursesw
MAKE_ARGS=	STRIP="${STRIP_CMD}" \
		VIMRCLOC=${ETCDIR} \
		VIMRUNTIMEDIR=${PREFIX}/share/vim/${VIM_VER}

CPE_VERSION=	${PORTVERSION:R}
GNU_CONFIGURE=	yes
PLIST_SUB=	VIM_VER=${VIM_VER}
PORTDATA=	${VIM_VER}
PORTSCOUT=	ignore:1
SHEBANG_FILES=	runtime/tools/demoserver.py runtime/tools/efm_perl.pl
VIM_VER=	${PORTNAME}${PORTVERSION:R:S|.||g}

# Flavors {{{1
# FLAVORS are used to build various Vim GUIs. The default flavor (`vim') is
# console-only.
FLAVORS=	console athena gtk2 gtk3 motif tiny
FLAVOR?=	${FLAVORS:[1]}
# Register conflicts with all other flavors
CONFLICTS_INSTALL=	${FLAVORS:N${FLAVOR}:S/^/vim-/}

.for f in ${FLAVORS:Nconsole}
${f}_PKGNAMESUFFIX=	-${f}
.endfor

# GUI flavors
.if ${FLAVOR:U} == console
GUI=		no
CONFIGURE_ARGS+=--without-x
PLIST_SUB+=	GUI="@comment " DESKTOP="@comment "

.elif ${FLAVOR:U} == tiny
GUI=		no
CONFIGURE_ARGS+=--without-x
PLIST=		${.CURDIR}/pkg-plist-tiny

.elif ${FLAVOR:U} == athena
GUI=		athena
CONFIGURE_ARGS+=--enable-xim --enable-fontset
PLIST_SUB+=	GUI="" DESKTOP="@comment "
USES+=		xorg
USE_XORG+=	ice sm x11 xaw xmu xpm xt

.elif ${FLAVOR:U} == gtk2
GUI=		gtk2
CONFIGURE_ARGS+=--enable-xim --enable-fontset
PLIST_SUB+=	GUI="" DESKTOP=""
USES+=		desktop-file-utils gnome xorg
USE_GNOME=	gdkpixbuf2 gtk20
USE_XORG=	ice sm x11 xpm xt

.elif ${FLAVOR:U} == gtk3
GUI=		gtk3
CONFIGURE_ARGS+=--enable-xim --enable-fontset
PLIST_SUB+=	GUI="" DESKTOP=""
USES+=		desktop-file-utils gnome xorg
USE_GNOME=	cairo gdkpixbuf2 gtk30
USE_XORG=	ice sm x11 xpm xt

.elif ${FLAVOR:U} == motif
GUI=		motif
CONFIGURE_ARGS+=--enable-xim --enable-fontset --with-motif=lib="${MOTIFLIB}"
CONFIGURE_ENV+=	MOTIFHOME=${LOCALBASE}
PLIST_SUB+=	GUI="" DESKTOP="@comment "
USES+=		motif xorg
USE_XORG=	ice sm x11 xmu xt
.endif

# Options {{{1
# The bulk of the OPTIONS are for normal (non-tiny) vim
### Make sure any new options are excluded below in CONSOLE/TINY
OPTIONS_DEFAULT=	CTAGS_BASE DEFAULT_VIMRC PYTHON
OPTIONS_DEFINE=		DEFAULT_VIMRC MAKE_JOBS NLS XTERM_SAVE
OPTIONS_SINGLE=		CTAGS
OPTIONS_SINGLE_CTAGS=	CTAGS_BASE CTAGS_EXUBERANT CTAGS_UNIVERSAL
OPTIONS_GROUP=		LANGBIND
OPTIONS_GROUP_LANGBIND=	LUA PERL PYTHON RUBY SCHEME TCL
OPTIONS_SUB=		yes

# vim-tiny excludes everything except CONSOLE, DEFAULT_VIMRC, MAKE_JOBS
.if ${FLAVOR:U} == tiny
OPTIONS_EXCLUDE:=	${OPTIONS_DEFINE:NDEFAULT_VIMRC:NMAKE_JOBS} \
			${OPTIONS_SINGLE_UI:NCONSOLE} \
			${OPTIONS_GROUP_LANGBIND} \
			${OPTIONS_SINGLE_CTAGS}
.endif

DEFAULT_VIMRC_DESC=	Install bundled vimrc as default setting
MAKE_JOBS_DESC=		Enable parallel build
XTERM_SAVE_DESC=	Restore xterm screen after exit

CTAGS_BASE_DESC=	Use system ctags
CTAGS_EXUBERANT_DESC=	Use exctags instead of ctags
CTAGS_UNIVERSAL_DESC=	Use uctags instead of ctags

# BROKEN: ruby.h: error: wrong number of arguments specified for 'deprecated' attribute
OPTIONS_EXCLUDE_powerpc64=	RUBY
OPTIONS_EXCLUDE_powerpc64le=	RUBY

# GENERAL OPTIONS
CTAGS_EXUBERANT_RUN_DEPENDS=	exctags:devel/ctags
CTAGS_UNIVERSAL_RUN_DEPENDS=	uctags:devel/universal-ctags

MAKE_JOBS_VARS_OFF=	MAKE_JOBS_UNSAFE=yes

NLS_CONFIGURE_ENABLE=	nls
NLS_USES=		gettext

# LANGUAGE BINDINGS
LANGBIND_DESC=		Optional language bindings

LUA_CONFIGURE_ENABLE=	luainterp
LUA_CONFIGURE_WITH=	lua-prefix=${LOCALBASE}
LUA_USES=		lua

PERL_CONFIGURE_ENABLE=	perlinterp
PERL_USES=		perl5

PYTHON_USES=		python:3.6+
PYTHON_USES_OFF=	python:env
PYTHON_CONFIGURE_ENABLE=	python3interp
PYTHON_CONFIGURE_WITH=	python3-command=${PYTHON_CMD}

RUBY_CONFIGURE_ENABLE=	rubyinterp
RUBY_USE=		RUBY=yes

SCHEME_DESC=		MzScheme (Racket) bindings
SCHEME_CONFIGURE_ENABLE=mzschemeinterp
SCHEME_BUILD_DEPENDS=	mzscheme:lang/racket
SCHEME_RUN_DEPENDS=	${SCHEME_BUILD_DEPENDS}

TCL_CONFIGURE_ENABLE=	tclinterp
TCL_CONFIGURE_WITH=	tclsh=tclsh${TCL_VER}
TCL_USES=		tcl

# Targets {{{1
.include <bsd.port.pre.mk>

.if ${PORT_OPTIONS:MPYTHON} && !empty(WITH_DEBUG)
CFLAGS+=		-DPy_DEBUG -DPy_DEBUG_NO_PYMALLOC
.endif

post-extract:
	${INSTALL_DATA} ${FILESDIR}/vietnamese_viscii.vim ${WRKSRC}/runtime/keymap

post-patch:
	${ECHO_CMD} '#define SYS_VIMRC_FILE "'${ETCDIR}'/vimrc"' >> ${WRKSRC}/src/feature.h
	${ECHO_CMD} '#define SYS_GVIMRC_FILE "'${ETCDIR}'/gvimrc"' >> ${WRKSRC}/src/feature.h
	${REINPLACE_CMD} -e 's|-liconv|${ICONV_LIB}|g; s|/usr/local|${LOCALBASE}|g' \
		${WRKSRC}/src/auto/configure

post-configure:
# Needed when devel/libsysinfo is installed:
	${REINPLACE_CMD} -e '/^#define HAVE_SYSINFO/ d' ${WRKSRC}/src/auto/config.h

post-install-DEFAULT_VIMRC-on:
	${MKDIR} ${STAGEDIR}${ETCDIR}
	${INSTALL_DATA} ${FILESDIR}/vimrc ${STAGEDIR}${ETCDIR}/vimrc.sample

.if ${FLAVOR:U} == tiny
do-install:
	${MKDIR} ${STAGEDIR}${DATADIR}/${VIM_VER}
	${INSTALL_PROGRAM} ${WRKSRC}/src/vim ${STAGEDIR}${PREFIX}/bin
	${TOUCH} ${STAGEDIR}${DATADIR}/${VIM_VER}/defaults.vim
	${INSTALL_MAN} ${WRKSRC}/runtime/doc/vim.1 ${STAGEDIR}${MAN1PREFIX}/man/man1

.else  # flavor != tiny
post-install:
	${RM} ${STAGEDIR}${PREFIX}/bin/ex
	${RM} ${STAGEDIR}${PREFIX}/bin/view
.  for F in eview evim gview gvim gvimdiff rgview rgvim rview rvim vimdiff
	${LN} -sf vim ${STAGEDIR}${PREFIX}/bin/$F
.  endfor

# OPTIONS
post-patch-CTAGS_BASE-on:
	${FIND} ${WRKSRC}/runtime -name 'menu*.vim' | ${XARGS} ${GREP} -l 'ctags -R' | \
		${XARGS} ${REINPLACE_CMD} -Ee 's!ctags -R ?\.!${FIND} . -type f \\| ${XARGS} ctags!g'

post-patch-CTAGS_EXUBERANT-on:
	${FIND} ${WRKSRC}/runtime -name 'menu*.vim' | ${XARGS} ${GREP} -l 'ctags -R' | \
		${XARGS} ${REINPLACE_CMD} -e 's|ctags -R|exctags -R|g'

post-patch-CTAGS_UNIVERSAL-on:
	${FIND} ${WRKSRC}/runtime -name 'menu*.vim' | ${XARGS} ${GREP} -l 'ctags -R' | \
		${XARGS} ${REINPLACE_CMD} -e 's|ctags -R|uctags -R|g'

post-patch-LUA-on:
	${REINPLACE_CMD} -e 's|set dummy lua;|set dummy ${LUA_CMD};|g' ${WRKSRC}/src/auto/configure
	${REINPLACE_CMD} -e '/$$LUA_LIBS/ s|^|LUA_LIBS="-L${LUA_LIBDIR} -llua-${LUA_VER}";|' ${WRKSRC}/src/auto/configure
	${REINPLACE_CMD} -e '/LUA_CFLAGS=/s|".*"|"-I${LUA_INCDIR}"|' ${WRKSRC}/src/auto/configure

post-patch-XTERM_SAVE-on:
	${REINPLACE_CMD} -e '/FEAT_XTERM_SAVE/ s|.*|#define FEAT_XTERM_SAVE|' ${WRKSRC}/src/feature.h

post-install-NLS-off:
	${MKDIR} ${STAGEDIR}${DATADIR}/${VIM_VER}/lang
	${MKDIR} ${STAGEDIR}${DATADIR}/${VIM_VER}/keymap
	${INSTALL_DATA} ${WRKSRC}/runtime/keymap/* ${STAGEDIR}${DATADIR}/${VIM_VER}/keymap
.endif	#!defined(TINY)

.include <bsd.port.post.mk>
