SHELL := /bin/bash
mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
APP := $(notdir $(patsubst %/,%,$(dir $(mkfile_path))))
CTX := $(shell kubectl config current-context)

.PHONY: check-helm
check-helm:
	@if ! which helm > /dev/null; then \
  	  echo "helm not installed; consider installing it from https://helm.sh/"; \
	fi

.PHONY: check-helm-secrets
check-helm-secrets:
	@if ! helm plugin list | grep secrets > /dev/null; then \
  	  echo "helm secrets not installed; consider installing it from https://github.com/jkroepke/helm-secrets"; \
	fi

.PHONY: check-helm-diff
check-helm-diff:
	@if ! helm plugin list | grep diff > /dev/null; then \
	  echo "helm secrets not installed; consider installing it from https://github.com/databus23/helm-diff"; \
	fi

.PHONY: check-kubectx
check-kubectx:
	@if ! which kubectx > /dev/null; then \
	  echo "kubectx not installed; consider installing it from https://github.com/ahmetb/kubectx"; \
	fi

.PHONY: check-env
check-env:
ifndef SOPS_AGE_KEY_FILE
	$(error SOPS_AGE_KEY_FILE is undefined. Please source profile.dev or profile.prod)
endif

.PHONY: check-prompt
check-prompt:
	@echo -n "Deploying from \"${CTX}\" context. Are you sure? [y/N] " && read ans && [ $${ans:-N} = y ]

.PHONY: switch-ctx
switch-ctx:
	@kubectx rpi

.PHONY: checks
checks: check-helm check-kubectx switch-ctx check-helm-secrets check-helm-diff check-env

.PHONY: template
template: checks
template:
	@source .metadata && \
    	if [[ -s "secrets.yaml" ]]; then \
    		helm secrets template ${APP} . -f values.yaml -f secrets.yaml; \
    	else \
    		helm template ${APP} . -f values.yaml; \
    	fi

.PHONY: install
install: checks check-prompt
install:
	@source .metadata && \
	if [[ -s "secrets.yaml" ]]; then \
		helm secrets install --create-namespace ${APP} . -f values.yaml -f secrets.yaml; \
	else \
		helm install --create-namespace ${APP} . -f values.yaml; \
	fi

.PHONY: upgrade-nodiff
upgrade-nodiff: checks
upgrade-nodiff:
	@source .metadata && \
	if [[ -s "secrets.yaml" ]]; then \
		helm secrets upgrade ${APP} . -f values.yaml -f secrets.yaml; \
	else \
		helm upgrade ${APP} . -f values.yaml; \
	fi

.PHONY: upgrade
upgrade: checks diff check-prompt upgrade-nodiff

.PHONY: diff
diff: checks check-helm-diff
diff:
	@echo "calculating diff.."
	@source .metadata && \
	if [[ -s "secrets.yaml" ]]; then \
		helm secrets diff upgrade ${APP} . -f values.yaml -f secrets.yaml; \
	else \
		helm diff upgrade ${APP} . -f values.yaml; \
	fi

.PHONY: uninstall
uninstall: checks check-prompt
uninstall:
	@source .metadata && helm uninstall ${APP}


.PHONY: decrypt-secrets
decrypt-secrets: checks
decrypt-secrets:
	@helm secrets dec secrets.yaml

.PHONY: encrypt-secrets
encrypt-secrets: checks
encrypt-secrets:
	@helm secrets enc secrets.yaml

.PHONY: edit-secrets
edit-secrets: checks
edit-secrets:
	@helm secrets edit secrets.yaml

.PHONY: view-secrets
view-secrets: checks
view-secrets:
	@helm secrets view secrets.yaml


# For extending makefile in each
# TODO maybe this can be generic as well
-include build.makefile
