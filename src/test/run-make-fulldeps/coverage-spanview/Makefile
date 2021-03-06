# needs-profiler-support
# min-llvm-version: 11.0

-include ../coverage/coverage_tools.mk

BASEDIR=../coverage-spanview
SOURCEDIR=../coverage

define SPANVIEW_HEADER
<!DOCTYPE html>
<!--

Preview this file as rendered HTML from the github source at:
https://htmlpreview.github.io/?https://github.com/rust-lang/rust/blob/master/src/test/run-make-fulldeps/coverage-spanview/expected_mir_dump.%s/%s

For revisions in Pull Requests (PR):
  * Replace "rust-lang" with the github PR author
  * Replace "master" with the PR branch name

-->
endef
export SPANVIEW_HEADER

all: $(patsubst $(SOURCEDIR)/lib/%.rs,%,$(wildcard $(SOURCEDIR)/lib/*.rs)) $(patsubst $(SOURCEDIR)/%.rs,%,$(wildcard $(SOURCEDIR)/*.rs))

# Ensure there are no `expected` results for tests that may have been removed or renamed
.PHONY: clear_expected_if_blessed
clear_expected_if_blessed:
ifdef RUSTC_BLESS_TEST
	rm -rf expected_mir_dump.*/
endif

-include clear_expected_if_blessed

# FIXME(richkadel): The actions for these two types of targets (libraries and binaries) should be
# combined.

%: $(SOURCEDIR)/lib/%.rs
	# Compile the test library with coverage instrumentation
	$(RUSTC) $(SOURCEDIR)/lib/$@.rs \
			$$( grep -q '^\/\/ require-rust-edition-2018' $(SOURCEDIR)/lib/$@.rs && \
				echo "--edition=2018" \
			) \
			--crate-type rlib \
			-Zinstrument-coverage \
			-Zdump-mir=InstrumentCoverage \
			-Zdump-mir-spanview \
			-Zdump-mir-dir="$(TMPDIR)"/mir_dump.$@

	for path in "$(TMPDIR)"/mir_dump.$@/*; do \
		file="$$(basename "$$path")"; \
		urlescaped="$$("$(PYTHON)" $(BASEDIR)/escape_url.py $$file)" || exit $$?; \
		printf "$$SPANVIEW_HEADER\n" "$@" "$$urlescaped" > "$$path".modified; \
		tail -n +2 "$$path" >> "$$path".modified; \
		mv "$$path".modified "$$path"; \
	done && true # for/done ends in non-zero status

ifdef RUSTC_BLESS_TEST
	mkdir -p expected_mir_dump.$@
	cp "$(TMPDIR)"/mir_dump.$@/*InstrumentCoverage.0.html expected_mir_dump.$@/
else
	# Check that the selected `mir_dump` files match what we expect (`--bless` refreshes `expected`)
	mkdir -p "$(TMPDIR)"/actual_mir_dump.$@
	rm -f "$(TMPDIR)"/actual_mir_dump.$@/*
	cp "$(TMPDIR)"/mir_dump.$@/*InstrumentCoverage.0.html "$(TMPDIR)"/actual_mir_dump.$@/
	$(DIFF) -r expected_mir_dump.$@/ "$(TMPDIR)"/actual_mir_dump.$@/
endif

%: $(SOURCEDIR)/%.rs
	# Compile the test program with coverage instrumentation
	$(RUSTC) $(SOURCEDIR)/$@.rs \
			$$( grep -q '^\/\/ require-rust-edition-2018' $(SOURCEDIR)/$@.rs && \
				echo "--edition=2018" \
			) \
			-L "$(TMPDIR)" \
			-Zinstrument-coverage \
			-Zdump-mir=InstrumentCoverage \
			-Zdump-mir-spanview \
			-Zdump-mir-dir="$(TMPDIR)"/mir_dump.$@

	for path in "$(TMPDIR)"/mir_dump.$@/*; do \
		file="$$(basename "$$path")"; \
		urlescaped="$$("$(PYTHON)" $(BASEDIR)/escape_url.py $$file)" || exit $$?; \
		printf "$$SPANVIEW_HEADER\n" "$@" "$$urlescaped" > "$$path".modified; \
		tail -n +2 "$$path" >> "$$path".modified; \
		mv "$$path".modified "$$path"; \
	done && true # for/done ends in non-zero status

ifdef RUSTC_BLESS_TEST
	mkdir -p expected_mir_dump.$@
	cp "$(TMPDIR)"/mir_dump.$@/*InstrumentCoverage.0.html expected_mir_dump.$@/
else
	# Check that the selected `mir_dump` files match what we expect (`--bless` refreshes `expected`)
	mkdir -p "$(TMPDIR)"/actual_mir_dump.$@
	rm -f "$(TMPDIR)"/actual_mir_dump.$@/*
	cp "$(TMPDIR)"/mir_dump.$@/*InstrumentCoverage.0.html "$(TMPDIR)"/actual_mir_dump.$@/
	$(DIFF) -r expected_mir_dump.$@/ "$(TMPDIR)"/actual_mir_dump.$@/
endif
