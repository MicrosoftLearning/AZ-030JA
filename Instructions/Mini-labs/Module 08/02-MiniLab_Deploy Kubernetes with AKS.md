﻿# ミニラボ: AKS で Kubernetes をデプロイする

このミニラボでは、次の内容を学習します。

* 新しいリソース グループを作成します。

* クラスター ネットワークを構成します。

* Azure Kubernetes Service クラスターを作成します。

* kubectl を使用して、Kubernetes クラスターに接続します。

* Kubernetes 名前空間を作成します。

## 新しいリソース グループの作成

まず、リソースをデプロイするリソース グループを作成する必要があります。

1. Azure アカウントで Azure Cloud Shell にサインインします。Cloud Shell の Bash バージョンを選択します。

[Azure Cloud Shell](https://shell.azure.com/)

2. リソース グループを作成するリージョンを選択する必要があります (**米国東部**など)。別の値を選択した場合は、このモジュールの残りの演習で使用するので、値を覚えておいてください。Cloud Shell セッション間で値を再定義する必要がある場合があります。これらの値を Bash 変数に記録するには、次のコマンドを実行します。

```Azure CLI
REGION_NAME=eastus
RESOURCE_GROUP=aksworkshop
SUBNET_NAME=aks-subnet
VNET_NAME=aks-vnet
 ```

例えば、echo ```$REGION_NAME``` のように echo コマンドを使用して、各値を確認できます。

3. **aksworkshop という名前で新しいリソース グループを作成します。この演習で作成したすべてのリソースをこのリソース グループにデプロイします。単一のリソース グループを使用すると、モジュールの終了後にリソースを簡単にクリーンアップできます。

```Azure CLI
az group create \
    --name $RESOURCE_GROUP \
    --location $REGION_NAME
```

## ネットワークの構成

AKS クラスターを展開する場合は、2 つのネットワーク モデルから選択できます。最初のモデルは *Kubenet ネットワーク* で、2 つ目は *Azure Container Networking Interface (CNI) ネットワーク* です。

## Kubenet ネットワーク

Kubenet ネットワークは、Kubernetes の既定のネットワーク モデルです。Kubenet ネットワークでは、ノードに Azure 仮想ネットワーク サブネットから IP アドレスが割り当てられます。ポッドは、論理的に異なるアドレス空間からノードの Azure 仮想ネットワーク サブネットに IP アドレスを受け取ります。

次に、ポッドが Azure 仮想ネットワーク上のリソースに到達できるように、ネットワーク アドレス変換 (NAT) が構成されます。トラフィックの送信元 IP アドレスは、ノードのプライマリ IP アドレスに変換され、ノード上で設定されます。ポッドはノード IP の背後に "隠された" IP アドレスを受け取ります。

## Azure Container Networking Interface (CNI) のネットワーク

Azure Container Networking Interface (CNI) を使用すると、AKS クラスターは既存の仮想ネットワークリソースと構成に接続されます。このネットワーク モデルでは、各ポッドはサブネットから IP アドレスを取得し、直接アクセスできます。これらの IP アドレスは、ネットワーク空間全体で一意で、事前に計算する必要があります。

使用する機能の一部では、*Azure Container Networking Interface ネットワーク*構成を使用して AKS クラスターをデプロイする必要があります。

以下で、AKS クラスターの仮想ネットワークを作成します。この仮想ネットワークを使用し、クラスターをデプロイするときにネットワーク モデルを指定します。

1. 最初に、仮想ネットワークとサブネットを作成します。クラスターにデプロイされたポッドには、このサブネットから IP が割り当てられます。次のコマンドを実行して、仮想ネットワークを作成します。

```Azure CLI
az network vnet create \
    --resource-group $RESOURCE_GROUP \
    --location $REGION_NAME \
    --name $VNET_NAME \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name $SUBNET_NAME \
    --subnet-prefix 10.240.0.0/16
```

2. 次に、以下のコマンドを実行して、サブネット ID を取得して Bash 変数に保存します。

```Azure CLI
SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
```

## AKS クラスターの作成

新しい仮想ネットワークを配置したら、新しいクラスターを作成できます。 ```az aks create``` コマンドを実行する前に知っておくべきことが 2 つあります。1 つ目は、選択したリージョンで利用可能な最新の非プレビュー Kubernetes バージョンのバージョン、2 つ目はクラスターの一意の名前です。

1. 最新の非プレビュー Kubernetes バージョンを取得するには、 ```az aks get-versions``` コマンドを使用します。コマンドから返される値を ```VERSION``` という名前の Bash 変数に格納します。バージョン番号を取得して保存する以下のコマンドを実行します。

```Azure CLI
SUBNET_ID=$(az network vnet subnet show \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $VNET_NAME \
    --name $SUBNET_NAME \
    --query id -o tsv)
```

2. AKS クラスター名は一意である必要があります。次のコマンドを実行して、一意の名前を保持する Bash 変数を作成します。

```Bash
AKS_CLUSTER_NAME=aksworkshop-$RANDOM
```

3. 次のコマンドを実行して、 ```$ AKS_CLUSTER_NAME``` に保存されている値を出力します。後で使用するために、これをメモしてください。必要に応じて今後、変数を再構成する必要があります。

```Bash
echo $AKS_CLUSTER_NAME
```

4. 次の ```az aks create``` コマンドを実行して、最新の Kubernetes バージョンを実行する AKS クラスターを作成します。このコマンドの完了には数分かかります。

```Azure CLI
az aks create \
--resource-group $RESOURCE_GROUP \
--name $AKS_CLUSTER_NAME \
--vm-set-type VirtualMachineScaleSets \
--load-balancer-sku standard \
--location $REGION_NAME \
--kubernetes-version $VERSION \
--network-plugin azure \
--vnet-subnet-id $SUBNET_ID \
--service-cidr 10.2.0.0/24 \
--dns-service-ip 10.2.0.10 \
--docker-bridge-address 172.17.0.1/16 \
--generate-ssh-keys
```

## kubectl を使用してクラスター接続をテストする

```kubectl``` は、クラスターとの対話に使用する主要な Kubernetes コマンドライン クライアントであり、Cloud Shell で利用可能です。 ```kubectl``` がクラスターに接続できるようにするには、クラスター コンテキストが必要です。コンテキストには、クラスターのアドレス、ユーザー、および名前空間が含まれます。 ```kubectl``` のインスタンスを設定するには、az aks get-credentials コマンドを使用します。

1. 次のコマンドを実行して、クラスターの資格情報を取得します。

```Azure CLI
az aks get-credentials \
    --resource-group $RESOURCE_GROUP \
    --name $AKS_CLUSTER_NAME
```

2. クラスター内のすべてのノードを一覧表示することで、デプロイされた内容を確認できます。すべてのノードを一覧表示するには、 ```kubectl``` get nodes コマンドを使用します。

```Bash
kubectl get nodes
```

クラスターのノード一覧が表示されます。次に例を示します。

```Ouput
NAME                                STATUS   ROLES   AGE  VERSION
aks-nodepool1-24503160-vmss000000   Ready    agent   1m   v1.15.7
aks-nodepool1-24503160-vmss000001   Ready    agent   1m   v1.15.7
aks-nodepool1-24503160-vmss000002   Ready    agent   1m   v1.15.7
```

## アプリケーションの Kubernetes 名前空間を作成する

Kubernetes の名前空間は、論理分離境界を作成します。リソースの名前は名前空間内で一意である必要がありますが、複数の名前空間全体で一意であってはなりません。Kubernetes リソースを操作するときに名前空間を指定しない場合、既定の名前空間が暗黙的に指定されます。

評価アプリケーションの名前空間を作成します。

1. クラスター内の現在の名前空間を一覧表示します。

```Bash
kubectl get namespace
```

この出力に似た名前空間の一覧が表示されます。

```Ouput
NAME              STATUS   AGE
default           Active   1h
kube-node-lease   Active   1h
kube-public       Active   1h
kube-system       Active   1h
```

2. ```kubectl``` create namespace コマンドを使用して、 **ratingsapp** というアプリケーションの名前空間を作成します。

```Bash
kubectl create namespace ratingsapp
```

名前空間が作成されたことを確認するメッセージが表示されます。

```Output
namespace/ratingsapp created
```


 
