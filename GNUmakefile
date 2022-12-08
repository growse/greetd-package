DEBNAME := greetd
APP_REMOTE := https://git.sr.ht/~kennylevinsen/greetd/archive/
VERSION := 0.8.0
APPDESCRIPTION :=  A login manager daemon
APPURL := https://git.sr.ht/~kennylevinsen/greetd
APP_VENDOR := https://git.sr.ht/~kennylevinsen/greetd
ARCH := arm64 amd64

TARBALL = ./$(VERSION).tar.gz
SRC_REMOTE := $(APP_REMOTE)$(TARBALL)

# Setup
BUILD_NUMBER ?= 0
DEBVERSION := $(VERSION:v%=%)-$(BUILD_NUMBER)
APP_SRCDIR := $(DEBNAME)-$(VERSION)
APP_CONFIGURE_SCRIPT = $(APP_SRCDIR)/Cargo.toml
CROSS_MANIFEST = $(APP_SRCDIR)/Cross.toml
CROSS_BIN :=  cross

# Let's map from rust architectures to deb architectures, because they're not the same!
RUST_arm64_ARCH := aarch64-unknown-linux-gnu
RUST_amd64_ARCH := x86_64-unknown-linux-gnu

.EXPORT_ALL_VARIABLES:

.PHONY: package
package: $(addsuffix .deb, $(addprefix $(DEBNAME)_$(DEBVERSION)_, $(foreach a, $(ARCH), $(a))))

.PHONY: build
build: $(addsuffix /release/greetd, $(addprefix $(APP_SRCDIR)/target/, $(foreach a, $(ARCH), $(a))))

.PHONY: extract
extract: $(APP_CONFIGURE_SCRIPT)

.PHONY: download
download: $(TARBALL)

$(TARBALL):	
	curl --fail -O $(SRC_REMOTE)	

$(APP_CONFIGURE_SCRIPT): $(TARBALL)	
	$(info making $@)
	tar -zmxvf $(TARBALL)

$(CROSS_MANIFEST): $(APP_CONFIGURE_SCRIPT)
	cp Cross.toml $(APP_SRCDIR)/Cross.toml

$(APP_SRCDIR)/target/%/release/greetd: $(APP_CONFIGURE_SCRIPT) $(CROSS_MANIFEST)
	cd $(APP_SRCDIR) && $(CROSS_BIN) build -v --target=$(RUST_$*_ARCH) --release
	cp -R $(APP_SRCDIR)/target/$(RUST_$*_ARCH) $(APP_SRCDIR)/target/$*

$(DEBNAME)_$(DEBVERSION)_%.deb: $(APP_SRCDIR)/target/%/release/greetd
	bundle exec fpm -f \
	-s dir \
	-t deb \
	--license BSD \
	--deb-priority optional \
	--maintainer github@growse.com \
	--vendor $(APP_VENDOR) \
	-n $(DEBNAME) \
	--description "$(APPDESCRIPTION)" \
	--url $(APPURL) \
	--prefix / \
	-a $* \
	-v $(DEBVERSION) \
	--deb-systemd $(APP_SRCDIR)/greetd.service \
	.

.PHONY: clean
clean:
	rm -f *.deb
	rm -rf -- greetd-0.8.0
	rm -f -- *.tar.gz
