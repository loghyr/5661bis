# Copyright (C) The IETF Trust (2011-2012)
# Manage the .xml for the NFSv4 minor version document.
#

YEAR=`date +%Y`
MONTH=`date +%B`
DAY=`date +%d`
PREVVERS=00
VERS=00
VPATH=../../${DOC_PREFIX}.d

XML2RFC=xml2rfc

DRAFT_BASE=draft-ietf-nfsv4-rfc5661bis-dot-x
DOC_PREFIX=dotx

all: txt filelist

txt: ${DRAFT_BASE}-$(VERS).txt

${DOC_PREFIX}:
	cd ../${DOC_PREFIX}.d ; VERS=$(SPECVERS) make all

xml: ${DRAFT_BASE}-$(VERS).xml

clobber:
	$(RM) ${DRAFT_BASE}-$(VERS).txt \
		${DRAFT_BASE}-$(VERS).nr

clean:
	rm -f $(AUTOGEN)
	rm -rf autogen
	rm -f ${DRAFT_BASE}-$(VERS).xml
	rm -rf draft-$(VERS)
	rm -f draft-$(VERS).tar.gz
	rm -rf testx.d
	rm -f tmp.out tmp.sh tmp.x tmp.txt tmp.filelist
	cd ../${DOC_PREFIX}.d ; make clean

# Parallel All
pall: 
	make xml
	( make txt ; echo .txt done ) & \
	wait

tmp.x:	tmp.txt
	rm -f tmp.x tmp.sh
	@# xml2rfc puts ^Ms at the end of each line.
	grep -w grep tmp.txt | grep -w sed | grep /// | tr -d "\r" > tmp.sh
	sh tmp.sh < tmp.txt > tmp.x

tmp.txt: ${DRAFT_BASE}-$(VERS).xml
	$(XML2RFC) --text ${DRAFT_BASE}-$(VERS).xml -o $@

${DRAFT_BASE}-$(VERS).txt: tmp.txt tmp.x
	rm -f tmp.out
	diff ../${DOC_PREFIX}.d/nfsv41.x tmp.x > tmp.out
	@if [ -s tmp.out ] ; then \
	  echo ; \
	  echo .x file in i-d differs from ../${DOC_PREFIX}.d/nfsv41.x. Aborting. ; \
	  echo ; \
	  exit 1 ; \
	fi
	rm -f tmp.out

	cp tmp.txt $@

${DRAFT_BASE}-$(VERS).nr: ${DRAFT_BASE}-$(VERS).xml
	$(XML2RFC) --nroff ${DRAFT_BASE}-$(VERS).xml -o $@

${DOC_PREFIX}_front_autogen.xml: ${DOC_PREFIX}_front.xml Makefile
	sed -e s/DAYVAR/${DAY}/g -e s/MONTHVAR/${MONTH}/g -e s/YEARVAR/${YEAR}/g < ${DOC_PREFIX}_front.xml > ${DOC_PREFIX}_front_autogen.xml

${DOC_PREFIX}_rfc_start_autogen.xml: ${DOC_PREFIX}_rfc_start.xml Makefile
	sed -e s/VERSIONVAR/${VERS}/g < ${DOC_PREFIX}_rfc_start.xml > ${DOC_PREFIX}_rfc_start_autogen.xml

autogen/basic_types.xml: ../${DOC_PREFIX}.d/spit_types.sh
	sh ../${DOC_PREFIX}.d/spit_types.sh $@

AUTOGEN =	\
		${DOC_PREFIX}_front_autogen.xml \
		${DOC_PREFIX}_rfc_start_autogen.xml \
		${DOC_PREFIX}_autogen.xml

../${DOC_PREFIX}.d/nfsv41.x: ${DOC_PREFIX}

${DOC_PREFIX}_autogen.xml: Makefile ../${DOC_PREFIX}.d/nfsv41.x
	@rm -f $@.tmp $@
	@cat ../${DOC_PREFIX}.d/nfsv41.x | sed 's?^?   /// ?' | sed 's/</\&lt;/g'| \
	awk ' \
		BEGIN	{ print "<figure>"; print" <artwork>"; } \
			{ print $0 ; } \
		END	{ print " </artwork>"; print"</figure>" ; } ' \
	| expand > $@.tmp
	mv $@.tmp $@

START_PREGEN = ${DOC_PREFIX}_rfc_start.xml
START=	${DOC_PREFIX}_rfc_start_autogen.xml
END=	${DOC_PREFIX}_rfc_end.xml

FRONT_PREGEN = ${DOC_PREFIX}_front.xml

IDXMLSRC_BASE = \
	${DOC_PREFIX}_middle_start.xml \
	${DOC_PREFIX}_middle_body.xml \
	${DOC_PREFIX}_middle_security_considerations.xml \
	${DOC_PREFIX}_middle_iana_considerations.xml \
	${DOC_PREFIX}_middle_end.xml \
	${DOC_PREFIX}_back_front.xml \
	${DOC_PREFIX}_back_references.xml \
	${DOC_PREFIX}_back_back.xml

IDCONTENTS = ${DOC_PREFIX}_front_autogen.xml $(IDXMLSRC_BASE)

IDXMLSRC = ${DOC_PREFIX}_front.xml $(IDXMLSRC_BASE)

${DRAFT_BASE}-$(VERS).xml: Makefile $(START) $(IDCONTENTS) $(END) $(AUTOGEN)
		rm -f $@ $@.tmp
		cp $(START) $@.tmp
		chmod +w $@.tmp
		for i in $(IDCONTENTS) ; do cat $$i >> $@.tmp ; done
		cat $(END) >> $@.tmp
		mv $@.tmp $@

spellcheck: $(IDXMLSRC)
	for f in $(IDXMLSRC); do echo "Spell Check of $$f"; spell +../dictionary.txt $$f; done

AUXFILES = \
	../dictionary.txt \
	../gendraft \
	../Makefile \
	../errortbl \
	../rfcdiff \
	../xml2rfc

DRAFTFILES = \
	${DRAFT_BASE}-$(VERS).txt \
	${DRAFT_BASE}-$(VERS).xml

draft-$(VERS).tar: $(IDCONTENTS) $(START_PREGEN) $(FRONT_PREGEN) $(AUXFILES) $(DRAFTFILES) ${DOC_PREFIX}
	rm -f draft-$(VERS).tar.gz
	tar -cvf draft-$(VERS).tar $(START_PREGEN) $(END) $(FRONT_PREGEN) $(IDCONTENTS) $(AUXFILES) $(DRAFTFILES) ../${DOC_PREFIX}.d; gzip draft-$(VERS).tar

filelist: Makefile
	echo "" > tmp.filelist ; \
	for f in Makefile $(START_PREGEN) $(END) $(FRONT_PREGEN) $(IDCONTENTS) ; do \
	echo ${DOC_PREFIX}-id.d/$$f >> tmp.filelist ; \
	done
