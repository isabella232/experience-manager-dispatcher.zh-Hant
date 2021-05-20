---
title: 將 SSL 與 Dispatcher 搭配使用
seo-title: 將 SSL 與 Dispatcher 搭配使用
description: 了解如何設定Dispatcher以使用SSL連線與AEM通訊。
seo-description: 了解如何設定Dispatcher以使用SSL連線與AEM通訊。
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
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '1375'
ht-degree: 1%

---

# 將 SSL 與 Dispatcher 搭配使用 {#using-ssl-with-dispatcher}

在Dispatcher和轉譯電腦之間使用SSL連線：

* [單向SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [相互SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>與SSL憑證相關的作業會系結至協力廠商產品。 Adobe白金級維護與支援合同未涵蓋這些條款。

## 當Dispatcher連線至AEM {#use-ssl-when-dispatcher-connects-to-aem}時使用SSL

設定Dispatcher以使用SSL連線與AEM或CQ轉譯例項通訊。

設定Dispatcher之前，請先將AEM或CQ設為使用SSL:

* AEM 6.2:[透過SSL啟用HTTP](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[透過SSL啟用HTTP](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 舊版AEM:請參閱[此頁面](https://helpx.adobe.com/tw/experience-manager/aem-previous-versions.html)。

### SSL相關請求標題{#ssl-related-request-headers}

當Dispatcher收到HTTPS請求時，Dispatcher會在後續請求中納入下列標題，以便傳送至AEM或CQ:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

透過`mod_ssl`的Apache-2.4要求包含與下列範例類似的標題：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 將Dispatcher設定為使用SSL {#configuring-dispatcher-to-use-ssl}

若要設定Dispatcher以透過AEM或SSL進行連線，您的[dispatcher.any](dispatcher-configuration.md)檔案需要下列屬性：

* 處理HTTPS要求的虛擬主機。
* 虛擬主機的`renders`部分包含一個項，用於標識使用HTTPS的CQ或AEM實例的主機名和埠。
* `renders`項目包含值`1`的名為`secure`的屬性。

注意：視需要建立另一個虛擬主機以處理HTTP要求。

下列範例dispatcher.any檔案顯示使用HTTPS連線至主機`localhost`和連接埠`8443`上執行之CQ例項的屬性值：

```
/farms
{
   /secure
   { 
      /virtualhosts
      {
         # select this farm for all incoming HTTPS requestss
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
         "https://*"
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

## 在Dispatcher與AEM之間設定相互SSL {#configuring-mutual-ssl-between-dispatcher-and-aem}

設定Dispatcher與轉譯電腦(通常為AEM或CQ發佈例項)之間的連線，以使用Mutual SSL:

* Dispatcher透過SSL連線至轉譯例項。
* 呈現例項會驗證Dispatcher憑證的有效性。
* Dispatcher會驗證轉譯例項憑證的CA是否受信任。
* （選用）Dispatcher會驗證轉譯例項的憑證是否符合轉譯例項的伺服器位址。

要配置相互SSL，您需要由受信任的證書頒發機構(CA)簽名的證書。 自簽名證書不足。 您可以擔任CA，或使用第三方CA的服務來簽署您的憑證。 若要設定相互SSL，您需要下列項目：

* 轉譯例項和Dispatcher的已簽署憑證
* CA證書（如果您充當CA）
* OpenSSL程式庫，用於產生CA、憑證和憑證要求。

執行下列步驟以設定互相SSL:

1. [](dispatcher-install.md) 為您的平台安裝最新版的Dispatcher。使用支援SSL的Dispatcher二進位檔(SSL位於檔案名稱中，例如dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar)。
1. [為Dispatcher和轉譯例項建](dispatcher-ssl.md#main-pars-title-3) 立或取得CA簽署的憑證。
1. [建立包含轉譯憑證](dispatcher-ssl.md#main-pars-title-6) 的金鑰存放區，並設定轉譯的HTTP服務以使用它。
1. [設定互相SSL的Dispatcher網](dispatcher-ssl.md#main-pars-title-4) 頁伺服器模組。

### 建立或獲取CA簽名證書{#creating-or-obtaining-ca-signed-certificates}

建立或取得CA簽署的憑證，以驗證發佈執行個體和Dispatcher。

#### 建立CA {#creating-your-ca}

如果您充當CA，請使用[OpenSSL](https://www.openssl.org/)建立簽署伺服器和客戶端證書的證書頒發機構。 （您必須安裝OpenSSL程式庫。） 如果您使用第三方CA，請勿執行此過程。

1. 開啟終端機，並將目前目錄變更為包含CA.sh檔案的目錄，例如`/usr/local/ssl/misc`。
1. 要建立CA，請輸入以下命令，然後在提示時提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >openssl.cnf檔案中的數個屬性可控制CA.sh指令碼的行為。 在建立CA之前，您應根據需要修改此檔案。

#### 建立證書{#creating-the-certificates}

使用OpenSSL建立要傳送至協力廠商CA或與您的CA簽署的憑證要求。

當您建立憑證時，OpenSSL會使用Common Name屬性來識別憑證持有者。 對於呈現實例的證書，如果要將Dispatcher配置為僅在證書與Publish實例的主機名匹配時才接受證書，請使用實例電腦的主機名作為「公用名」。 （請參閱[DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11)屬性。）

1. 開啟終端機，並將目前目錄變更為包含OpenSSL程式庫之CH.sh檔案的目錄。
1. 輸入以下命令，並在出現提示時提供值。 如有需要，請使用發佈執行個體的主機名稱作為通用名稱。 主機名是呈現的IP地址的DNS可解析的名稱：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用第三方CA，請將newreq.pem檔案發送到CA以簽名。 如果您擔任CA，請繼續執行步驟3。

1. 輸入以下命令以使用CA的證書籤名證書：

   ```shell
   ./CA.sh -sign
   ```

   在包含CA管理檔案的目錄中建立了兩個名為newcert.pem和newkey.pem的檔案。 這些是轉譯電腦的公開憑證和私密金鑰。

1. 將newcert.pem重新命名為rendercert.pem，並將newkey.pem重新命名為renderkey.pem。
1. 重複步驟2和3，為Dispatcher模組建立新憑證和新的公開金鑰。 請確定您使用Dispatcher例項專屬的通用名稱。
1. 將newcert.pem重新命名為dispcert.pem，並將newkey.pem重新命名為dispkey.pem。

### 在呈現電腦{#configuring-ssl-on-the-render-computer}上配置SSL

使用rendercert.pem和renderkey.pem檔案，在轉譯例項上設定SSL。

#### 將呈現證書轉換為JKS格式{#converting-the-render-certificate-to-jks-format}

使用以下命令將渲染證書（PEM檔案）轉換為PKCS#12檔案。 還包括簽署呈現證書的CA的證書：

1. 在終端窗口中，將當前目錄更改為呈現證書和私鑰的位置。
1. 輸入以下命令，將渲染證書（PEM檔案）轉換為PKCS#12檔案。 還包括簽署呈現證書的CA的證書：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 輸入以下命令以將PKCS#12檔案轉換為Java KeyStore(JKS)格式：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java金鑰存放區是使用預設別名建立。 視需要變更別名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 將CA證書添加到Render的Truststore {#adding-the-ca-cert-to-the-render-s-truststore}

如果您充當CA，請將CA憑證匯入金鑰存放區。 然後，配置運行呈現實例的JVM以信任密鑰庫。

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

1. 使用以下命令將證書導入密鑰庫：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 要配置運行呈現實例的JVM以信任密鑰庫，請使用以下系統屬性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果使用crx-quickstart/bin/quickstart指令碼啟動發佈實例，則可以修改CQ_JVM_OPTS屬性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 配置渲染實例{#configuring-the-render-instance}

使用呈現憑證，並搭配&#x200B;*發佈執行個體*&#x200B;區段上的啟用SSL中的指示，將呈現執行個體的HTTP服務設定為使用SSL:

* AEM 6.2:[透過SSL啟用HTTP](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[透過SSL啟用HTTP](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 舊版AEM:請參閱[此頁面。](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### 為Dispatcher模組{#configuring-ssl-for-the-dispatcher-module}配置SSL

若要設定Dispatcher以使用相互SSL，請準備Dispatcher憑證，然後設定Web伺服器模組。

### 建立Unified Dispatcher憑證{#creating-a-unified-dispatcher-certificate}

將Dispatcher憑證和未加密的私密金鑰合併為單一PEM檔案。 使用文本編輯器或`cat`命令建立與以下示例類似的檔案：

1. 開啟終端機，並將目前目錄變更為dispkey.pem檔案的位置。
1. 要解密私鑰，請輸入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文本編輯器或`cat`命令將未加密的私鑰和證書合併到與以下示例類似的單個檔案中：

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

### 指定要用於Dispatcher {#specifying-the-certificate-to-use-for-dispatcher}的憑證

將下列屬性新增至[Dispatcher模組設定](dispatcher-install.md#main-pars-55-35-1022)（在`httpd.conf`中）:

* `DispatcherCertificateFile`:Dispatcher統一憑證檔案的路徑，包含公開憑證和未加密的私密金鑰。當SSL伺服器要求Dispatcher用戶端憑證時，會使用此檔案。
* `DispatcherCACertificateFile`:CA證書檔案的路徑，如果SSL伺服器呈現的CA不受根授權機構信任，則使用此路徑。
* `DispatcherCheckPeerCN`:是否啟用( `On`)或禁用( `Off`)遠程伺服器證書的主機名檢查。

下列程式碼為設定範例：

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
