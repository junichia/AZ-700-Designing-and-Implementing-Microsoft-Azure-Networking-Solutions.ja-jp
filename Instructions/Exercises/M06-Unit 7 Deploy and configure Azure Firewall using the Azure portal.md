---
Exercise:
  title: M06 - ユニット 7 Azure portal を使用して Azure Firewall をデプロイして構成する
  module: 'Module 06 - Design and implement network security '
---

# <a name="m06-unit-7-deploy-and-configure-azure-firewall-using-the-azure-portal"></a>M06-ユニット 7 Azure portal を使用して Azure Firewall をデプロイして構成する

Contoso のネットワーク セキュリティ チームの一員であるあなたの次のタスクは、特定の Web サイトへのアクセスを許可または拒否するファイアウォール規則を作成することです。 以下の手順では、環境準備タスクとしてリソース グループ、仮想ネットワークとサブネット、仮想マシンを作成した後、ファイアウォールとファイアウォール ポリシーをデプロイし、既定のルートとアプリケーション、ネットワーク、DNAT 規則を構成して、最後にファイアウォールをテストします。

この演習では、以下のことを行います。

+ タスク 1: リソース グループを作成する
+ タスク 2: 仮想ネットワークとサブネットを作成する
+ タスク 3: 仮想マシンを作成する
+ タスク 4: ファイアウォールとファイアウォール ポリシーをデプロイする
+ タスク 5: 既定のルートを作成する
+ タスク 6: アプリケーション規則を構成する
+ タスク 7: ネットワーク規則を構成する
+ タスク 8: 送信先 NAT (DNAT) 規則を構成する
+ タスク 9: サーバーのネットワーク インターフェイスのプライマリおよびセカンダリ DNS アドレスを変更する
+ タスク 10: ファイアウォールをテストする
+ タスク 11: リソースをクリーンアップする


#### <a name="estimated-time-60-minutes"></a>予想所要時間: 60 分

## <a name="task-1-create-a-resource-group"></a>タスク 1: リソース グループを作成する

このタスクでは、新しいリソース グループを作成します。

1. Azure アカウントにログインします。

2. Azure portal のホーム ページで、**[リソース グループ]** を選択します。

3. **[作成]** をクリックします。 

4. **[基本]** タブで、**[リソース グループ]** に「**Test-FW-RG**」と入力します。

5. **[リージョン]** で、一覧から自分のリージョンを選択します。

   ![Azure Firewall 用のリソース グループを作成します](../media/create-resource-group-for-azure-firewall.png)

6. **[Review + create](レビュー + 作成)** をクリックします。

7. **Create** をクリックしてください。

 

## <a name="task-2-create-a-virtual-network-and-subnets"></a>タスク 2: 仮想ネットワークとサブネットを作成する

このタスクでは、1 つの仮想ネットワークと 2 つのサブネットを作成します。

1. Azure portal のホーム ページで検索ボックスに「**仮想ネットワーク**」と入力し、表示されたら **[仮想ネットワーク]** を選択します。

2. **[作成]** をクリックします。

3. 前に作成した **Test-FW-RG** リソース グループを選択します。

4. **[名前]** ボックスに「**Test-FW-VN**」と入力します。

   ![仮想ネットワークの作成 - [基本] タブ](../media/create-vnet-basics-for-azure-firewall.png)

5. **[次: IP アドレス]** をクリックします。 既定値として既に表示されていない場合は、IPv4 アドレス空間「10.0.0.0/16」を入力します。 

6. **[サブネット名]** で、**[default]** という単語をクリックします。

7. **[サブネットの編集]** ダイアログ ボックスで、名前を「**AzureFirewallSubnet**」に変更します。

8. **[サブネット アドレス範囲]** を「**10.0.1.0/26**」に変更します。

9. **[保存]** をクリックします。

10. **[サブネットの追加]** をクリックして、別のサブネットを作成します。この後で作成するワークロード サーバーを、そこでホストします。


    ![サブネットの追加](../media/add-workload-subnet.png)
    
