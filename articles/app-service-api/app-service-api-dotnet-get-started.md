---
title: "在 App Service 中開始使用 API Apps 和 ASP.NET | Microsoft Docs"
description: "了解如何在 Azure App Service 使用 Visual Studio 2015 建立、部署和取用 ASP.NET API 應用程式。"
services: app-service\api
documentationcenter: .net
author: alexkarcher-msft
manager: erikre
editor: 
ms.assetid: ddc028b2-cde0-4567-a6ee-32cb264a830a
ms.service: app-service-api
ms.workload: na
ms.tgt_pltfrm: dotnet
ms.devlang: na
ms.topic: hero-article
ms.date: 09/20/2016
ms.author: alkarche
translationtype: Human Translation
ms.sourcegitcommit: b1a633a86bd1b5997d5cbf66b16ec351f1043901
ms.openlocfilehash: c7b4e39e01ae335c3e6a5cf9cb1efe8a64490e35
ms.lasthandoff: 02/16/2017


---
# <a name="get-started-with-api-apps-aspnet-and-swagger-in-azure-app-service"></a>在 Azure App Service 中開始使用 API Apps、ASP.NET 和 Swagger
[!INCLUDE [selector](../../includes/app-service-api-get-started-selector.md)]

本文是一系列教學課程的第一篇，文章中將會說明如何使用 Azure App Service 中有助於開發和裝載 RESTful API 的功能。  本教學課程涵蓋 Swagger 格式 API 中繼資料的支援。

您將了解：

* 如何透過 Visual Studio 2015 的內建工具，在 Azure App Service 中建立及部署 [API 應用程式](app-service-api-apps-why-best-platform.md) 。
* 如何使用 Swashbuckle NuGet 封裝來動態產生 Swagger API 中繼資料，以便自動進行 API 探索。
* 如何使用 Swagger API 中繼資料，來自動產生 API 應用程式的用戶端程式碼。

## <a name="sample-application-overview"></a>範例應用程式概觀
在本教學課程中，您會使用簡單的待辦事項清單範例應用程式。 此應用程式具有單一頁面應用程式 (SPA) 前端、ASP.NET Web API 中介層和 ASP.NET Web API 資料層。

![API Apps 範例應用程式圖表](./media/app-service-api-dotnet-get-started/noauthdiagram.png)

