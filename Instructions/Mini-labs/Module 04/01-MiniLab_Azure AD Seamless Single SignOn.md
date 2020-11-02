# ミニラボ: Azure Active Directory のシームレスなシングル サインオン

 


Azure Active Directory (Azure AD) シームレス シングル サインオン (シームレスSSO) は、企業ネットワークに接続されている企業のデスクトップにいるユーザーを自動的にサインインします。シームレスな SSO により、追加のオンプレミス コンポーネントを必要とせずに、ユーザーはクラウドベースのアプリケーションに簡単にアクセスできます。


## 前提条件

このミニラボの前に次の前提条件を用意する必要があるため、このラボは**講師用デモ**として実行されます。

* **Azure AD Connect サーバーの設定**: サインイン方法としてパススルー認証を使用する場合、追加の前提条件チェックは必要ありません。サインイン方法としてパスワードハッシュ同期を使用し、Azure AD Connect と Azure AD の間にファイアウォールがある場合は、次のことを確認してください。

	* Azure AD Connect のバージョン 1.1.644.0 以降を使用しています。
	
	* ファイアウォールまたはプロキシで DNS 許可リストが許可されている場合は、ポート 443 を介して *.msappproxy.net URL への接続を許可します。 


* **ドメイン管理者の資格情報を設定**: 次の条件を満たす Active Directory のフォレストごとにドメイン管理者の資格情報が必要です。

	* Azure AD Connect を介して Azure AD に同期します。
	
	* シームレス SSO を有効にするユーザーが含まれています。

## Azure AD Connect を有効にする

1. [Azure AD Connect](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/whatis-hybrid-identity) を介してシームレスな SSO を有効にします。

	* Azure AD Connect の新規インストールを行う場合は、 [カスタム インストール パス](https://docs.microsoft.com/ja-jp/azure/active-directory/hybrid/how-to-connect-install-custom) を選択してください。「**ユーザー サインイン** 」 ページで、**「シングル サインオンを有効にする 」 オプション**をオンにします。

		![Azure AD Connect: ユーザー サインイン](../../Linked_Image_Files/SSO_demo_image1.png)

	* Azure AD Connect がインストール済みの場合は、「Azure AD Connect 」 の 「**ユーザー サインインの変更** 」 ページを選択し、「**次へ** 」 を選択します。

		![Azure AD Connect: ユーザーのサインインを変更する](../../Linked_Image_Files/SSO_demo_image2.png)

1. **「シングル サインオンを有効にする 」** ページが表示されるまで、ウィザードを続行します。次の条件を満たす Active Directory フォレストごとにドメイン管理者の資格情報を提供します。

	* Azure AD Connect を介して Azure AD に同期します。

	* シームレス SSO を有効にするユーザーが含まれています。

1. ウィザードの完了後、シームレス SSO がテナントで有効になります。

## シームレス SSO が有効であることを確認する

以下の手順に従って、シームレス SSO が正しく有効化されていることを確認します。

1. テナントのグローバル管理者資格情報を使用して [[Azure Active Directory 管理センター](https://aad.portal.azure.com/)] にサインインします。

1. 左側のペインで **「Azure Active Directory 」** を選択します。

1. **「Azure AD Connect 」** を選択します。

1. シームレスなシングル サインオン機能が 「*有効* 」 であることを確認します。

	![Azure  portal: Azure AD Connect ペイン](../../Linked_Image_Files/SSO_demo_image3.png)

>重要

シームレスな SSO は、各 AD フォレストの社内 Active Directory (AD) に AZUREADSSOACC という名前のコンピューター アカウントを作成します。AZUREADSSOACC コンピューター アカウントは、セキュリティ上の理由から強力に保護する必要があります。ドメイン管理者だけがコンピューター アカウントを管理できるようにする必要があります。コンピューター アカウントの Kerberos 委任が無効になっており、Active Directory の他のアカウントが AZUREADSSOACC コンピューター アカウントに対する委任アクセス許可を持っていないかどうかを確認します。コンピューター アカウントは、誤って削除されないように、安全でありドメイン管理者だけがアクセスできる組織単位 (OU) に保存します。
