# ミニラボ: Azure Container Instances の実行

このラボでは、Azure でコンテナーを作成し、完全修飾ドメイン名 (FQDN) を使用してインターネットに公開します。

Azure Container Instances は、簡単なアプリケーション、タスク自動化、ジョブのビルドなど、分離されたコンテナーで操作できるシナリオに適しています。利点のいくつかは次のとおりです。

* **高速起動**: コンテナーを数秒で起動します。

* **1 秒あたりの請求**: 費用が発生するのはコンテナーの実行中だけです。

* **ハイパーバイザーレベルのセキュリティ**: アプリケーションを VM にある場合と同じように完全に分離します。

* **カスタム サイズ**: CPU コアとメモリの値を正確に指定します。

* **永続的ストレージ**: 取得するコンテナーに Azure ファイルの共有を直接マウントし、状態を取得および永続化します。

* **Linux および Windows**: 同じ API で Windows と Linux コンテナーの両方をスケジュールできます。

複数のコンテナー間でのサービスの検出、自動スケーリング、アプリケーションの調整されたアップグレードなど、コンテナーの完全なオーケストレーションが必要な場合については、Azure Kubernetes Service (AKS) をお勧めします。

## コンテナーの作成

1. Azure サブスクリプションを使用して、[https://portal.azure.com ](https://portal.azure.com/)で Azure portal にサインインします。

2. Cloud Shell アイコンを使用して、Azure portal から Azure Cloud Shell を開きます。

![画像 7](../../Linked_Image_Files/demo_Azure_containers_image1.png)

3. **learn-deploy-aci-rg** という名前の新しいリソース グループを作成して、モジュールが終了したときにこれらのリソースを簡単にクリーン アップできるようにします。別のリソース グループ名を選択した場合は、このモジュールの残りの演習で使用する名前を覚えておいてください。また、リソース グループを作成するリージョン (**たとえば、米国東部**) を選択する必要があります。

```Azure CLI
az group create --name learn-deploy-aci-rg --location eastus
```

コンテナーを作成するには、名前、Docker イメージ、Azure リソース グループを ```az container create``` コマンドに指定します。必要に応じて、DNS 名ラベルを指定することにより、コンテナーをインターネットに公開できます。この例では、小さな Web アプリをホストするコンテナーをデプロイします。また、イメージを配置する場所を選択することもできます。 **米国東部**リージョンを使用しますが、近い場所に変更することもできます。

4. コンテナーをインターネットに公開するために DNS 名を指定します。DNS 名は一意である必要があります。学習のために、このコマンドを Cloud Shell から実行して、一意の名前を保持する Bash 変数を作成します。

```Azure CLI
DNS_NAME_LABEL=aci-demo-$RANDOM
```

5. 次の ```az container create``` コマンドを実行してコンテナー インスタンスを起動します。

```Azure
az container create \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --image microsoft/aci-helloworld \
  --ports 80 \
  --dns-name-label $DNS_NAME_LABEL \
  --location eastus
```

```$DNS_NAME_LABEL``` は、DNS 名を指定します。イメージ名 **microsoft/aci-helloworld** は、基本的な Node.js Web アプリケーションを実行する Docker Hub でホストされている Docker イメージを参照します。

6. ```az container create``` コマンドが完了したら、```az container show``` を実行してステータスを確認します。

```Azure CLI
az container show \
  --resource-group learn-deploy-aci-rg \
  --name mycontainer \
  --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
  --out table
```

コンテナーの完全修飾ドメイン名 (FQDN) とそのプロビジョニングの状態が表示されます。次に例を示します。

```Output
FQDN ProvisioningState

-------------------------------------- -------------------

aci-demo.eastus.azurecontainer.io Succeeded
```

コンテナーの状態が **Creating** の場合は、しばらく待ってから、 **Succeeded** 状態になるまでコマンドを再び実行します。

7. ブラウザーからコンテナーの FQDN に移動して、実行中であることを確認します。これを参照します。

![ブラウザーで実行されているサンプル Node.js コンテナー アプリのスクリーンショット。](../../Linked_Image_Files/demo_Azure_containers_image2.png)
