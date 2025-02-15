WHEELDIR ?= /tmp/wheelhouse
ST2_COMPONENT := $(notdir $(CURDIR))
ST2PKG_RELEASE ?= 1
CHANGELOG_COMMENT ?= "automated build, version: $(ST2PKG_VERSION)"

ifneq (,$(wildcard /etc/debian_version))
	DEBIAN := 1
	#DEB_EPOCH := $(shell echo $(ST2PKG_VERSION) | grep -q dev || echo '1')
	DEB_DISTRO := $(shell lsb_release -cs)
else ifneq (,$(wildcard /etc/rocky-release))
	EL_DISTRO:=rocky
	EL_VERSION := $(shell cat /etc/rocky-release | grep -oP '(?<= )[0-9]+(?=\.)')
	REDHAT := 1
else ifneq (,$(wildcard /etc/centos-release))
	EL_DISTRO:=centos
	EL_VERSION := $(shell cat /etc/centos-release | grep -oP '(?<= )[0-9]+(?=\.)')
	REDHAT := 1
else ifneq (,$(wildcard /etc/redhat-release))
	EL_DISTRO:=redhat
	EL_VERSION := $(shell cat /etc/redhat-release | grep -oP '(?<= )[0-9]+(?=\.)')
	REDHAT := 1
else
	REDHAT := 1
	DEB_DISTRO := unstable
endif

ifeq ($(DEB_DISTRO),bionic)
	PYTHON_BINARY := /usr/bin/python3
	PIP_BINARY := /usr/local/bin/pip3.6
else ifeq ($(DEB_DISTRO),focal)
	PYTHON_BINARY := /usr/bin/python3
	PIP_BINARY := /usr/local/bin/pip3.8
else ifeq ($(EL_VERSION),8)
	PYTHON_BINARY := /usr/bin/python3.8
	PIP_BINARY := /usr/local/bin/pip3.8
	PYTHON_ALT_BINARY := python3.8
else ifeq ($(EL_VERSION),7)
	PYTHON_BINARY := /usr/bin/python3
	PIP_BINARY := /usr/local/bin/pip3
	PYTHON_ALT_BINARY := python3
else
	PYTHON_BINARY := /usr/bin/python3
	PIP_BINARY := pip3
	PYTHON_ALT_BINARY := python3
endif

RUNNERS := $(shell ls ../contrib/runners)

# Moved from top of file to handle when only py2 or py3 available
ST2PKG_VERSION ?= $(shell $(PYTHON_BINARY) -c "from $(ST2_COMPONENT) import __version__; print(__version__),")

# Note: We dynamically obtain the version, this is required because dev
# build versions don't store correct version identifier in __init__.py
# and we need setup.py to normalize it (e.g. 1.4dev -> 1.4.dev0)
ST2PKG_NORMALIZED_VERSION ?= $(shell $(PYTHON_BINARY) setup.py --version || echo "failed_to_retrieve_version")

# Makefile function to retry the failed command 'N' times
# Make sure you use it for nothing but networking stuff (think about race conditons)
# Example: $(call retry,3,some_script.sh)
retry = $(2) $(foreach t,$(shell seq 1 ${1}),|| (echo -e "\033[33m Failed ($$?): '$(2)'\n Retrying $t ... \033[0m"; $(2)))

.PHONY: all install wheelhouse
all: info install

.PHONY: info
info:
	@echo "DEBIAN=$(DEBIAN)"
	@echo "REDHAT=$(REDHAT)"
	@echo "EL_DISTRO=$(EL_DISTRO)"
	@echo "EL_VERSION=$(EL_VERSION)"
	@echo "DEB_DISTRO=$(DEB_DISTRO)"
	@echo "PYTHON_BINARY=$(PYTHON_BINARY)"
	@echo "PIP_BINARY=$(PIP_BINARY)"

install: wheelhouse changelog