11. **[サブネットの編集]** ダイアログ ボックスで、名前を「**Workload-SN**」に変更します。

12. **[サブネット アドレス範囲]** を「**10.0.2.0/24**」に変更します。

13. **[追加]** をクリックします。

14. **[Review + create](レビュー + 作成)** をクリックします。

15. **Create** をクリックしてください。

 

## <a name="task-3-create-a-virtual-machine"></a>タスク 3: 仮想マシンを作成する

このタスクでは、ワークロードの仮想マシンを作成し、前に作成した Workload-SN サブネットに配置します。

1. Azure portal で、**[Cloud Shell]** ペイン内に **PowerShell** セッションを開きます。

2. [Cloud Shell] ペインのツール バーで、[ファイルのアップロード/ダウンロード] アイコンを選択し、ドロップダウン メニューで [アップロード] を選択して、**firewall.json** と **firewall.parameters.json** の各ファイルを、ソース フォルダー **F:\Allfiles\Exercises\M06** から Cloud Shell のホーム ディレクトリに 1 つずつアップロードします。

3. 次の ARM テンプレートをデプロイして、この演習に必要な VM を作成します。

   ```powershell
   $RGName = "Test-FW-RG"
   
   New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile firewall.json -TemplateParameterFile firewall.parameters.json
   ```
  
4. デプロイが完了したら、Azure portal のホーム ページに移動し、**[仮想マシン]** を選択します。

5. 仮想マシンが作成されていることを確認してください。

6. VM のデプロイが完了したら、 **[リソースに移動]** を選びます。

7. **Srv-Work** の **[概要]** ページの右側にある **[ネットワーク]** で、この VM の **[プライベート IP アドレス]** (例: **10.0.2.4**) を記録します。
 

## <a name="task-4-deploy-the-firewall-and-firewall-policy"></a>タスク 4: ファイアウォールとファイアウォール ポリシーをデプロイする

このタスクでは、ファイアウォール ポリシーを構成して仮想ネットワークにファイアウォールをデプロイします。

1. Azure portal のホーム ページで **[リソースの作成]** を選択し、検索ボックスに「**ファイアウォール**」と入力して、表示されたら **[ファイアウォール]** を選択します。

2. **[ファイアウォール]** ページで、**[作成]** をクリックします。

3. **[基本]** タブで、次の表の情報を使用してファイアウォールを作成します。

   | **設定**          | **Value**                                                    |
   | -------------------- | ------------------------------------------------------------ |
   | サブスクリプション         | サブスクリプションを選択します。                                     |
   | Resource group       | **Test-FW-RG**                                               |
   | ファイアウォール名        | **Test-FW01**                                                |
   | リージョン               | 自分のリージョン                                                  |
   | ファイアウォール SKU        | **Standard**                                                 |
   | ファイアウォール管理  | **ファイアウォール ポリシーを使用してこのファイアウォールを管理する**            |
   | ファイアウォール ポリシー      | **[新規追加]** を選択します<br />名前: **fw-test-pol**<br />リージョン: **自分のリージョン** |

   ![新しいファイアウォール ポリシーを作成します](../media/create-firewall-policy.png)

   | 仮想ネットワークの選択 | **[既存のものを使用]**                         |
   | ------------------------ | ---------------------------------------- |
   | 仮想ネットワーク          | **Test-FW-VN**                           |
   | パブリック IP アドレス        | **[新規追加]** を選択します<br />名前: **fw-pip** |


   ![ファイアウォールにパブリック IP アドレスを追加します](../media/assign-public-ip-to-firewall.png)

4. すべての設定を調べて、次のスクリーンショットと一致していることを確認します。

   ![ファイアウォールの作成 - 設定の確認](../media/review-all-configurations-for-firewall.png)

5. **[Review + create](レビュー + 作成)** をクリックします。

6. **[作成]** をクリックし、ファイアウォールのデプロイが完了するまで待ちます。

