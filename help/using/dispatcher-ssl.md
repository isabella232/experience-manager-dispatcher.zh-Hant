---
title: 將 SSL 與 Dispatcher 搭配使用
seo-title: 將 SSL 與 Dispatcher 搭配使用
description: 瞭解如何設定Dispatcher以使用SSL連線與AEM通訊。
seo-description: 瞭解如何設定Dispatcher以使用SSL連線與AEM通訊。
uuid: 1a8f448c-d3d8-4798-a5cb-9579171171ed
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 771cfd85-6c26-4ff2-a3fe-dff8d8f7920b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f9fb0e94dbd1c67bf87463570e8b5eddaca11bf3
workflow-type: tm+mt
source-wordcount: '1375'
ht-degree: 0%

---


# 將 SSL 與 Dispatcher 搭配使用 {#using-ssl-with-dispatcher}

在Dispatcher和Render電腦之間使用SSL連接：

* [單向SSL](#use-ssl-when-dispatcher-connects-to-aem)
* [相互SSL](#configuring-mutual-ssl-between-dispatcher-and-aem)

>[!NOTE]
>
>與SSL憑證相關的作業會系結至第三方產品。 Adobe白金級維護與支援合約未涵蓋這些條款。

## Dispatcher Connects to AEM {#use-ssl-when-dispatcher-connects-to-aem}時使用SSL

設定Dispatcher以使用SSL連線與AEM或CQ演算例項通訊。

在設定Dispatcher之前，請先設定AEM或CQ以使用SSL:

* AEM 6.2:[啟用HTTP Over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[啟用HTTP Over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 舊版AEM:請參閱[本頁](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)。

### 與SSL相關的請求標頭{#ssl-related-request-headers}

當Dispatcher收到HTTPS請求時，Dispatcher會在後續請求中包含下列標頭，並將其傳送至AEM或CQ:

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

透過Apache-2.4（含`mod_ssl`）提出的請求包含類似下列範例的標題：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 配置Dispatcher以使用SSL {#configuring-dispatcher-to-use-ssl}

若要設定Dispatcher以透過SSL與AEM或CQ連線，您的[dispatcher.any](dispatcher-configuration.md)檔案需要下列屬性：

* 處理HTTPS請求的虛擬主機。
* 虛擬主機的`renders`區段包含一個項目，可識別使用HTTPS的CQ或AEM例項的主機名稱和連接埠。
* `renders`項目包含名為`secure`的值`1`的屬性。

注意：如果需要，請建立另一個虛擬主機以處理HTTP請求。

以下示例dispatcher.any檔案顯示了使用HTTPS連接到在主機`localhost`和埠`8443`上運行的CQ實例的屬性值：

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

## 在Dispatcher和AEM {#configuring-mutual-ssl-between-dispatcher-and-aem}之間配置互相SSL

設定Dispatcher與演算電腦（通常是AEM或CQ發佈例項）之間的連線，以使用Mutual SSL:

* Dispatcher通過SSL連接到渲染實例。
* 演算實例驗證Dispatcher證書的有效性。
* Dispatcher會驗證演算例項憑證的CA是否受信任。
* （可選）Dispatcher會驗證演算例項的憑證是否符合演算例項的伺服器位址。

要配置互用SSL，您需要由受信任證書頒發機構(CA)簽名的證書。 自簽證書不夠。 您可以擔任CA，或使用協力廠商CA的服務來簽署憑證。 若要設定相互SSL，您需要下列項目：

* 演算實例和Dispatcher的已簽名證書
* CA證書（如果您充當CA）
* OpenSSL程式庫，以產生CA、憑證和憑證要求。

請執行下列步驟來設定互用SSL:

1. [為](dispatcher-install.md) 您的平台安裝最新版本的Dispatcher。使用支援SSL的Dispatcher二進位檔案（SSL在檔案名中，如dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar）。
1. [為Dispatcher和render實例建立或](dispatcher-ssl.md#main-pars-title-3) 獲取CA簽名證書。
1. [建立包含轉譯憑證](dispatcher-ssl.md#main-pars-title-6) 的金鑰庫，並設定轉譯的HTTP服務以使用它。
1. [配置Dispatcher Web伺服器模](dispatcher-ssl.md#main-pars-title-4) 塊以實現相互SSL。

### 建立或獲取CA簽名證書{#creating-or-obtaining-ca-signed-certificates}

建立或取得CA簽署的憑證，以驗證發佈執行個體和Dispatcher。

#### 建立CA {#creating-your-ca}

如果您是CA，請使用[OpenSSL](https://www.openssl.org/)來建立簽署伺服器和用戶端憑證的憑證授權機構。 （您必須安裝OpenSSL程式庫。） 如果您使用第三方CA，請勿執行此程式。

1. 開啟終端，將當前目錄更改為包含CA.sh檔案的目錄，如`/usr/local/ssl/misc`。
1. 要建立CA，請輸入以下命令，然後在提示時提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >openssl.cnf檔案中的數個屬性可控制CA.sh指令碼的行為。 在建立CA之前，應根據需要修改此檔案。

#### 建立證書{#creating-the-certificates}

使用OpenSSL建立要傳送至協力廠商CA或與CA簽署的憑證要求。

當您建立憑證時，OpenSSL會使用「公用名稱」屬性來識別憑證持有人。 對於演算實例的證書，如果要將Dispatcher配置為只在證書與Publish實例的主機名匹配時才接受證書，請使用實例電腦的主機名作為通用名稱。 （請參閱[DispatcherCheckPeerCN](dispatcher-ssl.md#main-pars-title-11)屬性）。

1. 開啟終端機，並將目前的目錄變更為包含OpenSSL程式庫CH.sh檔案的目錄。
1. 輸入以下命令，並在出現提示時提供值。 如有需要，請使用發佈實例的主機名作為公用名。 主機名是DNS可解析的Render IP地址名稱：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用協力廠商CA，請將newreq.pem檔案傳送至CA進行簽署。 如果您是CA，請繼續步驟3。

1. 輸入以下命令，使用CA的證書籤名證書：

   ```shell
   ./CA.sh -sign
   ```

   在包含CA管理檔案的目錄中建立了兩個名為newcert.pem和newkey.pem的檔案。 這些是演算電腦的公開憑證和私密金鑰。

1. 將newcert.pem更名為rendercert.pem，並將newkey.pem更名為renderkey.pem。
1. 重複步驟2和3，為Dispatcher模組建立新證書和新公共密鑰。 確保使用特定於Dispatcher實例的公用名稱。
1. 將newcert.pem更名為dispcert.pem，並將newkey.pem更名為dispkey.pem。

### 在Render Computer {#configuring-ssl-on-the-render-computer}上配置SSL

使用rendercert.pem和renderkey.pem檔案，在render實例上設定SSL。

#### 將Render Certificate轉換為JKS格式{#converting-the-render-certificate-to-jks-format}

使用以下命令將渲染證書（即PEM檔案）轉換為PKCS#12檔案。 也包含簽署轉譯憑證的CA憑證：

1. 在終端窗口中，將當前目錄更改為渲染證書和私鑰的位置。
1. 輸入以下命令，將渲染證書（即PEM檔案）轉換為PKCS#12檔案。 也包含簽署轉譯憑證的CA憑證：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 輸入以下命令，將PKCS#12檔案轉換為Java KeyStore(JKS)格式：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java密鑰庫是使用預設別名建立的。 視需要變更別名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 將CA證書添加到Render的Truststore {#adding-the-ca-cert-to-the-render-s-truststore}

如果您是CA，請將CA憑證匯入密鑰庫。 然後，配置運行演算實例的JVM以信任密鑰庫。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文本編輯器開啟cacert.pem檔案並刪除以下行之前的所有文本：

   `-----BEGIN CERTIFICATE-----`

1. 使用以下命令將證書導入密鑰庫：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 要配置運行演算實例的JVM以信任密鑰庫，請使用以下系統屬性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果您使用crx-quickstart/bin/quickstart指令碼來啟動發佈實例，則可以修改CQ_JVM_OPTS屬性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 配置Render實例{#configuring-the-render-instance}

使用演算憑證及「發佈例項」區段中的「啟用SSL」指示，將演算例項的HTTP服務設定為使用SSL:**

* AEM 6.2:[啟用HTTP Over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM 6.1:[啟用HTTP Over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 舊版AEM:請參閱[本頁。](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### 為Dispatcher模組{#configuring-ssl-for-the-dispatcher-module}配置SSL

要配置Dispatcher以使用互相SSL，請準備Dispatcher證書，然後配置Web伺服器模組。

### 建立Unified Dispatcher證書{#creating-a-unified-dispatcher-certificate}

將調度器憑證和未加密的私密金鑰結合為單一PEM檔案。 使用文本編輯器或`cat`命令建立與以下示例類似的檔案：

1. 開啟終端，將當前目錄更改為dispkey.pem檔案的位置。
1. 要解密私鑰，請輸入以下命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文字編輯器或`cat`命令，將未加密的私密金鑰和憑證結合為類似下列範例的單一檔案：

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

### 指定要用於Dispatcher {#specifying-the-certificate-to-use-for-dispatcher}的證書

將以下屬性添加到[Dispatcher模組配置](dispatcher-install.md#main-pars-55-35-1022)（在`httpd.conf`中）:

* `DispatcherCertificateFile`:指向Dispatcher統一證書檔案的路徑，該檔案包含公共證書和未加密的私鑰。當SSL伺服器請求Dispatcher客戶端證書時，將使用此檔案。
* `DispatcherCACertificateFile`:CA憑證檔案的路徑，當SSL伺服器呈現根授權機構不信任的CA時使用。
* `DispatcherCheckPeerCN`:是否啟用( `On`)或禁用( `Off`)主機名檢查遠程伺服器證書。

下列程式碼為範例設定：

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