以下是 [AngularJS](https://angularjs.org/) 前端的螢幕擷取畫面。

![API Apps 範例應用程式待辦事項清單](./media/app-service-api-dotnet-get-started/todospa.png)

此 Visual Studio 方案包含三個專案：

![](./media/app-service-api-dotnet-get-started/projectsinse.png)

* **ToDoListAngular** - 前端：呼叫中介層的 AngularJS SPA。
* **ToDoListAPI** - 中介層：呼叫資料層以對待辦事項項目執行 CRUD 作業的 ASP.NET Web API 專案。
* **ToDoListDataAPI** - 資料層：對待辦事項項目執行 CRUD 作業的 ASP.NET Web API 專案。

三層式架構是許多可使用 API Apps 來實作的架構之一，在此則僅供示範之用。 每一層中的程式碼都會盡可能以最簡單的方式來示範 API Apps 功能；例如，資料層會使用伺服器記憶體而非資料庫來做為持續性機制。

一旦完成本教學課程，您將會擁有兩個在 App Service API 應用程式中於雲端啟動並執行的 Web API 專案。

本系列的下一個教學課程會將 SPA 前端部署至雲端。

## <a name="prerequisites"></a>必要條件
* ASP.NET Web API - 本教學課程指示假設您基本上已了解如何在 Visual Studio 中使用 ASP.NET [Web API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api) 。
* Azure 帳戶 - 您可以[申請免費 Azure 帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)或[啟用 Visual Studio 訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F)。
  
    如果您想要在註冊 Azure 帳戶之前先開始使用 Azure App Service，請移至 [試用 App Service](https://azure.microsoft.com/try/app-service/)。 如此，您可以在 App Service 中立即建立短期的入門應用程式 — **不需信用卡**，不需任何承諾。
* Visual Studio 2015 和 [Azure SDK for .NET](https://azure.microsoft.com/downloads/archive-net-downloads/) - SDK 會自動安裝 Visual Studio 2015 (如果您還沒有它)。
  
  * 在 Visual Studio 中，按一下 [說明] -> [關於 Microsoft Visual Studio]，並確定您已安裝 "Azure App Service Tools v2.9.1" 或更新版本。
    
    ![Azure App Tools 版本](./media/app-service-api-dotnet-get-started/apiversion.png)
    
    > [!NOTE]
    > 視您的電腦上有多少 SDK 相依性而定，安裝 SDK 可能需要很長的時間 (從數分鐘到半小時以上不等)。
    > 
    > 

## <a name="download-the-sample-application"></a>下載範例應用程式
1. 下載 [Azure-Samples/app-service-api-dotnet-to-do-list](https://github.com/Azure-Samples/app-service-api-dotnet-todo-list) 儲存機制。
   
    您可以按一下 [下載 ZIP]  按鈕，或複製本機電腦上的儲存機制。
2. 在 Visual Studio 2015 或 2013 中開啟 ToDoList 解決方案。
   
   1. 您必須信任每個解決方案。
         ![安全性警告](./media/app-service-api-dotnet-get-started/securitywarning.png)
3. 建置解決方案 (CTRL + SHIFT + B) 以還原 NuGet 套件。
   
    如果您想要先查看應用程式會如何運作再加以部署，您可以在本機執行此應用程式。 確定 ToDoListDataAPI 是您的啟始專案並執行此解決方案。 您應該會在瀏覽器中看到 HTTP 403 錯誤。

## <a name="use-swagger-api-metadata-and-ui"></a>使用 Swagger API 中繼資料和 UI
Azure App Service 內建支援 [Swagger 2.0](http://swagger.io/) API 中繼資料。 每個 API 應用程式都可以指定會以 Swagger JSON 格式傳回 API 中繼資料的 URL 端點。 從該端點傳回的中繼資料可以用來產生用戶端程式碼。

ASP.NET Web API 專案可以使用 [Swashbuckle](https://www.nuget.org/packages/Swashbuckle) NuGet 封裝，動態產生 Swagger 中繼資料。 Swashbuckle NuGet 封裝已安裝在您所下載的 ToDoListDataAPI 和 ToDoListAPI 專案中。

在教學課程的這一節中，請看一下所產生的 Swagger 2.0 中繼資料，然後試用以 Swagger 中繼資料為基礎的測試 UI。

1. 將 ToDoListDataAPI 專案 (**而非** ToDoListAPI 專案) 設定為起始專案。
   
    ![將 ToDoDataAPI 設定為啟始專案](./media/app-service-api-dotnet-get-started/startupproject.png)
2. 按 F5 或按一下 [偵錯] > [啟動偵錯] 來以偵錯模式執行專案。
   
    瀏覽器會開啟並顯示 [HTTP 403 錯誤] 頁面。
3. 在瀏覽器的位址列中，於 URL 結尾加入 `swagger/docs/v1` 。 (URL 是 `http://localhost:45914/swagger/docs/v1`。)
   
    這是 Swashbuckle 用來傳回 API 的 Swagger 2.0 JSON 中繼資料的預設 URL。
   
    如果您是使用 Internet Explorer，瀏覽器會提示您下載「v1.json」  檔案。
   
    ![在 IE 中下載 JSON 中繼資料](./media/app-service-api-dotnet-get-started/iev1json.png)
   
    如果您是使用 Chrome、Firefox 或 Edge，瀏覽器會在瀏覽器視窗中顯示 JSON。 不同瀏覽器有不同的 JSON 處理方式，因此瀏覽器視窗看起來可能不會和範例完全相同。
   
    ![Chrome 中的 JSON 中繼資料](./media/app-service-api-dotnet-get-started/chromev1json.png)
   
    下列範例顯示 API 的 Swagger 中繼資料的第一個區段 (包含 Get 方法的定義)。 此中繼資料可驅動您將在下列步驟中使用的 Swagger UI，而您將在本教學課程稍後的章節中使用它來自動產生用戶端程式碼。
   
        {
          "swagger": "2.0",
          "info": {
            "version": "v1",
            "title": "ToDoListDataAPI"
          },
          "host": "localhost:45914",
          "schemes": [ "http" ],
          "paths": {
            "/api/ToDoList": {
              "get": {
                "tags": [ "ToDoList" ],
                "operationId": "ToDoList_GetByOwner",
                "consumes": [ ],
                "produces": [ "application/json", "text/json", "application/xml", "text/xml" ],
                "parameters": [
                  {
                    "name": "owner",
                    "in": "query",
                    "required": true,
                    "type": "string"
                  }
                ],
                "responses": {
                  "200": {
                    "description": "OK",
                    "schema": {
                      "type": "array",
                      "items": { "$ref": "#/definitions/ToDoItem" }
                    }
                  }
                },
                "deprecated": false
              },
4. 關閉瀏覽器，並停止 Visual Studio 偵錯。
5. 在 [方案總管] 的 ToDoListDataAPI 專案中，開啟 *App_Start\SwaggerConfig.cs* 檔案，然後向下捲動至第 174 行並將下列程式碼取消註解。
   
        /*
            })
        .EnableSwaggerUi(c =>
            {
        */
   
    「SwaggerConfig.cs」  檔案建立於您在專案中安裝 Swashbuckle 封裝時。 此檔案會提供數種方式來設定 Swashbuckle。
   
    您已取消註解的程式碼會啟用您將使用於下列步驟中的 Swagger UI。 當您使用 API 應用程式專案範本建立 Web API 專案時，基於安全性考量，預設會註解化此程式碼。
6. 再次執行此專案。
7. 在瀏覽器的位址列中，於 URL 結尾加入 `swagger` 。 (URL 是 `http://localhost:45914/swagger`。)
8. 當 Swagger UI 頁面出現時，按一下 [ToDoList]  以查看可用的方法。
   
    ![Swagger UI 可用的方法](./media/app-service-api-dotnet-get-started/methods.png)
9. 按一下清單中的第一個 [Get]  按鈕。
10. 在 [參數] 區段中，輸入星號來做為 `owner` 參數的值，然後按一下 [立即試用]。
    
    當您在稍後的教學課程中新增驗證時，中介層會對資料層提供實際的使用者識別碼。 現在，當應用程式在未啟用驗證的狀態下執行時，所有工作皆會以星號做為其擁有者識別碼。
    
    ![Swagger UI 試用](./media/app-service-api-dotnet-get-started/gettryitout1.png)
    
    Swagger UI 會呼叫 ToDoList Get 方法並顯示回應碼和 JSON 結果。
    
    ![Swagger UI 試用結果](./media/app-service-api-dotnet-get-started/gettryitout.png)
11. 按一下 [Post]，然後按一下 [模型結構描述] 之下的方塊。
    
    按一下模型結構描述會預先填入輸入方塊，而您可以在其中指定 Post 方法的參數值。 (如果這不適用於 Internet Explorer，請使用不同的瀏覽器或在下一個步驟中手動輸入參數值。)  
    
    ![Swagger UI 試用文章](./media/app-service-api-dotnet-get-started/post.png)
12. 在 `todo` 參數輸入方塊中變更 JSON，讓它看起來如同下列範例，或以您自己的描述文字替代：
    
        {
          "ID": 2,
          "Description": "buy the dog a toy",
          "Owner": "*"
        }
13. 按一下 [立即試用] 。
    
    ToDoList API 會傳回表示成功的 HTTP 204 回應碼。
14. 按一下第一個 [Get] 按鈕，然後在頁面的該區段中按一下 [立即試用] 按鈕。
    
    Get 方法回應現在包含新的待辦事項項目。
15. 選擇性︰也請試用 Put、Delete 和 Get by ID方法。
16. 關閉瀏覽器，並停止 Visual Studio 偵錯。

Swashbuckle 可搭配任何 ASP.NET Web API 專案使用。 如果您要將 Swagger 中繼資料產生新增至現有的專案，只需安裝 Swashbuckle 封裝。

> [!NOTE]
> Swagger 中繼資料包含每個 API 作業的唯一識別碼。 根據預設，Swashbuckle 可能會為 Web API 控制器方法產生重複的 Swagger 作業識別碼。 如果控制器有多載的 HTTP 方法，例如 `Get()` 和 `Get(id)`，就會發生此情況。 如需如何處理多載的相關資訊，請參閱 [自訂 Swashbuckle 產生的 API 定義](app-service-api-dotnet-swashbuckle-customize.md)。 如果您在 Visual Studio 中使用 Azure API 應用程式範本建立 Web API 專案， *SwaggerConfig.cs* 檔案中就會自動新增用來產生唯一作業識別碼的程式碼。  
> 
> 

## <a id="createapiapp"></a> 在 Azure 中建立 API 應用程式並於其中部署程式碼
在本節中，您會使用已整合至 Visual Studio 的 [發佈 Web]  精靈中的 Azure 工具，在 Azure 中建立新的 API 應用程式。 然後您會將 ToDoListDataAPI 專案部署到新的 API 應用程式，並藉由執行 Swagger UI 來呼叫 API。

1. 在 [方案總管] 中，以滑鼠右鍵按一下 ToDoListDataAPI 專案，然後按一下 [發佈]。
   
    ![在 Visual Studio 中按一下 [發佈]](./media/app-service-api-dotnet-get-started/pubinmenu.png)
2. 在 [發佈 Web] 精靈的 [設定檔] 步驟中，按一下 [Microsoft Azure App Service]。
   
   ![在 [發佈 Web] 中按一下 [Azure App Service]](./media/app-service-api-dotnet-get-started/selectappservice.png)
3. 如果您尚未登入，請登入您的 Azure 帳戶；或者如果過期，請重新整理您的認證。
4. 在 [App Service] 對話方塊中，選擇您想要使用的 Azure [訂用帳戶]，然後按一下 [新增]。
   
    ![在 [App Service] 對話方塊中按一下 [新增]](./media/app-service-api-dotnet-get-started/clicknew.png)
   
    [建立 App Service] 對話方塊的 [主控] 索引標籤中隨即出現。
   
    因為您正在部署已安裝 Swashbuckle 的 Web API 專案，所以 Visual Studio 假設您要建立 API 應用程式。 這是以 [API 應用程式名稱] 標題以及 [變更類型] 下拉式清單設為 [API 應用程式] 的這個事實所指出。
   
    ![[App Service] 對話方塊中的應用程式類型](./media/app-service-api-dotnet-get-started/apptype.png)
5. 輸入在 *azurewebsites.net* 網域中唯一的 [API 應用程式名稱]。 您可以接受 Visual Studio 所建議的預設名稱。
   
    如果您輸入的名稱已有他人使用，您會在右邊看到紅色驚嘆號。
   
    API 應用程式的 URL 會是 `{API app name}.azurewebsites.net`。
6. 在 [資源群組] 下拉式清單中，按一下 [新增]，然後輸入 "ToDoListGroup" 或其他您偏好使用的名稱。
   
    資源群組是 Azure 資源的集合，例如 API 應用程式、資料庫、VM 等等。    在本教學課程中，最好建立新的資源群組，因為這麼做即可在一個步驟中輕鬆刪除您為本教學課程建立的所有 Azure 資源。
   
    此方塊可讓您選取現有的[資源群組](../azure-resource-manager/resource-group-overview.md)，或藉由輸入與您的訂用帳戶中任何現有的資源群組不同的名稱，以建立新的資源群組。
7. 按一下 [App Service 方案] 下拉式清單旁邊的 [新增] 按鈕。
   
    螢幕擷取畫面顯示 [API 應用程式名稱]、[訂用帳戶] 和 [資源群組] 的範例值 -- 您的值會有所不同。
   
    ![建立 App Service 對話方塊](./media/app-service-api-dotnet-get-started/createas.png)
   
    在下列步驟中，您會為新的資源群組建立 App Service 方案。 App Service 方案會指定 API 應用程式執行所在的計算資源。 例如，如果您選擇免費層，則 API 應用程式會在共用 VM 上執行，若為某些付費層，它則會在專用 VM 上執行。 如需 App Service 方案的詳細資訊，請參閱 [App Service 方案概觀](../app-service/azure-web-sites-web-hosting-plans-in-depth-overview.md)。
8. 在 [設定 App Service 方案]  對話方塊中，輸入 "ToDoListPlan" 或其他您偏好使用的名稱。
9. 在 [位置]  下拉式清單中，選擇最接近您的位置。
   
    這個設定會指定應用程式將執行所在的 Azure 資料中心。 選擇與您接近的位置以盡量減少 [延遲](http://www.bing.com/search?q=web%20latency%20introduction&qs=n&form=QBRE&pq=web%20latency%20introduction&sc=1-24&sp=-1&sk=&cvid=eefff99dfc864d25a75a83740f1e0090)。
10. 在 [大小] 下拉式清單中，按一下 [免費]。
    
    在本教學課程中，免費定價層會提供足夠的效能。
11. 在 [設定 App Service 方案] 對話方塊中，按一下 [確定]。
    
    ![在 [設定 App Service 方案] 中按一下 [確定]](./media/app-service-api-dotnet-get-started/configasp.png)
12. 在 [建立 App Service] 對話方塊中，按一下 [建立]。
    
    ![在 [建立 App Service] 對話方塊中按一下 [建立]](./media/app-service-api-dotnet-get-started/clickcreate.png)
    
    Visual Studio 會建立 API 應用程式，以及具有 API 應用程式所有必要設定的發佈設定檔。 然後它會開啟 [發佈 Web]  精靈以供您部署專案。
    
    [發佈 Web] 精靈會開啟在 [連線] 索引標籤 (如下所示)。
    
    在 [連線] 索引標籤上，[伺服器] 和 [網站名稱] 設定會指向您的 API 應用程式。 [使用者名稱] 和 [密碼] 是 Azure 為您建立的部署認證。 在部署之後，Visual Studio 會將瀏覽器開啟到 [目的地 URL] \(這是 [目的地 URL] 的唯一目的)。  
13. 按 [下一步] 。
    
    ![在 [發佈 Web] 的 [連線] 索引標籤中按 [下一步]](./media/app-service-api-dotnet-get-started/connnext.png)
    
    下一個索引標籤是 [設定]  索引標籤 (如下所示)。 您可以在此變更組建組態索引標籤，以部署用於 [遠端偵錯](../app-service-web/web-sites-dotnet-troubleshoot-visual-studio.md#remotedebug)的偵錯組建。 此索引標籤也會提供數個 [檔案發佈選項] ：
    
    * 在目的地移除多餘的檔案
    * 在發行期間預先編譯
    * 從 App_Data 資料夾中排除檔案
    
    在本教學課程中，您不需要這些。 如需它們執行了哪些作業的說明，請參閱 [操作說明：在 Visual Studio 中使用單鍵發佈來部署 Web 專案](https://msdn.microsoft.com/library/dd465337.aspx)。
14. 按 [下一步] 。
    
    ![在 [發佈 Web] 的 [設定] 索引標籤中按 [下一步]](./media/app-service-api-dotnet-get-started/settingsnext.png)
    
    接下來是 [預覽]  索引標籤 (如下所示)，它能讓您有機會查看哪些檔案即將從您的專案複製到 API 應用程式。 當您將專案部署至您先前已部署至的 API 應用程式時，只會複製已變更的檔案。 如果您想要查看即將複製的項目清單，可以按一下 [開始預覽]  按鈕。
15. 按一下 [發行] 。
    
    ![在 [發佈 Web] 的 [預覽] 索引標籤中按一下 [發佈]](./media/app-service-api-dotnet-get-started/clickpublish.png)
    
    Visual Studio 會將 ToDoListDataAPI 專案部署到新的 API 應用程式。 [輸出]  視窗會記錄成功的部署，而在開啟至 API 應用程式 URL 的瀏覽器視窗會出現 [已成功建立] 頁面。
    
    ![成功部署的輸出視窗](./media/app-service-api-dotnet-get-started/deploymentoutput.png)
    
    ![新的 API 應用程式已成功建立頁面](./media/app-service-api-dotnet-get-started/appcreated.png)
16. 在瀏覽器的位址列中將 "swagger" 新增至 URL，然後按 Enter 鍵 (URL 是 `http://{apiappname}.azurewebsites.net/swagger`。)
    
    瀏覽器會顯示您稍早看到的相同 Swagger UI，但該 UI 現在在雲端執行。 試用 Get 方法，然後您會發現您回到預設的 2 個待辦事項。 您稍早所做的變更已儲存在本機電腦的記憶體中。
17. 開啟 [Azure 入口網站](https://portal.azure.com/)。
    
    Azure 入口網站是用來管理 Azure 資源 (例如 API 應用程式) 的 Web 介面。
18. 按一下 [其他服務] > [應用程式服務]。
    
    ![瀏覽應用程式服務](./media/app-service-api-dotnet-get-started/browseas.png)
19. 在 [應用程式服務] 刀鋒視窗中，尋找並按一下新的 API 應用程式。 (在 Azure 入口網站中，在右側開啟的視窗稱為*刀鋒視窗*。)
    
    ![[應用程式服務] 刀鋒視窗](./media/app-service-api-dotnet-get-started/choosenewapiappinportal.png)
    
    有兩個刀鋒視窗會開啟。 其中一個包含 API 應用程式的概觀，另一個包含您可以檢視和變更的一長串設定。
20. 在 [設定] 刀鋒視窗中，尋找 [API] 區段並按一下 [API 定義]。
    
    ![[設定] 刀鋒視窗中的 API 定義](./media/app-service-api-dotnet-get-started/apidefinsettings.png)
    
    [API 定義]  刀鋒視窗可讓您指定會以 JSON 格式傳回 Swagger 2.0 中繼資料的 URL。 當 Visual Studio 建立 API 應用程式時，它會將 API 定義 URL 設定為您稍早所見的 Swashbuckle 產生的中繼資料預設值，也就是 API 應用程式的基底 URL 加上 `/swagger/docs/v1`。
    
    ![API 定義 URL](./media/app-service-api-dotnet-get-started/apidefurl.png)
    
    當您選取要對其產生用戶端程式碼的 API 應用程式時，Visual Studio 會從這個 URL 擷取中繼資料。

## <a id="codegen"></a> 產生資料層的用戶端程式碼
將 Swagger 整合到 Azure API 應用程式的優點之一，就是自動產生程式碼。 產生的用戶端類別讓您能更容易地撰寫會呼叫 API 應用程式的程式碼。

ToDoListAPI 專案已有產生的用戶端程式碼，但在下列步驟中，您要先將其刪除再予以重新產生，以了解如何產生程式碼。

1. 在 Visual Studio [方案總管] 的 ToDoListAPI 專案中，刪除 [ToDoListDataAPI] 資料夾。 **注意︰請只刪除資料夾，而非 ToDoListDataAPI 專案。**
   
    ![刪除產生的用戶端程式碼](./media/app-service-api-dotnet-get-started/deletecodegen.png)
   
    此資料夾先前已透過您即將進行的程式碼產生程序加以建立。
2. 以滑鼠右鍵按一下 ToDoListAPI 專案，然後按一下 [新增] > [REST API 用戶端]。
   
    ![在 Visual Studio 中新增 REST API 用戶端](./media/app-service-api-dotnet-get-started/codegenmenu.png)
3. 在 [新增 REST API 用戶端] 對話方塊中，依序按一下 [Swagger URL] 和 [選取 Azure 資產]。
   
    ![選取 Azure 資產](./media/app-service-api-dotnet-get-started/codegenbrowse.png)
4. 在 [App Service] 對話方塊中，展開您在本教學課程中使用的資源群組，選取您的 API 應用程式，然後按一下 [確定]。
   
    ![選取用於產生程式碼的 API 應用程式](./media/app-service-api-dotnet-get-started/codegenselect.png)
   
    請注意，當您返回 [加入 REST API 用戶端]  對話方塊時，文字方塊中已填入您稍早在入口網站中看到的 API 定義 URL 值。
   
    ![API 定義 URL](./media/app-service-api-dotnet-get-started/codegenurlplugged.png)
   
   > [!TIP]
   > 取得用於產生程式碼之中繼資料的令一個方法是直接輸入 URL，而不需透過瀏覽對話方塊。 或者，如果您要在部署至 Azure 之前產生用戶端程式碼，您可以在本機執行 Web API 專案、移至可提供 Swagger JSON 檔案的 URL、儲存檔案，然後使用 [選取現有的 Swagger 中繼資料檔案]  選項。
   > 
   > 
5. 在 [新增 REST API 用戶端] 對話方塊中，按一下 [確定]。
   
    Visual Studio 會建立以 API 應用程式命名的資料夾，並且產生用戶端類別。
   
    ![所產生用戶端的程式碼檔案](./media/app-service-api-dotnet-get-started/codegenfiles.png)
6. 在 ToDoListAPI 專案中，開啟 *Controllers\ToDoListController.cs*，查看使用所產生的用戶端呼叫 API 的程式碼 (第 40 行)。
   
    下列程式碼片段示範此程式碼如何具現化用戶端物件和呼叫 Get 方法。
   
        private static ToDoListDataAPI NewDataAPIClient()
        {
            var client = new ToDoListDataAPI(new Uri(ConfigurationManager.AppSettings["toDoListDataAPIURL"]));
            return client;
        }
   
        public async Task<IEnumerable<ToDoItem>> Get()
        {
            using (var client = NewDataAPIClient())
            {
                var results = await client.ToDoList.GetByOwnerAsync(owner);
                return results.Select(m => new ToDoItem
                {
                    Description = m.Description,
                    ID = (int)m.ID,
                    Owner = m.Owner
                });
            }
        }
   
    建構函式參數會從 `toDoListDataAPIURL` 應用程式設定取得端點 URL。 在 Web.config 檔案中，該值設為 API 專案的本機 IIS Express URL，以便讓您在本機執行應用程式。 如果您省略建構函式參數，預設端點會是您產生程式碼的 URL。
7. 將會根據您的 API 應用程式名稱，以不同的名稱產生您的用戶端類別；在 *Controllers\ToDoListController.cs* 中變更此程式碼，讓類型名稱符合您的專案中產生的內容。 例如，如果您將 API 應用程式命名為 ToDoListDataAPI071316，則會將此程式碼︰
   
        private static ToDoListDataAPI NewDataAPIClient()
        {
            var client = new ToDoListDataAPI(new Uri(ConfigurationManager.AppSettings["toDoListDataAPIURL"]));

變更為以下程式碼：

        private static ToDoListDataAPI071316 NewDataAPIClient()
        {
            var client = new ToDoListDataAPI071316(new Uri(ConfigurationManager.AppSettings["toDoListDataAPIURL"]));


## <a name="create-an-api-app-to-host-the-middle-tier"></a>建立 API 應用程式來裝載中間層
稍早您 [已建立資料層 API 應用程式並於其中部署程式碼](#createapiapp)。  現在，您必須遵循相同的程序來建立中介層 API 應用程式。

1. 在 [方案總管] 中，以滑鼠右鍵按一下中介層 ToDoListAPI 專案 (而非資料層 ToDoListDataAPI)，然後按一下 [發佈]。
   
    ![在 Visual Studio 中按一下 [發佈]](./media/app-service-api-dotnet-get-started/pubinmenu2.png)
2. 在 [發佈 Web] 精靈的 [設定檔] 索引標籤中，按一下 [Microsoft Azure App Service]。
3. 在 [App Service] 對話方塊中，按一下 [新增]。
4. 在 [建立 App Service] 對話方塊的 [主控] 索引標籤中，接受預設的 [API 應用程式名稱] 或輸入 *azurewebsites.net* 網域中唯一的名稱。
5. 選擇您所使用的 Azure [訂用帳戶]  。
6. 在 [資源群組]  下拉式清單中，選擇您稍早建立的資源群組。
7. 在 [App Service 方案]  下拉式清單中，選擇您稍早建立的同一個方案。 它將會預設為該值。
8. 按一下 [建立] 。
   
    Visual Studio 會建立 API 應用程式、建立其發佈設定檔，並顯示 [發佈 Web] 精靈的 [連接] 步驟。
9. 在 [發佈 Web] 精靈的 [連接] 步驟中，按一下 [發佈]。
   
   Visual Studio 會將 ToDoListAPI 專案部署到新的 API 應用程式，並將瀏覽器開啟至 API 應用程式的 URL。 [已成功建立] 頁面隨即出現。

## <a name="configure-the-middle-tier-to-call-the-data-tier"></a>設定用來呼叫資料層的中介層
如果您現在呼叫中介層 API 應用程式，它會使用仍在 Web.config 檔案中的 localhost URL 嘗試呼叫資料層。 在這一節中，您會將資料層 API 應用程式 URL 輸入到中介層 API 應用程式中的環境設定。 當中介層 API 應用程式中的程式碼擷取資料層 URL 設定時，環境設定會覆寫 Web.config 檔案中的內容。

1. 移至 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至您建立以裝載 TodoListAPI (中介層) 專案之 API 應用程式的 [API 應用程式] 刀鋒視窗。
2. 在 API 應用程式的 [設定] 刀鋒視窗中，按一下 [應用程式設定]。
3. 在 API 應用程式的 [應用程式設定] 刀鋒視窗中，向下捲動至 [應用程式設定] 區段，並新增下列應用程式金鑰和值。 此值會是您在本教學課程所發佈的第一個 API 應用程式的 URL。
   
   | **Key** | toDoListDataAPIURL |
   | --- | --- |
   | **值** |https://{您的資料層 API 應用程式名稱}.azurewebsites.net |
   | **範例** |https://todolistdataapi.azurewebsites.net |
4. 按一下 [儲存] 。
   
    ![按一下 [應用程式設定] 的 [儲存]](./media/app-service-api-dotnet-get-started/asinportal.png)
   
    在 Azure 中執行程式碼時，這個值現在會覆寫 Web.config 檔案中的 localhost URL。

## <a name="test"></a>測試
1. 在瀏覽器視窗中，瀏覽至您剛才為 ToDoListAPI 建立的新中介層 API 應用程式的 URL。 按一下 API 應用程式在入口網站中的主要刀鋒視窗內的 URL，即可前往該位置。
2. 在瀏覽器的位址列中將 "swagger" 新增至 URL，然後按 Enter 鍵 (URL 是 `http://{apiappname}.azurewebsites.net/swagger`。)
   
    瀏覽器會顯示您稍早在 ToDoListDataAPI 所看到的相同 Swagger UI，但現在 `owner` 不是 Get 作業的必要欄位，因為中介層 API 應用程式會為您將該值傳送到資料層 API 應用程式。 (當您進行驗證教學課程時，中介層會傳送 `owner` 參數的實際使用者識別碼；它現在是硬式編碼星號。)
3. 試用 Get 方法和其他方法來確認中介層 API 應用程式會成功呼叫資料層 API 應用程式。
   
    ![Swagger UI Get 方法](./media/app-service-api-dotnet-get-started/midtierget.png)

## <a name="troubleshooting"></a>疑難排解
如果您在瀏覽本教學課程時遇到問題，以下是一些疑難排解的相關說明︰

* 確定您使用最新版本的 [Azure SDK for .NET](http://go.microsoft.com/fwlink/?linkid=518003)。
* 專案名稱的其中兩個類似 (ToDoListAPI、ToDoListDataAPI)。 當您使用專案時，如果專案名稱不同於指示中所述，請確定您所開啟的是正確的專案。
* 如果您位於公司網路內，並嘗試透過防火牆部署至 Azure App Service，請確定連接埠 443 和 8172 已針對 Web Deploy 開啟。 如果您無法開啟這些連接埠，則可以使用其他部署方法。  請參閱 [將您的應用程式部署至 Azure App Service](../app-service-web/web-sites-deploy.md)。
* 「路由名稱必須是唯一的」錯誤 -- 如果您不小心將錯誤的專案部署到 API 應用程式，稍後再將正確的專案部署到其中，您可能會收到這些錯誤。 若要修正此問題，請將正確的專案重新部署至 API 應用程式，然後在 [發佈 Web] 精靈的 [設定] 索引標籤上選取 [在目的地移除多餘的檔案]。

在 Azure App Service 中執行 ASP.NET API 應用程式之後，建議您深入了解可簡化疑難排解步驟的 Visual Studio 功能。 如需記錄和遠端偵錯等功能的相關資訊，請參閱[在 Visual Studio 中針對 Azure App Service 應用程式進行疑難排解](../app-service-web/web-sites-dotnet-troubleshoot-visual-studio.md)。

## <a name="next-steps"></a>後續步驟
您已了解如何將現有 Web API 專案部署到 API 應用程式、產生 API 應用程式的用戶端程式碼，以及從 .NET 用戶端取用 API 應用程式。 此系列中的下一個教學課程示範如何 [使用 CORS 從 JavaScript 用戶端取用 API 應用程式](app-service-api-cors-consume-javascript.md)。

如需產生用戶端程式碼的詳細資訊，請參閱 GitHub.com 上的 [Azure/AutoRest](https://github.com/azure/autorest) 儲存機制。 如需使用所產生用戶端的相關問題協助，請開啟 [AutoRest 儲存機制中的問題](https://github.com/azure/autorest/issues)。

如果您想要從頭開始建立新的 API 應用程式專案，請使用 **Azure API 應用程式** 範本。

![Visual Studio 中的 API 應用程式範本](./media/app-service-api-dotnet-get-started/apiapptemplate.png)

[Azure API 應用程式] 專案範本等同於選擇 [空白] ASP.NET 4.5.2 範本、按一下核取方塊以加入 Web API 支援，然後安裝 Swashbuckle NuGet 封裝。 此外，範本會加入為了避免建立重複的 Swagger 作業識別碼而設計的某些 Swashbuckle 組態程式碼。 在建立 API 應用程式專案之後，您可以使用在本教學課程中看到的相同方式，將它部署到 API 應用程式。


