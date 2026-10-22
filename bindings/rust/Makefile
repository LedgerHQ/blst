# Publishing to the Ledger internal cargo registry, as `ledger-blst`.
#
# The `package` and `publish` targets are the contract that
# LedgerHQ/enclave-github-workflows/.github/workflows/job-rust-publish.yml
# expects from a repository it publishes.
#
# The crate's `include` list names /blst/src/*.c, /blst/build/** and
# /blst/bindings/blst.h, which live at the repository root, two levels above
# this crate. Cargo only packages files under the crate, so both targets link
# the root in as `blst` first, exactly as upstream's publish.sh does.
#
# publish.sh is not used, because it calls `cargo +stable publish` and the
# team toolchain image installs nightly only.

CARGO          ?= cargo
CARGO_REGISTRY ?= enclave-cargo-prod-green

# The link is not committed, so cargo sees a dirty tree and needs --allow-dirty.
CARGO_FLAGS = --allow-dirty

.PHONY: all build test package publish link unlink

all: build

build: link
	$(CARGO) build

test: link
	$(CARGO) test

link:
	@test -e blst || ln -s ../.. blst

unlink:
	@test -h blst && rm -f blst || true

# Packages the crate without uploading anything, and fails if the C sources the
# build needs did not make it in. Without them the published crate cannot build.
package: link
	@list="$$($(CARGO) package --list $(CARGO_FLAGS))"; \
	for f in blst/src/server.c blst/bindings/blst.h blst/build/assembly.S; do \
		printf '%s\n' "$$list" | grep -qx "$$f" \
			|| { echo "missing from package: $$f" >&2; exit 1; }; \
	done; \
	echo "=== $$(printf '%s\n' "$$list" | wc -l) files packaged, C sources present ==="

publish: link
	$(CARGO) publish --registry $(CARGO_REGISTRY) $(CARGO_FLAGS)
