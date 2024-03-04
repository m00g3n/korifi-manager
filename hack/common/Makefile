PROJECT_ROOT       ?= ../..
CLUSTER_NAME       ?= kyma
REGISTRY_PORT      ?= 5001
REGISTRY_NAME      ?= ${CLUSTER_NAME}-registry
MANAGER_IMAGE_NAME ?= appcon-manager-dev-local
MANAGER_IMAGE_TAG  ?= 0.0.1

.PHONY: run
run: ## Create k3d cluster and deploy module with the lifecycle-manager.
run: \
        kyma \
	create-k3d \
	create-kyma-system-ns \
	module-image \
	deploy \
	apply-korifi-cr \
	verify-korifi	

.PHONY: kyma
kyma:
	@make -C ${PROJECT_ROOT} kyma

.PHONY: create-k3d
create-k3d: ## Create k3d with kyma CRDs.
	${KYMA} provision k3d \
		--registry-port ${REGISTRY_PORT} \
		--name ${CLUSTER_NAME} \
		-p 8080:80@loadbalancer \
		-p 8443:443@loadbalancer \
		--ci

.PHONY: create-kyma-system-ns
create-kyma-system-ns: ## Create kyma-system namespace.
	kubectl create ns kyma-system

.PHONY: module-image
module-image:
	@make -C ${PROJECT_ROOT} module-image \
		IMG=localhost:${REGISTRY_PORT}/${MANAGER_IMAGE_NAME}:${MANAGER_IMAGE_TAG}

.PHONY: deploy
deploy:
	@make -C ${PROJECT_ROOT} deploy \
		IMG=k3d-${REGISTRY_NAME}:${REGISTRY_PORT}/${MANAGER_IMAGE_NAME}:${MANAGER_IMAGE_TAG}

.PHONY: apply-korifi-cr
apply-keda: ## Apply the Korifi CR.
	kubectl apply -n kyma-system \
		-f ${PROJECT_ROOT}/config/samples/operator_v1_korifi.yaml

##@ Module management

.PHONY: verify-korifi
verify-keda: ## incomplete
	kubectl wait \
		--for condition=Available \
		deployment korify-controller-manager \
		-n kyma-system \
		--timeout=60s || \
		(make print-keda-details && false)

.PHONY: print-korifi-details
print-keda-details: ## Print all pods, deploys and korifi CRs in the kyma-system namespace.
	kubectl get korifi -n kyma-system -oyaml
	kubectl get deploy -n kyma-system -oyaml
	kubectl get pods -n kyma-system -oyaml
