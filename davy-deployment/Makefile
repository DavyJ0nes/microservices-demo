#### VARIABLES ####

NAMESPACE = sock-shop
DEPLOYMENT_GCP_PROJECT = interview-davy-socks
DNS_GCP_PROJECT = davyj0nes-demo
USER_GCP_EMAIL = davyrogerjones@gmail.com

#### COMMANDS ####

.PHONY: create-cluster-admin
create-cluster-admin:
	kubectl create clusterrolebinding user-cluster-admin-binding --clusterrole=cluster-admin --user=${USER_GCP_EMAIL}

.PHONY: create-ns
create-ns:
	kubectl create namespace ${NAMESPACE}

.PHONY: create-gcp-service-account
create-gcp-service-account:
	# gcloud iam service-accounts create clouddns --project=${DNS_GCP_PROJECT}
	# gcloud iam service-accounts add-iam-policy-binding \
		--role='roles/dns.admin' --project=${DNS_GCP_PROJECT}
	gcloud iam service-accounts keys create service-account.json\
		--iam-account clouddns@${DNS_GCP_PROJECT}.iam.gserviceaccount.com  --project=${DNS_GCP_PROJECT}
	kubectl create secret generic clouddns-service-account \
		--from-file service-account.json -n ${NAMESPACE}

.PHONY: deploy-nginx
deploy-nginx:
	helm install stable/nginx-ingress --namespace kube-system

.PHONY: deploy-cert-manager
deploy-cert-manager:
	helm install --name cert-manager --namespace kube-system stable/cert-manager

.PHONY: configure-cert-manager
configure-cert-manager:
	# Ensure you've run `make gcp-service-account` before doing this as it will fail
	kubectl -n ${NAMESPACE} create -f front-end/acme-issuer.yml
	kubectl -n ${NAMESPACE} create -f front-end/certificate.yml

.PHONY: deploy-backend
deploy-backend:
	kubectl create -f back-end/backend-services.yml

.PHONY: deploy-frontend
deploy-frontend:
	kubectl create -f front-end/front-end.yml

.PHONY: destroy-backend
destroy-backend:
	kubectl delete -f back-end/backend-services.yml

.PHONY: destroy-frontend
destroy-frontend:
	kubectl delete -f front-end/front-end.yml

.PHONY: get-stuff
get-stuff:
	@echo "PODS" && echo "----"
	@kubectl get pods -n ${NAMESPACE}
	@echo "\nSERVICES" && echo "--------"
	@kubectl get svc -n ${NAMESPACE}
	@echo "\nINGRESS" && echo "-------"
	@kubectl get ingress -n ${NAMESPACE}

.PHONY: get-all
get-all:
	@echo "PODS" && echo "----"
	@kubectl get pods --all-namespaces
	@echo "\nSERVICES" && echo "--------"
	@kubectl get svc --all-namespaces
	@echo "\nINGRESS" && echo "-------"
	@kubectl get ingress --all-namespaces

#### FUNCTIONS ####

define blue
	@tput setaf 4
	@echo $1
	@tput sgr0
endef
