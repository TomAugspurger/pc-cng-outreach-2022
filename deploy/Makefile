.PHONY: group cluster hub clean storage acr image

LOCATION?=westeurope
GROUP?=rg-pc-cng
CLUSTER?=cngcluster
ACR_NAME?=cngacr
STORAGE_ACCOUNT?=$(GROUP)
STORAGE_CONTAINER?=$(GROUP)
MAX_USER_NODE_COUNT?=500
# 00
NODE_COUNT?=0
SUBSCRIPTION="Planetary Computer"

group:
	az group create --name $(GROUP) --location $(LOCATION)

cluster:
	az aks create --resource-group $(GROUP) --name $(CLUSTER) \
		--generate-ssh-keys \
		--node-count=1 \
		--nodepool-name core \
		--nodepool-labels hub.jupyter.org/node-purpose=core
	az aks get-credentials --name $(CLUSTER) --resource-group $(GROUP)
	az aks nodepool add \
	   --name users \
	   --cluster-name $(CLUSTER) \
	   --resource-group $(GROUP) \
	   --enable-cluster-autoscaler \
	   --node-count 1 \
	   --min-count 0 --max-count $(MAX_USER_NODE_COUNT) \
	   --node-vm-size Standard_D8s_v3 \
	   --labels hub.jupyter.org/node-purpose=user
	az aks nodepool add \
	   --name manualusers \
	   --cluster-name $(CLUSTER) \
	   --resource-group $(GROUP) \
	   --node-count 0 \
	   --node-vm-size Standard_D8s_v3 \
	   --labels hub.jupyter.org/node-purpose=user
	az aks nodepool add \
	   --name workers \
	   --cluster-name $(CLUSTER) \
	   --resource-group $(GROUP) \
	   --enable-cluster-autoscaler \
	   --node-count 1 \
	   --min-count 0 --max-count 200 \
	   --node-vm-size Standard_E16_v3 \
	   --priority Spot \
	   --eviction-policy Delete \
	   --spot-max-price -1 \
	   --labels="k8s.dask.org/dedicated=worker"

hub:
	helm upgrade --wait --install --create-namespace \
		dask dask/daskhub \
		--version=2021.7.0 \
		--namespace=dhub \
		--values=config.yaml \
		--values=secrets.yaml \
		--kube-context=$(CLUSTER)

scale:
	az aks nodepool scale \
		--name=manualusers \
		--cluster-name=$(CLUSTER) \
		--resource-group=$(GROUP) \
		--node-count=$(NODE_COUNT)

clean:
	az group delete -n $(GROUP)

acr:
	# az acr create --resource-group=$(GROUP) --name=$(ACR_NAME) --sku=Standard --location=$(LOCATION)
	az aks update -g $(GROUP) -n $(CLUSTER) --attach-acr $(ACR_NAME)

image:
	az acr build -g $(GROUP) -r $(ACR_NAME) -t pccng:latest .
