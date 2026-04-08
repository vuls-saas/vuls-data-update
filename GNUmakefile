.PHONY: sync-instructions
sync-instructions:
	@mkdir -p .claude/rules
	@for f in .github/instructions/*.instructions.md; do \
		base=$$(basename "$$f" .instructions.md); \
		sed '/^---$$/,/^---$$/d' "$$f" > ".claude/rules/$$base.md"; \
		echo "synced: $$f → .claude/rules/$$base.md"; \
	done
