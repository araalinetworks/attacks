ifeq (, $(shell which helm))
$(error "Helm not installed, to install brew install helm")
endif

ifdef DRYRUN
    DRYRUN=--dry-run --debug
endif

ifndef NAMESPACE
$(error NAMESPACE is undefined)
endif

VALUESYAML = ./values.yaml

deploy:
	helm install yelb-$(NAMESPACE) $(DRYRUN) -f $(VALUESYAML) -n $(NAMESPACE) .

clean:
	helm delete yelb-$(NAMESPACE) -n $(NAMESPACE)
