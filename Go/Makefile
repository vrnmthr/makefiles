# Go settings
GO := go
DEP := dep
GOARCH := $(shell go env GOARCH)
GOOS := $(shell go env GOOS)

# Project settings
BUILD_DIR := build
BIN_DIR := bin
GO_PACKAGES := $(shell go list ./... | grep -v /vendor/)
BINARIES := $(shell ls cmd)

######################################################
#
# Development targets
#
######################################################

.PHONY: all
all: ensure build test bench

.PHONY: ensure
ensure:
	@$(DEP) ensure
	@$(DEP) status

.PHONY: vendor
vendor:
	@$(DEP) status
	@$(DEP) ensure -v -vendor-only

# Builds all binaries in cmd/ into bin/
.PHONY: build
build:
	@rm -rf $(BIN_DIR)
	@for pkg in $(GO_PACKAGES); do \
		go install $$pkg || exit 1; \
	done; \
	mkdir -p $(BIN_DIR)
	@for cmd in $(BINARIES); do \
		mv $(GOPATH)/bin/$$cmd $(BIN_DIR) || exit 1; \
	done; \

# Tests all packages with race and coverage. Places output in build/
.PHONY: test
test: clean
	@for pkg in $(GO_PACKAGES); do \
		go test -v -race -coverprofile=$(BUILD_DIR)/coverage_tmp.out $$pkg 2>&1 | tee -a $(BUILD_DIR)/test.out || exit 1 ; \
		tail -n +2 $(BUILD_DIR)/coverage_tmp.out >> $(BUILD_DIR)/coverage.out; \
	done;
	@go tool cover -html=$(BUILD_DIR)/coverage.out -o $(BUILD_DIR)/coverage.html
	@go tool cover -func=$(BUILD_DIR)/coverage.out -o $(BUILD_DIR)/coverage.func
	@echo '==================================='
	@echo '========== TESTS SUMMARY =========='
	@echo '==================================='
	@grep -E '^---|^===' $(BUILD_DIR)/test.out
	@echo '==========================================='
	@echo '========== PACKAGE TEST COVERAGE =========='
	@echo '==========================================='
	@grep -vE '^20' $(BUILD_DIR)/test.out | grep 'coverage:' -A 1

.PHONY: bench
bench:
	$(GO) test -run=- -bench=. -benchmem ./...

.PHONY: fmt
fmt:
	$(GO) fmt ./...

.PHONY: clean
clean:
	@rm -rf $(BUILD_DIR)
	@mkdir $(BUILD_DIR)
	@echo "mode: count" > $(BUILD_DIR)/coverage.out
	@echo "" > $(BUILD_DIR)/coverage_tmp.out

# TODO: add Docker/Kubernetes targets