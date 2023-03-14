---
title: 搭配 Dispatcher 使用 SSL
seo-title: Using SSL with Dispatcher
description: 了解如何設定 Dispatcher 以便使用 SSL 連線與 AEM 通訊。
seo-description: Learn how to configure Dispatcher to communicate with AEM using SSL connections.
uuid: 1a8f448c-d3d8-4798-a5cb-9579171171ed
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 771cfd85-6c26-4ff2-a3fe-dff8d8f7920b
index: y
internal: n
snippet: y
exl-id: ec378409-ddb7-4917-981d-dbf2198aca98
source-git-commit: e87af532ee3268f0a45679e20031c3febc02de58
workflow-type: tm+mt
source-wordcount: '1355'
ht-degree: 69%

---

# 搭配 Dispatcher 使用 SSL {#using-ssl-with-dispatcher}

在 Dispatcher 與轉譯器電腦之間使用 SSL 連線：

* [單向 SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [雙向 SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>與SSL憑證相關的作業會系結至協力廠商產品。 Adobe 白金級維護和支援合約中不涵蓋這些作業。

## 在 Dispatcher 連線到 AEM 時使用 SSL {#use-ssl-when-dispatcher-connects-to-aem}

設定 Dispatcher 以便使用 SSL 連線與 AEM 或 CQ 轉譯器執行個體通訊。

在設定 Dispatcher 之前，請設定 AEM 或 CQ 使用 SSL：

* AEM 6.2：[啟用透過 SSL 的 HTTP](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)
* AEM 6.1：[啟用透過 SSL 的 HTTP](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)
* AEM 舊版：請參閱[此頁面](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)。

### 與 SSL 相關的請求標頭 {#ssl-related-request-headers}

當Dispatcher收到HTTPS請求時，Dispatcher會在後續要求中納入下列標題，以便傳送至AEM或CQ:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

透過帶有 `mod_ssl` 的 Apache-2.4 發送的請求包含與以下範例類似的標頭：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 設定 Dispatcher 使用 SSL {#configuring-dispatcher-to-use-ssl}

若要設定 Dispatcher 來透過 SSL 與 AEM 或 CQ 連線，您的 [dispatcher.any](dispatcher-configuration.md) 檔案需要以下屬性：

* 處理 HTTPS 請求的虛擬主機。
* 虛擬主機的 `renders` 區段包含一個項目，此項目會識別使用 HTTPS 的 CQ 或 AEM 執行個體的主機名稱及連接埠。
* `renders` 項目包含名為 `secure` 的屬性，其值為 `1`。

注意：如有必要，請建立另一個虛擬主機以處理HTTP要求。

下列範例 `dispatcher.any` 檔案顯示使用HTTPS連線至主機上執行之CQ例項的屬性值 `localhost` 和埠 `8443`:

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requests
         "https://*"
      }
      /renders
      {
      /0001
         {
            # hostname or IP of the render
            /hostname "localhost"
            # port of the render
            /port "8443"
            # connect via HTTPS
            /secure "1"
         }
      }
     # the rest of the properties are omitted
   }

   /non-secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTP requests
         "http://*"
      }
      /renders
      {
         /0001
      {
         # hostname or IP of the render
         /hostname "localhost"
         # port of the render
         /port "4503"
      }
   }
    # the rest of the properties are omitted
}
```

## 設定 Dispatcher 與 AEM 之間的雙向 SSL {#configuring-mutual-ssl-between-dispatcher-and-aem}

若要使用Mutual SSL，請設定Dispatcher與轉譯電腦(通常為AEM或CQ發佈例項)之間的連線：

* Dispatcher 會透過 SSL 連線到轉譯器執行個體。
* 轉譯器執行個體會驗證 Dispatcher 的憑證是否有效。
* Dispatcher 會驗證轉譯器執行個體憑證的 CA 是否值得信任。
* (選擇性) Dispatcher 會驗證轉譯器執行個體的憑證是否符合轉譯器執行個體的伺服器位址。

若要設定雙向 SSL，您需要由受信任的憑證授權單位 (CA) 簽署的憑證。 自我簽署憑證是不夠的。 您可以充當 CA 或使用第三方 CA 的服務來簽署您的憑證。 若要設定雙向 SSL，您需要以下項目：

* 轉譯器執行個體和 Dispatcher 適用的已簽署憑證
* CA 憑證 (如果您充當 CA)
* 用來產生 CA、憑證和憑證請求的 OpenSSL 程式庫。

要配置互用SSL，請執行以下步驟：

1. [安裝](dispatcher-install.md)您的平台適用的最新版 Dispatcher。 使用支援 SSL 的 Dispatcher 二進位檔案 (SSL 位在檔案名稱中，例如 dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar)。
1. [建立或取得 CA 簽署的憑證](dispatcher-ssl.md#main-pars-title-3)，該憑證適用於 Dispatcher 和轉譯器執行個體。
1. [建立包含呈現證書的密鑰庫](dispatcher-ssl.md#main-pars-title-6) 並設定轉譯的HTTP服務。
1. [設定 Dispatcher 網頁伺服器模組](dispatcher-ssl.md#main-pars-title-4)以供雙向 SSL 使用。

### 建立或取得 CA 簽署的憑證 {#creating-or-obtaining-ca-signed-certificates}

建立或取得 CA 簽署的憑證，該憑證用來驗證發佈執行個體和 Dispatcher。

#### 建立您的 CA {#creating-your-ca}

如果您充當 CA，請使用 [OpenSSL](https://www.openssl.org/) 建立可簽署伺服器和用戶端憑證的憑證授權單位。 (您必須已安裝 OpenSSL 程式庫。) 如果您使用第三方 CA，請勿執行此程序。

1. 開啟終端機，並將目前目錄變更為包含 `CA.sh` 檔案，例如 `/usr/local/ssl/misc`.
1. 要建立CA，請輸入以下命令，然後在出現提示時提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >中的數個屬性 `openssl.cnf` 檔案控制CA.sh指令碼的行為。 在建立CA之前，根據需要編輯此檔案。

#### 建立憑證 {#creating-the-certificates}

使用 OpenSSL 建立要傳送給第三方 CA 或透過您的 CA 簽署的憑證請求。

當您建立憑證時，OpenSSL 會使用一般名稱屬性來識別憑證持有者。 對於呈現實例的證書，如果要將Dispatcher配置為接受證書，並且僅當其與Publish實例的主機名匹配時，才使用實例電腦的主機名作為公用名。 (請參閱 [DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11) 屬性。)

1. 開啟終端機，並將目前目錄切換到包含您的 OpenSSL 程式庫的 CH.sh 檔案的目錄。
1. 輸入以下命令，然後在收到提示時提供值。 如有必要，請使用發佈執行個體的主機名稱作為通用名稱。 主機名稱是轉譯器的 IP 位址的 DNS 可解析名稱：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用第三方 CA，請傳送 newreq.pem 檔案給 CA 進行簽署。 如果您充當 CA，請繼續前往步驟 3。

1. 要使用CA的證書籤名證書，請輸入以下命令：

   ```shell
   ./CA.sh -sign
   ```

   兩個檔案已命名 `newcert.pem` 和 `newkey.pem` 在包含CA管理檔案的目錄中建立。 這兩個檔案分別是轉譯電腦的公開憑證和私密金鑰。

1. 重新命名 `newcert.pem` to `rendercert.pem`，並重新命名 `newkey.pem` to `renderkey.pem`.
1. 重複步驟2和3，為Dispatcher模組建立憑證和公開金鑰。 務必使用 Dispatcher 執行個體專屬的一般名稱。
1. 重新命名 `newcert.pem` to `dispcert.pem`，並重新命名 `newkey.pem` to `dispkey.pem`.

### 在轉譯器電腦上設定 SSL {#configuring-ssl-on-the-render-computer}

在轉譯例項上使用 `rendercert.pem` 和 `renderkey.pem` 檔案。

#### 將轉譯憑證轉換為JKS(Java™ KeyStore)格式 {#converting-the-render-certificate-to-jks-format}

使用以下命令將渲染證書（PEM檔案）轉換為PKCS#12檔案。 也請包含簽署了轉譯器憑證的 CA 的憑證：

1. 在終端機視窗中，將目前目錄切換到轉譯器憑證和私密金鑰的位置。
1. 要將渲染證書（PEM檔案）轉換為PKCS#12檔案，請輸入以下命令。 也請包含簽署了轉譯器憑證的 CA 的憑證：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 要將PKCS#12檔案轉換為Java™ KeyStore(JKS)格式，請輸入以下命令：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java™金鑰存放區是使用預設別名建立。 如有需要，請變更別名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 將 CA 憑證新增到轉譯器的信任存放區 {#adding-the-ca-cert-to-the-render-s-truststore}

如果您充當 CA，請將您的 CA 憑證匯入金鑰儲存區。 然後，將執行轉譯器執行個體的 JVM 設定為信任該金鑰儲存區。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文本編輯器開啟cacert.pem檔案，並刪除以下行前面的所有文本：

   `-----BEGIN CERTIFICATE-----`

1. 使用以下命令，將憑證匯入金鑰儲存區：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 若要將執行轉譯器執行個體的 JVM 設定為信任該金鑰儲存區，請使用以下系統屬性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果您使用 crx-quickstart/bin/quickstart 指令碼啟動您的發佈執行個體，您可以修改 CQ_JVM_OPTS 屬性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 設定轉譯器執行個體 {#configuring-the-render-instance}

若要設定轉譯例項的HTTP服務以使用SSL，請使用轉譯憑證，並附上 *在發佈執行個體上啟用SSL* 小節：

* AEM 6.2：[啟用透過 SSL 的 HTTP](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)
* AEM 6.1：[啟用透過 SSL 的 HTTP](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)
* AEM 舊版：請參閱[此頁面。](https://experienceleague.adobe.com/docs/experience-manager-release-information/aem-release-updates/previous-updates/aem-previous-versions.html?lang=zh-Hant)

### 為 Dispatcher 模組設定 SSL {#configuring-ssl-for-the-dispatcher-module}

若要設定 Dispatcher 使用雙向 SSL，請準備 Dispatcher 憑證，然後設定網頁伺服器模組。

### 建立統一的 Dispatcher 憑證 {#creating-a-unified-dispatcher-certificate}

將Dispatcher憑證和未加密的私密金鑰合併為單一PEM檔案。 使用文字編輯器或 `cat` 命令建立類似於以下範例的檔案：

1. 開啟終端機，並將目前目錄切換到 dispkey.pem 檔案的位置。
1. 若要將私密金鑰解密，請輸入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文字編輯器或 `cat` 命令，將未加密的私密金鑰和憑證合併到類似於以下範例的單一檔案中：

   ```xml
   -----BEGIN RSA PRIVATE KEY-----
   MIICxjBABgkqhkiG9w0B...
   ...M2HWhDn5ywJsX
   -----END RSA PRIVATE KEY-----
   -----BEGIN CERTIFICATE-----
   MIIC3TCCAk...
   ...roZAs=
   -----END CERTIFICATE-----
   ```

### 指定要用於 Dispatcher 的憑證 {#specifying-the-certificate-to-use-for-dispatcher}

將以下屬性新增到 [Dispatcher 模組設定](dispatcher-install.md#main-pars-55-35-1022) (在 `httpd.conf` 中)：

* `DispatcherCertificateFile`：指向 Dispatcher 統一憑證檔案的路徑，該檔案包含公開憑證和未加密的私密金鑰。 當 SSL 伺服器請求 Dispatcher 用戶端憑證時，就會使用這個檔案。
* `DispatcherCACertificateFile`：指向 CA 憑證檔案的路徑，在 SSL 伺服器提供的 CA 未受到根授權單位信任時使用。
* `DispatcherCheckPeerCN`：是要啟用 (`On`) 還是停用 (`Off`) 對遠端伺服器憑證的主機名稱檢查。

以下程式碼為設定範例：

```xml
<IfModule disp_apache2.c>
  DispatcherConfig conf/dispatcher.any
  DispatcherLog    logs/dispatcher.log
  DispatcherLogLevel 3
  DispatcherNoServerHeader 0
  DispatcherDeclineRoot 0
  DispatcherUseProcessedURL 0
  DispatcherPassError 0
  DispatcherCertificateFile disp_unified.pem
  DispatcherCACertificateFile cacert.pem
  DispatcherCheckPeerCN On
</IfModule>
```