7. ファイアウォールのデプロイが完了したら、**[リソースに移動]** をクリックします。

8. **Test-FW01** の **[概要]** ページの右側で、このファイアウォールの **[ファイアウォールのプライベート IP]** (例: **10.0.1.4**) を記録します。

9. 左側のメニューで、**[設定]** の **[パブリック IP 構成]** をクリックします。

10. **fw-pip** パブリック IP の構成で、**[IP アドレス]** のアドレスを記録します (例: **20.90.136.51**)。

 

## <a name="task-5-create-a-default-route"></a>タスク 5: 既定のルートを作成する

このタスクでは、Workload-SN サブネットで、アウトバウンドの既定ルートがファイアウォールを通過するように構成します。

1. Azure portal のホーム ページで **[リソースの作成]** を選択し、検索ボックスに「**ルート**」と入力して、表示されたら **[ルート テーブル]** を選択します。

2. **[ルート テーブル]** ページで、**[作成]** をクリックします。

3. **[基本]** タブで、次の表の情報を使用して新しいルート テーブルを作成します。

   | **設定**              | **Value**                |
   | ------------------------ | ------------------------ |
   | サブスクリプション             | サブスクリプションを選択します。 |
   | Resource group           | **Test-FW-RG**           |
   | リージョン                   | 自分のリージョン              |
   | 名前                     | **Firewall-route**       |
   | ゲートウェイのルートを伝達する | **あり**                  |


4. **[Review + create](レビュー + 作成)** をクリックします。

5. **Create** をクリックしてください。

   ![ルート テーブルの作成](../media/create-route-table.png)

6. デプロイが完了したら、**[リソースに移動]** を選択します。

7. **Firewall-route** のページの **[設定]** で、**[サブネット]** をクリックして、**[関連付け]** をクリックします。

8. **[仮想ネットワーク]** で、**Test-FW-VN** を選択します。

9. **[サブネット]** で、**Workload-SN** を選択します。 必ずこのルートの Workload-SN サブネットのみを選択してください。それ以外の場合、ファイアウォールが正常に動作しません。

10. **[OK]** をクリックします。

11. **[設定]** で **[ルート]** を選択してから、**[追加]** をクリックします。

12. **[ルート名]** に「**fw-dg**」と入力します。

13. **[アドレス プレフィックス送信先]** に「**0.0.0.0/0**」と入力します。

14. **[ネクスト ホップの種類]** で、**[仮想アプライアンス]** を選択します。

15. **[ネクスト ホップ アドレス]** に、前に記録したファイアウォールのプライベート IP アドレスを入力します (例: **10.0.1.4**)

16. **[追加]** をクリックします。

    ![ファイアウォール ルートを追加します](../media/add-firewall-route.png)

 

## <a name="task-6-configure-an-application-rule"></a>タスク 6: アプリケーション規則を構成する

このタスクでは、www.google.com へのアウトバウンド アクセスを許可するアプリケーション規則を追加します。

1. Azure portal のホーム ページで、**[すべてのリソース]** を選択します。

2. リソースの一覧で、自分のファイアウォール ポリシー **fw-test-pol** をクリックします。

3. **[設定]** で **[Application Rules](アプリケーション規則)** をクリックします。

4. **[規則コレクションの追加]** をクリックします。

5. **[規則コレクションの追加]** ページで、次の表の情報を使用して新しいアプリケーション規則を作成します。

   | **設定**            | **Value**                                 |
   | ---------------------- | ----------------------------------------- |
   | 名前                   | **App-Coll01**                            |
   | 規則コレクションの種類   | **アプリケーション**                           |
   | 優先度               | **200**                                   |
   | 規則コレクション アクション | **許可**                                 |
   | 規則コレクション グループ  | **DefaultApplicationRuleCollectionGroup** |
   | **[規則] セクション**      |                                           |
   | 名前                   | **Allow-Google**                          |
   | 変換元の型            | **IP アドレス**                            |
   | source                 | **10.0.2.0/24**                           |
   | Protocol               | **http、https**                            |
   | 変換先の型       | **FQDN**                                  |
   | 宛先            | **www.google.com**                        |


   ![アプリケーション規則コレクションを追加する](../media/add-an-application-rule-for-firewall.png)

