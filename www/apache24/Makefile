# $FreeBSD$

PORTNAME=	apache24
PORTVERSION=	2.4.3
CATEGORIES=	www ipv6
MASTER_SITES=	${MASTER_SITE_APACHE_HTTPD}
DISTNAME=	httpd-${PORTVERSION}
DIST_SUBDIR=	apache24

MAINTAINER?=	apache@FreeBSD.org
COMMENT?=	Version 2.4.x of Apache web server

LIB_DEPENDS=	expat:${PORTSDIR}/textproc/expat2 \
		apr-1:${PORTSDIR}/devel/apr1 \
		pcre:${PORTSDIR}/devel/pcre

MAKE_JOBS_SAFE=	yes

LATEST_LINK?=	apache24

CONFLICTS=	caudium14-1.* \
		apache-2.0.* apache-*-2.0.* \
		apache-*-2.2.* apache22-* \
		apache-*-2.4.* apache24-*

USE_APACHE=	common24
USE_BZIP2=	yes
USE_ICONV=	yes
USE_AUTOTOOLS=	autoconf libtool
USE_PERL5=	yes
USE_RC_SUBR=	apache24 htcacheclean
LIBTOOLFILES=	configure.in

USERS=		www
GROUPS=		www

# for slave ports
.if !defined(MASTERDIR)
APACHEDIR=	${.CURDIR}
.else
APACHEDIR=	${MASTERDIR}
.endif

WITH_HTTP_PORT?=	80

.include "${APACHEDIR}/Makefile.options"
.include <bsd.port.options.mk>
.include "${APACHEDIR}/Makefile.doc"

APR_CONFIG?=		${LOCALBASE}/bin/apr-1-config
APU_CONFIG?=		${LOCALBASE}/bin/apu-1-config

# APU modules used by AUTHN_DBD DBD
APU_DBD_MYSQL?=		${LOCALBASE}/lib/apr-util-1/apr_dbd_mysql.so
APU_DBD_PGSQL?=		${LOCALBASE}/lib/apr-util-1/apr_dbd_pgsql.so
APU_DBD_SQLITE3?=	${LOCALBASE}/lib/apr-util-1/apr_dbd_sqlite3.so
# APU module used by AUTHNZ_LDAP LDAP
APU_LDAP?=		${LOCALBASE}/lib/apr-util-1/apr_ldap.so
# APU module used by SESSION_CRYPTO
APU_CRYPTO_OPENSSL?=	${LOCALBASE}/lib/apr-util-1/apr_crypto_openssl.so

CONFIGURE_ARGS=	--prefix=${PREFIX_RELDEST} \
		--enable-layout=FreeBSD \
		--with-perl=${PERL5} \
		--with-port=${WITH_HTTP_PORT} \
		--with-expat=${LOCALBASE} \
		--with-iconv=${LOCALBASE} \
		--enable-http \
		--with-pcre=${LOCALBASE} \
		--with-apr=${APR_CONFIG} \
		--with-apr-util=${APU_CONFIG}

CONFIGURE_ENV=	\
		CONFIG_SHELL="${SH}" \
		LOCALBASE="${LOCALBASE}"

PREFIX_RELDEST=	${PREFIX:S,^${DESTDIR},,}

MAKE_ENV+=	EXPR_COMPAT=yes \
		INSTALL_MAN="${INSTALL_MAN}"

#=====================================================
# CONFIGURE_ARGS willl be handled in Makefile.modules,
# here we do only OPTIONS fixups

.if ${PORT_OPTIONS:MSSL}
USE_OPENSSL=	yes
.endif

.if ${PORT_OPTIONS:MSOCACHE_DC}
LIB_DEPENDS+=	distcache:${PORTSDIR}/security/distcache
.endif

.if ${PORT_OPTIONS:MLUA}
USE_LUA=	yes
.endif

.if ${PORT_OPTIONS:MSUEXEC_RSRCLIMIT}
EXTRA_PATCHES+=	${FILESDIR}/extra-patch-suexec_rsrclimit
.  if empty(PORT_OPTIONS:MSUEXEC)
IGNORE=		suEXEC resource limit patch requires mod_suexec.\
		Please (re)run 'make config' and choose SUEXEC option also
.  endif
.endif

.if ${PORT_OPTIONS:MSUEXEC_USERDIR}
EXTRA_PATCHES+= ${FILESDIR}/extra-patch-suexec_userdir
.  if empty(PORT_OPTIONS:MSUEXEC)
IGNORE=		suEXEC UserDir patch requires mod_suexec.\
		Please (re)run 'make config' and choose SUEXEC option also
.  endif
.endif

.include <bsd.port.pre.mk>
.include "${APACHEDIR}/Makefile.modules"

pre-configure::
	@${ECHO_MSG}	""
	@${ECHO_MSG}	"  You can check your modules configuration by using make show-modules"
	@${ECHO_MSG}	""

show-options:
	@${SED} -ne 's/^##//p' ${APACHEDIR}/Makefile.doc

post-patch:
	@${REINPLACE_CMD} -e 's|freebsd5|freebsd|' \
		${WRKSRC}/configure.in ${WRKSRC}/configure
	@${RM} -f ${WRKSRC}/docs/docroot/*.bak
	@${REINPLACE_CMD} -e 's," PLATFORM ",FreeBSD,' ${WRKSRC}/server/core.c
	@${INSTALL_DATA} ${WRKSRC}/NOTICE ${WRKSRC}/docs/manual

post-configure:
	@FTPUSERS=`${EGREP} -v '^#' /etc/ftpusers| ${TR} -s "\n" " "` ;\
	${REINPLACE_CMD} -e "s,%%FTPUSERS%%,$${FTPUSERS}," ${WRKSRC}/docs/conf/extra/httpd-userdir.conf
	@${REINPLACE_CMD} -e "s,%%WWWOWN%%,${WWWOWN}," -e  "s,%%WWWGRP%%,${WWWGRP}," ${WRKSRC}/docs/conf/httpd.conf
	@${REINPLACE_CMD} -e "s,%%PREFIX%%,${PREFIX}," ${WRKSRC}/support/envvars-std

post-install:
	@${MKDIR} ${ETCDIR}/envvars.d
	@${MKDIR} ${ETCDIR}/Includes
	@${INSTALL_DATA} ${FILESDIR}/no-accf.conf ${ETCDIR}/Includes/

.if ${PORT_OPTIONS:MLOG_FORENSIC}
	@${INSTALL_SCRIPT} ${WRKSRC}/support/check_forensic ${PREFIX}/sbin
.endif

.if ${PORT_OPTIONS:MDOCS}
.  for dir in style/lang style/xsl/util style/xsl
	@[ -d ${DOCSDIR}/${dir}/ ] && ${TOUCH} ${DOCSDIR}/${dir}/.keepme
.  endfor
.endif
	@${CAT} ${PKGMESSAGE}

# maintainer only, check for new modules
modlist: extract
	@${AWK} '/: checking whether to enable mod_/ \
		{printf "%%%%%s%%%%libexec/apache24/%s.so\n", \
		toupper($$8), $$8}' ${WRKSRC}/configure \
		| ${TR} -d '"' \
		| ${GREP} -E -v 'MOD_(ECHO|EXAMPLE|HTTP|IDENT|ISAPI|SO)' \
		| ${SORT} -u

.include <bsd.port.post.mk>
