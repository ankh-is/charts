default: test

STACK_VERSION := 7.12.0
RELEASE := metricbeat
ES_NAMESPACE := observability
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:7.12.0
TIMEOUT := 1200s

install:
	helm upgrade --wait --timeout=$(TIMEOUT) --install --values values.yaml $(RELEASE) -n $(ES_NAMESPACE)  ./

test: secrets install goss

purge:
	kubectl delete secrets -n $(ES_NAMESPACE) || true
	helm del $(RELEASE) -n $(ES_NAMESPACE)

pull-elasticsearch-image:
	docker pull $(ELASTICSEARCH_IMAGE)