6. **[追加]** をクリックします。

 

## <a name="task-7-configure-a-network-rule"></a>タスク 7: ネットワーク規則を構成する

このタスクでは、ポート 53 (DNS) で 2 つの IP アドレスへのアウトバウンド アクセスを許可するネットワーク規則を追加します。

1. **fw-test-pol** のページで、**[設定]** の下の **[ネットワーク ルール]** をクリックします。

2. **[規則コレクションの追加]** をクリックします。

3. **[規則コレクションの追加]** ページで、次の表の情報を使用して新しいネットワーク規則を作成します。

   | **設定**            | **Value**                                                    |
   | ---------------------- | ------------------------------------------------------------ |
   | 名前                   | **Net-Coll01**                                               |
   | 規則コレクションの種類   | **Network**                                                  |
   | 優先度               | **200**                                                      |
   | 規則コレクション アクション | **許可**                                                    |
   | 規則コレクション グループ  | **DefaultNetworkRuleCollectionGroup**                        |
   | **[規則] セクション**      |                                                              |
   | 名前                   | **Allow-DNS**                                                |
   | 変換元の型            | **IP アドレス**                                               |
   | source                 | **10.0.2.0/24**                                              |
   | Protocol               | **UDP**                                                      |
   | ターゲット ポート      | **53**                                                       |
   | 変換先の型       | **IP アドレス**                                               |
   | 宛先            | **209.244.0.3、209.244.0.4**<br />これらは、Century Link によって運用されているパブリック DNS サーバーです |


    ![ネットワーク規則コレクションを追加する](../media/add-a-network-rule-for-firewall.png)

4. **[追加]** をクリックします。

 

## <a name="task-8-configure-a-destination-nat-dnat-rule"></a>タスク 8: 送信先 NAT (DNAT) 規則を構成する

このタスクでは、ファイアウォール経由でリモート デスクトップを Srv-Work 仮想マシンに接続できるようにする DNAT 規則を追加します。

1. **fw-test-pol** のページで、**[設定]** の下の **[DNAT Rules](DNAT 規則)** をクリックします。

2. **[規則コレクションの追加]** をクリックします。

3. **[規則コレクションの追加]** ページで、次の表の情報を使用して新しい DNAT 規則を作成します。

   | **設定**           | **Value**                                                    |
   | --------------------- | ------------------------------------------------------------ |
   | 名前                  | **rdp**                                                      |
   | 規則コレクションの種類  | **DNAT**                                                     |
   | 優先度              | **200**                                                      |
   | 規則コレクション グループ | **DefaultDnatRuleCollectionGroup**                           |
   | **[規則] セクション**     |                                                              |
   | 名前                  | **rdp-nat**                                                  |
   | 変換元の型           | **IP アドレス**                                               |
   | source                | *                                                            |
   | Protocol              | **TCP**                                                      |
   | ターゲット ポート     | **3389**                                                     |
   | 変換先の型      | **IP アドレス**                                               |
   | 宛先           | 前に記録した **fw-pip** のファイアウォール パブリック IP アドレスを入力します。<br />**例 - 20.90.136.51** |
   | 変換されたアドレス    | 前に記録した **Srv-Work** のプライベート IP アドレスを入力します。<br />**例 - 10.0.2.4** |
   | 変換されたポート       | **3389**                                                     |


        ![Add a DNAT rule collection](../media/add-a-dnat-rule.png)

4. **[追加]** をクリックします。

 

## <a name="task-9-change-the-primary-and-secondary-dns-address-for-the-serversnetwork-interface"></a>タスク 9: サーバーのネットワーク インターフェイスのプライマリおよびセカンダリ DNS アドレスを変更する

