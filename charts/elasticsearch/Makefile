default: test

STACK_VERSION := 7.12.0
RELEASE := elasticsearch
ES_NAMESPACE := observability
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:7.12.0
TIMEOUT := 1200s

install:
	helm upgrade --wait --timeout=$(TIMEOUT) --install --values values.yaml $(RELEASE) -n $(ES_NAMESPACE) ./

test: secrets install goss

purge:
	kubectl delete secrets elastic-credentials elastic-certificates elastic-certificate-pem elastic-certificate-crt|| true
	kubectl delete secrets -n $(ES_NAMESPACE) elastic-credentials elastic-certificates elastic-certificate-pem elastic-certificate-crt|| true
	helm del $(RELEASE)

pull-elasticsearch-image:
	docker pull $(ELASTICSEARCH_IMAGE)

secrets:
	docker rm -f elastic-helm-charts-certs || true
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12 || true
	password=$$([ ! -z "$$ELASTIC_PASSWORD" ] && echo $$ELASTIC_PASSWORD || echo $$(docker run --rm busybox:1.31.1 /bin/sh -c "< /dev/urandom tr -cd '[:alnum:]' | head -c20")) && \
	docker run --name elastic-helm-charts-certs -i -w /tmp \
		$(ELASTICSEARCH_IMAGE) \
		/bin/sh -c " \
			elasticsearch-certutil ca --out /tmp/elastic-stack-ca.p12 --pass '' && \
			elasticsearch-certutil cert --name elasticsearch-master --dns elasticsearch-master,kibana-kibana,logstash-logstash --ca /tmp/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /tmp/elastic-certificates.p12" && \
	docker cp elastic-helm-charts-certs:/tmp/elastic-certificates.p12 ./ && \
	docker rm -f elastic-helm-charts-certs && \
	openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem && \
	openssl x509 -outform der -in elastic-certificate.pem -out elastic-certificate.crt && \
	kubectl create secret generic elastic-certificates --from-file=elastic-certificates.p12 && \
	kubectl create secret generic elastic-certificate-pem --from-file=elastic-certificate.pem && \
	kubectl create secret generic elastic-certificate-crt --from-file=elastic-certificate.crt && \
	kubectl create secret generic elastic-credentials --from-literal=password=$$password --from-literal=username=elastic && \
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12

sync-secrets:
	kubectl create namespace ${ES_NAMESPACE} || true
	kubectl get secret elastic-certificates \
	--namespace=default -o json \
	| jq 'del(.metadata.namespace)' \
	| kubectl create -n $(ES_NAMESPACE) -f -

	kubectl get secret elastic-certificate-pem \
	--namespace=default -o json \
	| jq 'del(.metadata.namespace)' \
	| kubectl create -n $(ES_NAMESPACE) -f -

	kubectl get secret elastic-certificate-crt \
	--namespace=default -o json \
	| jq 'del(.metadata.namespace)' \
	| kubectl create -n $(ES_NAMESPACE) -f -

	kubectl get secret elastic-credentials \
	--namespace=default -o json \
	| jq 'del(.metadata.namespace)' \
	| kubectl create -n $(ES_NAMESPACE) -f -
