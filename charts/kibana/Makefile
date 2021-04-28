default: test

STACK_VERSION := 7.12.0
RELEASE := kibana
ES_NAMESPACE := observability
ELASTICSEARCH_IMAGE := docker.elastic.co/kibana/kibana:7.12.0
TIMEOUT := 1200s

install:
	helm upgrade --wait --timeout=$(TIMEOUT) --install --values values.yaml $(RELEASE) -n $(ES_NAMESPACE)  ./

test: secrets install goss

purge:
	kubectl delete secrets kibana || true
	kubectl delete secrets -n $(ES_NAMESPACE) kibana || true
	helm del $(RELEASE) -n $(ES_NAMESPACE)

pull-elasticsearch-image:
	docker pull $(ELASTICSEARCH_IMAGE)

secrets:
	encryptionkey=$$(docker run --rm busybox:1.31.1 /bin/sh -c "< /dev/urandom tr -dc _A-Za-z0-9 | head -c50") && \
	kubectl create secret generic kibana --from-literal=encryptionkey=$$encryptionkey

sync-secrets:
	kubectl get secret kibana \
	--namespace=default -o json \
	| jq 'del(.metadata.namespace)' \
	| kubectl create -n $(ES_NAMESPACE) -f -

get-credentials:
	kubectl get secrets elastic-credentials -o json |jq -r '.data.username'|base64 -d; echo
	kubectl get secrets elastic-credentials -o json |jq -r '.data.password'|base64 -d; echo