post_install:
	# post_install is triggered from debian/rules file.
	# Don't call it from makefile install target!
	install -m640 /dev/null $(DESTDIR)/etc/st2/htpasswd
	install -m644 ../conf/st2.package.conf $(DESTDIR)/etc/st2/st2.conf
	install -m644 ../conf/logrotate.conf $(DESTDIR)/etc/logrotate.d/st2
	install -m644 ../st2actions/conf/logging.conf $(DESTDIR)/etc/st2/logging.actionrunner.conf
	install -m644 ../st2actions/conf/syslog.conf $(DESTDIR)/etc/st2/syslog.actionrunner.conf
	install -m644 ../st2actions/conf/logging.notifier.conf $(DESTDIR)/etc/st2/logging.notifier.conf
	install -m644 ../st2actions/conf/syslog.notifier.conf $(DESTDIR)/etc/st2/syslog.notifier.conf
	install -m644 ../st2actions/conf/logging.workflowengine.conf $(DESTDIR)/etc/st2/logging.workflowengine.conf
	install -m644 ../st2actions/conf/syslog.workflowengine.conf $(DESTDIR)/etc/st2/syslog.workflowengine.conf
	install -m644 ../st2api/conf/logging.conf $(DESTDIR)/etc/st2/logging.api.conf
	install -m644 ../st2api/conf/logging.gunicorn.conf $(DESTDIR)/etc/st2/logging.api.gunicorn.conf
	install -m644 ../st2api/conf/syslog.conf $(DESTDIR)/etc/st2/syslog.api.conf
	install -m644 ../st2stream/conf/logging.conf $(DESTDIR)/etc/st2/logging.stream.conf
	install -m644 ../st2stream/conf/logging.gunicorn.conf $(DESTDIR)/etc/st2/logging.stream.gunicorn.conf
	install -m644 ../st2stream/conf/syslog.conf $(DESTDIR)/etc/st2/syslog.stream.conf
	install -m644 ../st2auth/conf/logging.conf $(DESTDIR)/etc/st2/logging.auth.conf
	install -m644 ../st2auth/conf/logging.gunicorn.conf $(DESTDIR)/etc/st2/logging.auth.gunicorn.conf
	install -m644 ../st2auth/conf/syslog.conf $(DESTDIR)/etc/st2/syslog.auth.conf
	install -m644 ../st2reactor/conf/logging.timersengine.conf $(DESTDIR)/etc/st2/logging.timersengine.conf
	install -m644 ../st2reactor/conf/syslog.timersengine.conf $(DESTDIR)/etc/st2/syslog.timersengine.conf
	install -m644 ../st2reactor/conf/logging.sensorcontainer.conf $(DESTDIR)/etc/st2/logging.sensorcontainer.conf
	install -m644 ../st2reactor/conf/syslog.sensorcontainer.conf $(DESTDIR)/etc/st2/syslog.sensorcontainer.conf
	install -m644 ../st2reactor/conf/logging.rulesengine.conf $(DESTDIR)/etc/st2/logging.rulesengine.conf
	install -m644 ../st2reactor/conf/syslog.rulesengine.conf $(DESTDIR)/etc/st2/syslog.rulesengine.conf
	install -m644 ../st2reactor/conf/logging.garbagecollector.conf $(DESTDIR)/etc/st2/logging.garbagecollector.conf
	install -m644 ../st2reactor/conf/syslog.garbagecollector.conf $(DESTDIR)/etc/st2/syslog.garbagecollector.conf
	install -m644 ../st2actions/conf/logging.scheduler.conf $(DESTDIR)/etc/st2/logging.scheduler.conf
	install -m644 ../st2actions/conf/syslog.scheduler.conf $(DESTDIR)/etc/st2/syslog.scheduler.conf
	install -m755 ../st2client/conf/st2.complete.sh $(DESTDIR)/etc/bash_completion.d/st2
	install -m755 bin/runners.sh $(DESTDIR)/opt/stackstorm/st2/bin/runners.sh
	sed -i -r "/args\s*=\s*/s%logs%/var/log/st2%g" $(DESTDIR)/etc/st2/logging.*conf
	sed -i "/\[logger_root\]/,/\[.*\]\|\s*$$/ {s/level=DEBUG/level=INFO/}" $(DESTDIR)/etc/st2/logging.*conf
	sed -i "/\[logger_root\]/,/\[.*\]\|\s*$$/ {s/level=DEBUG/level=INFO/}" $(DESTDIR)/etc/st2/syslog.*conf
ifeq ($(REDHAT),1)
	if [[ `$(PYTHON_ALT_BINARY) /root/scripts/platform_major_version.py` = 7 ]] || [[ `$(PYTHON_ALT_BINARY) /root/scripts/platform_major_version.py` = 8 ]]; then sed -i -r 's/virtualenv_opts\s*=.*/virtualenv_opts =/' $(DESTDIR)/etc/st2/st2.conf; fi
endif

populate_version: .stamp-populate_version
.stamp-populate_version:
	# populate version should be run before any pip/setup.py works
	sh ../scripts/populate-version.sh
	touch $@

requirements:
	$(PYTHON_BINARY) ../scripts/fixate-requirements.py -s ../st2*/in-requirements.txt ../contrib/runners/*/in-requirements.txt -f ../fixed-requirements.txt
	cat requirements.txt

changelog: populate_version
ifeq ($(DEBIAN),1)
	[ -z "$(DEB_EPOCH)" ] && _epoch="" || _epoch="$(DEB_EPOCH):"; \
		dch -m --force-distribution -v$${_epoch}$(ST2PKG_VERSION)-$(ST2PKG_RELEASE) -D$(DEB_DISTRO) $(CHANGELOG_COMMENT)
endif

wheelhouse: .build-runners .stamp-wheelhouse
.stamp-wheelhouse: | populate_version requirements inject-deps
	cat requirements.txt
	# Try to install wheels 2x in case the first one fails
	$(PIP_BINARY) --use-deprecated=legacy-resolver  wheel --wheel-dir=$(WHEELDIR) --find-links=$(WHEELDIR) -r requirements.txt || \
		$(PIP_BINARY) --use-deprecated=legacy-resolver wheel --wheel-dir=$(WHEELDIR) --find-links=$(WHEELDIR) -r requirements.txt
	touch $@

.build-runners:
	@echo "##########################################"
	@echo "#            Building Runners            #"
	@echo "##########################################"
	@echo $(RUNNERS)
	@for runner in $(RUNNERS); do \
		echo "Installing $$runner"; \
		$(PIP_BINARY) wheel --wheel-dir=$(WHEELDIR) --find-links=$(WHEELDIR) -r ../contrib/runners/$$runner/requirements.txt ../contrib/runners/$$runner; \
	done
	touch $@

# Note: We want to dynamically inject "st2client" dependency. This way we can
# pin it to the version we build so the requirement is satisfied by locally
# built wheel and not version from PyPi
inject-deps: .stamp-inject-deps
.stamp-inject-deps:
	echo "st2client==$(ST2PKG_NORMALIZED_VERSION)" >> requirements.txt
	touch $@
