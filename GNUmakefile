TEST?=$(shell go list ./... |grep -v 'vendor' |grep -v 'utils')
GOFMT_FILES?=$(shell find . -name '*.go' |grep -v vendor)
PKG_NAME=nutanix
WEBSITE_REPO=github.com/hashicorp/terraform-website

_ARCH := $(shell uname -m)
_OS   := $(shell uname | tr '[:upper:]' '[:lower:]')

ifndef ARCH
	ARCH := $(shell uname -m)
	export ARCH
endif
	
ifeq ($(ARCH),x86_64)
	ARCH := amd64
endif

ifeq ($(_ARCH),x86_64)
	_ARCH := amd64
endif

ifndef OS
	OS := $(shell uname | tr '[:upper:]' '[:lower:]')
	export OS
endif

ifdef RELEASE
ifndef FLAGS
	FLAGS := -ldflags "-s -w"
endif
endif

default: dist/terraform-provider-nutanix_$(OS)_$(ARCH)

build: dist/terraform-provider-nutanix_$(OS)_$(ARCH)

dist/terraform-provider-nutanix_$(OS)_$(ARCH): ${GOFMT_FILES}
ifeq ($(ARCH)$(OS),$(_ARCH)$(_OS))
	go build -o dist/terraform-provider-nutanix_$(OS)_$(ARCH) $(FLAGS)
else
	CGO_ENABLED=0 GOOS=$(OS) GOARCH=$(ARCH) CC=$(CC) go build -o dist/terraform-provider-nutanix_$(OS)_$(ARCH) $(FLAGS)
endif

clean:
	rm -rf dist/*

dist:
	mkdir -p dist/

install: build
	mkdir -p $(HOME)/.terraform.d/plugins
	install dist/terraform-provider-nutanix_$(OS)_$(ARCH) $(HOME)/.terraform.d/plugins/terraform-provider-nutanix

test: fmtcheck
	go test $(TEST) -timeout=30s -parallel=4
	
testacc: fmtcheck
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m -coverprofile c.out
	go tool cover -html=c.out

fmt:
	@echo "==> Fixing source code with gofmt..."
	goimports -w ./$(PKG_NAME)
	goimports -w ./client
	goimports -w ./utils

fmtcheck:
	@sh -c "'$(CURDIR)/scripts/gofmtcheck.sh'"

errcheck:
	@sh -c "'$(CURDIR)/scripts/errcheck.sh'"

lint: fmtcheck
	@echo "==> Checking source code against linters..."
	@GOGC=30 golangci-lint run

tools:
	GO111MODULE=off go get -u github.com/client9/misspell/cmd/misspell
	GO111MODULE=off go get -u github.com/golangci/golangci-lint/cmd/golangci-lint
	GO111MODULE=off go get github.com/mitchellh/gox	

vet:
	@echo "go vet ."
	@go vet $$(go list ./... | grep -v vendor/) ; if [ $$? -eq 1 ]; then \
		echo ""; \
		echo "Vet found suspicious constructs. Please check the reported constructs"; \
		echo "and fix them if necessary before submitting the code for review."; \
		exit 1; \
	fi

test-compile:
	@if [ "$(TEST)" = "./..." ]; then \
		echo "ERROR: Set TEST to a specific package. For example,"; \
		echo "  make test-compile TEST=./$(PKG_NAME)"; \
		exit 1; \
	fi
	go test -c $(TEST) $(TESTARGS)

# test:
# 	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m -coverprofile c.out
# 	go tool cover -html=c.out

cibuild: tools
	rm -rf pkg/
	gox -output "pkg/{{.OS}}_{{.Arch}}/terraform-provider-nutanix"


citest:
	TF_ACC=1 go test $(TEST) -v $(TESTARGS) -timeout 120m -coverprofile c.out


website:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

website-lint:
	@echo "==> Checking website against linters..."
	@misspell -error -source=text website/

website-test:
ifeq (,$(wildcard $(GOPATH)/src/$(WEBSITE_REPO)))
	echo "$(WEBSITE_REPO) not found in your GOPATH (necessary for layouts and assets), get-ting..."
	git clone https://$(WEBSITE_REPO) $(GOPATH)/src/$(WEBSITE_REPO)
endif
	@$(MAKE) -C $(GOPATH)/src/$(WEBSITE_REPO) website-provider-test PROVIDER_PATH=$(shell pwd) PROVIDER_NAME=$(PKG_NAME)

.NOTPARALLEL:

.PHONY: default test testacc fmt fmtcheck errcheck lint tools vet test-compile cibuild citest website website-lint website-test archdir
