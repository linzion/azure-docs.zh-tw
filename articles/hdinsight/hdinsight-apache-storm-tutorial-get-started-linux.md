---
title: "HDInsight 上 Apache Storm 的 Storm-starter 範例 | Microsoft Docs"
description: "了解如何使用 Apache Storm 和 HDInsight 上的 storm-starter 範例來執行巨量資料分析及處理資料。"
keywords: "storm-starter, apache storm 範例"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: d710dcac-35d1-4c27-a8d6-acaf8146b485
ms.service: hdinsight
ms.devlang: java
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 05/25/2017
ms.author: larryfr
ms.custom: H1Hack27Feb2017,hdinsightactive,hdiseo17may2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 74f34bdbf5707510c682814716aa0b95c19a5503
ms.openlocfilehash: 5936a45973eb41d2c5b3a1a47c663e393e628f54
ms.contentlocale: zh-tw
ms.lasthandoff: 06/09/2017

---
#<a name="get-started-with-apache-storm-on-hdinsight-using-the-storm-starter-examples"></a>使用 storm-starter 範例在 HDInsight 上開始使用 Apache Storm

了解如何使用 storm-starter 範例，在 HDInsight 中使用 Apache Storm。

Apache Storm 是一個可處理資料串流的分散式、容錯、即時的運算系統。 在 Storm on Azure HDInsight 中，您可以建立雲端式 Storm 叢集，以執行即時的巨量資料分析。