この演習でのテストのため、このタスクでは、Srv-Work サーバーのプライマリおよびセカンダリ DNS アドレスを構成します。 ただし、これは Azure Firewall での一般的な要件ではありません。

1. Azure portal のホーム ページで、**[リソース グループ]** を選択します。

2. リソース グループの一覧で、自分のリソース グループ **Test-FW-RG** をクリックします。

3. このリソース グループのリソースの一覧で、**Srv-Work** 仮想マシンのネットワーク インターフェイスを選択します (たとえば **srv-work350**)。

   ![リソース グループで NIC を選択します](../media/change-dns-servers-srv-work-nic-1.png)

4. **[設定]** で、 **[DNS サーバー]** を選択します。

5. **[DNS サーバー]** で、 **[カスタム]** を選択します。

6. **[DNS サーバーの追加]** テキスト ボックスに「**209.244.0.3**」と入力し、次のテキスト ボックスに「**209.244.0.4**」と入力します。

7. **[保存]** を選択します。

   ![NIC で DNS サーバーを変更します](../media/change-dns-servers-srv-work-nic-2.png)

8. **Srv-Work** 仮想マシンを再起動します。

 

## <a name="task-10-test-the-firewall"></a>タスク 10: ファイアウォールをテストする

この最後のタスクでは、ファイアウォールをテストして、規則が正しく構成され、想定どおりに動作していることを確認します。 この構成により、ファイアウォールのパブリック IP アドレスを介して、ファイアウォール経由で Srv-Work 仮想マシンにリモート デスクトップ接続できます。

1. お使いの PC で **[リモート デスクトップ接続]** を開きます。

2. **[コンピューター]** ボックスに、ファイアウォールのパブリック IP アドレス (例: **20.90.136.51**) に続けて「**:3389**」と入力します (例: **20.90.136.51:3389**)。

3. **[ユーザー名]** ボックスに「**TestUser**」と入力します。

4. **[Connect]** をクリックします。

   ![ファイアウォールのパブリック IP アドレスへの RDP 接続](../media/remote-desktop-connection-1.png)

5. **[資格情報を入力してください]** ダイアログ ボックスで、**TestPa$$w0rd!** というパスワードを使用して、**Srv-Work** サーバーの仮想マシンにログインします。

6. **[OK]** をクリックします。

7. 証明書メッセージで **[はい]** をクリックします。

8. Internet Explorer を開き、**https://www.google.com** にアクセスします。

9. **[セキュリティ アラート]** ダイアログ ボックスで、**[OK]** をクリックします。

10. Internet Explorer でセキュリティ アラートのポップアップが表示される場合は、**[閉じる]** をクリックします。

11. Google のホーム ページが表示されます。

    ![Srv-work サーバーでの RDP セッション - google.com が表示されたブラウザー](../media/remote-desktop-connection-2.png)

12. **https://www.microsoft.com** にアクセスします。

13. ファイアウォールによってブロックされます。

    ![Srv-work サーバーでの RDP セッション - google.com でブロックされたブラウザー](../media/remote-desktop-connection-3.png)

 
## <a name="task-11-clean-up-resources"></a>タスク 11: リソースをクリーンアップする 

>**注**:新規に作成し、使用しなくなったすべての Azure リソースを削除することを忘れないでください。 使用していないリソースを削除することで、予期しない料金が発生しなくなります。

1. Azure portal で、**[Cloud Shell]** ペイン内に **PowerShell** セッションを開きます。

1. 次のコマンドを実行して、このモジュールのラボ全体を通して作成したすべてのリソース グループを削除します。

   ```powershell
   Remove-AzResourceGroup -Name 'Test-FW-RG' -Force -AsJob
   ```

    >**注**:このコマンドは非同期で実行されるため (-AsJob パラメーターによって決定されます)、同じ PowerShell セッション内で直後に別の PowerShell コマンドを実行できますが、リソース グループが実際に削除されるまでに数分かかります。