> [!IMPORTANT]
> Linux 是唯一使用於 HDInsight 3.4 版或更新版本的作業系統。 如需詳細資訊，請參閱 [Windows 上的 HDInsight 淘汰](hdinsight-component-versioning.md#hdi-version-33-nearing-retirement-date)。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **Azure 訂用帳戶**。 請參閱[取得 Azure 免費試用](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。

* **熟悉 SSH 和 SCP**。 如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="create-a-storm-cluster"></a>建立 Storm 叢集

請使用下列步驟建立 Storm on HDInsight 叢集：

1. 從 [Azure 入口網站](https://portal.azure.com)選取 [+ 新增]、[情報 + 分析] 及 [HDInsight]，然後選取 [HDInsight]。

    ![建立 HDInsight 叢集](./media/hdinsight-apache-storm-tutorial-get-started-linux/create-hdinsight.png)

2. 在 [基本概念] 刀鋒視窗中，輸入下列資訊：

    * **叢集名稱**︰HDInsight 叢集的名稱。
    * **訂用帳戶**：選取要使用的訂用帳戶。
    * **叢集登入使用者名稱**和**叢集登入密碼**：透過 HTTPS 存取叢集時使用的登入資訊。 您會使用這些認證來存取例如 Ambari Web UI 或 REST API 等服務。
    * **安全殼層 (SSH) 使用者名稱**：透過 SSH 存取叢集時使用的登入資訊。 依預設，密碼要與叢集登入密碼相同。
    * **資源群組**：在其中建立叢集的資源群組。
    * **位置**：在其中建立叢集的 Azure 區域。

    ![選取訂用帳戶](./media/hdinsight-apache-storm-tutorial-get-started-linux/hdinsight-basic-configuration.png)

3. 選取 [叢集類型]，並且在 [叢集組態] 刀鋒視窗中設定下列值︰

    * **叢集類型**：Storm

    * **作業系統**：Linux

    * **版本**：Storm 1.0.1 (HDI 3.5)

    * **叢集層**：標準

    最後，使用 [選取] 按鈕來儲存設定。

    ![選取叢集類型](./media/hdinsight-apache-storm-tutorial-get-started-linux/set-hdinsight-cluster-type.png)

4. 選取叢集類型之後，請使用 [選取] 按鈕來設定叢集類型。 接下來，使用 [下一步] 按鈕來完成基本組態。

5. 從 [儲存體] 刀鋒視窗中，選取或建立儲存體帳戶。 本文件的步驟是，將此刀鋒視窗中的其他欄位保留為預設值。 使用 [下一步] 按鈕以儲存儲存體組態。

    ![設定 HDInsight 的儲存體帳戶](./media/hdinsight-apache-storm-tutorial-get-started-linux/set-hdinsight-storage-account.png)

6. 從 [摘要] 刀鋒視窗中，檢閱叢集組態。 使用 [編輯] 連結來變更所有不正確的設定。 最後，使用 [建立] 按鈕來建立叢集。

    ![叢集組態摘要](./media/hdinsight-apache-storm-tutorial-get-started-linux/hdinsight-configuration-summary.png)

    > [!NOTE]
    > 建立叢集可能需要花費 20 分鐘的時間。

## <a name="run-a-storm-starter-sample-on-hdinsight"></a>在 HDInsight 上執行 storm-starter 範例

1. 使用 SSH 連線到 HDInsight 叢集

        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net

    如果您已經使用密碼保護您 SSH 使用者帳戶的安全，系統會提示您輸入密碼。 如果您使用的是公開金鑰，您需要使用 `-i` 參數來指定對應的私密金鑰。 例如， `ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`。

    如需相關資訊，請參閱[搭配 HDInsight 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)。

2. 使用下列命令以啟動範例拓撲：

        storm jar /usr/hdp/current/storm-client/contrib/storm-starter/storm-starter-topologies-*.jar org.apache.storm.starter.WordCountTopology wordcount

    > [!NOTE]
    > 在舊版 HDInsight 上，拓撲的類別名稱是 `storm.starter.WordCountTopology` 而不是 `org.apache.storm.starter.WordCountTopology`。

    此命令會在叢集上使用 'wordcount' 的易記名稱，啟動範例 WordCount 拓撲。 命令會隨機產生句子，並計算句子中每個字詞的出現次數。

    > [!NOTE]
    > 將您自己的拓撲提交至叢集時，必須先複製包含叢集的 jar 檔案，再使用 `storm` 命令。 使用 `scp` 命令來複製檔案。 例如， `scp FILENAME.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.jar`
    >
    > WordCount 範例和其他 storm-starter 範例都已經包含在叢集中，位置是 `/usr/hdp/current/storm-client/contrib/storm-starter/`。

如果您有興趣檢視 storm-starter 範例的來源，可以在 [https://github.com/apache/storm/tree/1.0.x-branch/examples/storm-starter](https://github.com/apache/storm/tree/1.0.x-branch/examples/storm-starter) 找到程式碼。 這個連結是 Storm 1.0.x，隨附於 HDInsight 3.5。 如需其他 Storm 版本，請使用頁面頂端的 __Branch__ 分支 按鈕來選取其他 Storm 版本。

## <a name="monitor-the-topology"></a>監視拓撲

Storm UI 提供 Web 介面來處理執行中的拓撲，包含在您的 HDInsight 叢集中。

使用下列步驟以 Storm UI 監視拓撲。

1. 若要顯示 Storm UI，請開啟網頁瀏覽器並前往 https://CLUSTERNAME.azurehdinsight.net/stormui。 將 **CLUSTERNAME** 取代為您叢集的名稱。

    > [!NOTE]
    > 如果要求您提供使用者名稱和密碼，請輸入叢集系統管理員 (admin) 和建立叢集時使用的密碼。

2. 在 [拓撲摘要] 下，選取 [名稱] 欄中的 [wordcount] 項目。 關於拓撲的詳細資訊隨即顯示。

    ![包含 storm-starter WordCount 拓樸資訊的 Storm 儀表板。](./media/hdinsight-apache-storm-tutorial-get-started-linux/topology-summary.png)

    此頁面提供以下資訊：

    * **拓撲統計資料 (Topology stats)** ：拓撲效能的基本資訊，已整理為時間範圍。

        > [!NOTE]
        > 選取特定的時間範圍，可以變更頁面中其他區段所顯示之資訊的時間範圍。

    * **Spouts** ：spout 的基本資訊，包括每個 spout 傳回的最後一個錯誤。

    * **Bolts** ：bolt 的基本資訊。

    * **拓撲組態 (Topology configuration)** ：拓撲組態的詳細資訊。

    此頁面也提供可對拓撲採取的動作：

    * **啟用** ：繼續處理已停用的拓撲。

    * **停用** ：暫停執行中拓撲。

    * **重新平衡** ：調整拓撲的平行處理原則。 變更叢集中的節點數目之後，您應該重新平衡執行中拓撲。 重新平衡調整平行處理原則，以彌補叢集中增加/減少的節點數目。 如需詳細資訊，請參閱[了解 Storm 拓撲的平行處理原則](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。

    * **終止 (Kill)** ：在指定的逾時之後終止 Storm 拓撲。

3. 在此頁面中，選取 [Spouts] 或 [Bolts] 區段中的一個項目。 關於所選元件的詳細資訊隨即顯示。

    ![包含所選元件相關資訊的 Storm 儀表板。](./media/hdinsight-apache-storm-tutorial-get-started-linux/component-summary.png)

    此頁面會顯示以下資訊：

    * **Spout/Bolt 統計資料 (Spout/Bolt stats)** ：元件效能的基本資訊，已整理為時間範圍。

        > [!NOTE]
        > 選取特定的時間範圍，可以變更頁面中其他區段所顯示之資訊的時間範圍。

    * **輸入統計資料 (Input stats)** (僅 bolt)：提供的資訊是關於產生 bolt 所使用之資料的元件。

    * **輸出統計資料 (Output stats)** ：此 bolt 發出之資料的資訊。

    * **執行程式** ：此元件之執行個體的資訊。

    * **錯誤** ：此元件產生的錯誤。

4. 檢視 spout 或 bolt 的詳細資料時，請在 [執行程式] 區段的 [連接埠] 欄中選取一個項目，以檢視特定元件執行個體的詳細資料。

        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["with"]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["nature"]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [snow]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [snow, 747293]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [white]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [white, 747293]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [seven]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [seven, 1493957]

    在此範例中，「七」這個字出現 1493957 次。 此計數就是啟動拓撲後，該字所出現的次數。

## <a name="stop-the-topology"></a>停止拓撲

返回 word-count 拓撲的 [拓撲摘要] 頁面，然後選取 [拓撲動作] 區段中的 [終止] 按鈕。 出現提示時，請先輸入要等候 10 秒，再停止拓撲。 逾時期限過後，當您瀏覽儀表板的 [Storm UI]  區段時，就不會再看到拓撲。

## <a name="delete-the-cluster"></a>刪除叢集

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

如果您在建立 HDInsight 叢集時遇到問題，請參閱[存取控制需求](hdinsight-administer-use-portal-linux.md#create-clusters)。

## <a id="next"></a>接續步驟

在本 Apache Storm 教學課程中，您已了解使用 Storm on HDInsight 的基本概念。 接下來，了解如何 [使用 Maven 開發 Java 型拓撲](hdinsight-storm-develop-java-topology.md)。

如果您已熟悉開發 Java 型拓撲，而且想要將現有的拓撲部署至 HDInsight，請參閱 [部署和管理 HDInsight 上的 Apache Storm 拓撲](hdinsight-storm-deploy-monitor-topology-linux.md)。

如果您是 .NET 開發人員，您可以使用 Visual Studio 建立 C# 或混合式 C#/Java 拓撲。 如需詳細資訊，請參閱 [使用 Visual Studio 的 Hadoop 工具開發 Apache Storm on HDInsight 的 C# 拓撲](hdinsight-storm-develop-csharp-visual-studio-topology.md)。

如需可搭配 Storm on HDInsight 使用的拓撲範例，請參閱下列範例︰

* [Storm on HDInsight 的範例拓撲](hdinsight-storm-example-topology.md)

[apachestorm]: https://storm.incubator.apache.org
[stormdocs]: http://storm.incubator.apache.org/documentation/Documentation.html
[stormstarter]: https://github.com/apache/storm/tree/master/examples/storm-starter
[stormjavadocs]: https://storm.incubator.apache.org/apidocs/
[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[preview-portal]: https://portal.azure.com/

