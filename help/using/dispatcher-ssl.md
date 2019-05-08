---
title: 搭配Dispatcher使用SSL
seo-title: 搭配Dispatcher使用SSL
description: 瞭解如何使用SSL連線，設定Dispatcher與AEM通訊。
seo-description: 瞭解如何使用SSL連線，設定Dispatcher與AEM通訊。
uuid: 1a8f448c-d3 d8-4798-a5 cb-9579171171ed
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: 771domeration85-6c26-4ff2-a3 Fe-dff8 d8 f7920 b
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 搭配Dispatcher使用SSL {#using-ssl-with-dispatcher}

使用Dispatcher和演算電腦之間的SSL連線：

* [單向SSL](dispatcher-ssl.md#main-pars-title-1)
* [共同SSL](dispatcher-ssl.md#main-pars-title-2)

>[!NOTE]
>
>與SSL憑證相關的操作會系結至第三方產品。Adobe白金級維護與支援合約未涵蓋這些項目。

## Dispatcher連線至AEM時使用SSL {#use-ssl-when-dispatcher-connects-to-aem}

設定Dispatcher以使用SSL連線與AEM或CQ演算例項通訊。

設定Dispatcher之前，請設定AEM或CQ以使用SSL：

* AEM6.2： [啓用HTTP over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM6.1： [啓用HTTP over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 較舊的AEM版本：檢視 [此頁面](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)。

### SSL相關請求標題 {#ssl-related-request-headers}

當Dispatcher收到HTTPS請求時，Dispatcher會在後續要求中包含下列標題，以便傳送至AEM或CQ：

* `X-Forwarded-SSL`
* `X-Forwarded-SSL-Cipher`
* `X-Forwarded-SSL-Keysize`
* `X-Forwarded-SSL-Session-ID`

透過Apache-2.2的請求 `mod_ssl` 包含類似下列範例的標題：

```shell
X-Forwarded-SSL: on
X-Forwarded-SSL-Cipher: DHE-RSA-AES256-SHA
X-Forwarded-SSL-Session-ID: 814825E8CD055B4C166C2EF6D75E1D0FE786FFB29DEB6DE1E239D5C771CB5B4D
```

### 設定Dispatcher以使用SSL {#configuring-dispatcher-to-use-ssl}

若要設定Dispatcher以透過SSL或CQ(透過SSL [)連線至AEM或CQ，請使用](dispatcher-configuration.md) 下列屬性：

* 處理HTTPS請求的虛擬主機。
* 虛擬主機 `renders` 的區段包含一個項目，可識別使用HTTPS的CQ或AEM實例的主機名稱和連接埠。
* `renders` 項目包含指名值的 `secure` 屬性 `1`。

注意：視需要建立另一個虛擬主機來處理HTTP要求。

下列範例dispatcher. any檔案會顯示使用HTTPS連接至主機 `localhost` 和連接埠上執行之CQ實例的屬性值 `8443`：

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

## 在Dispatcher和AEM之間設定相互SSL {#configuring-mutual-ssl-between-dispatcher-and-aem}

設定Dispatcher和演算電腦(通常是AEM或CQ發佈例項)之間的連線，以使用共同SSL：

* Dispatcher會透過SSL連線至演算例項。
* 演算例項會驗證Dispatcher憑證的有效性。
* Dispatcher會驗證演算例項憑證的CA是否受信任。
* (選擇性) Dispatcher驗證演算例項的憑證是否符合演算例項的伺服器位址。

若要設定共同SSL，您需要受信任憑證授權機構(CA)簽署的憑證。自行簽署的憑證不適用。您可以作為CA或使用第三方CA的服務簽署憑證。若要設定共同SSL，您需要下列項目：

* 演算例項和Dispatcher的簽署憑證
* CA憑證(如果您是CA)
* 用於產生CA、憑證和憑證要求的OpenSSL程式庫。

執行下列步驟以設定共同SSL：

1. [安裝](dispatcher-install.md) 適用於平台的最新版Dispatcher。使用支援SSL的Dispatcher二進位(SSL位於檔案名稱中，例如dispatcher-apache2.4-linux-x86-64-ssl10-4.1.7.tar)。
1. [針對Dispatcher建立或取得CA簽署憑證](dispatcher-ssl.md#main-pars-title-3) ，以及演算例項。
1. [建立包含演算憑證的keystore](dispatcher-ssl.md#main-pars-title-6) ，並設定演算的HTTP服務以使用它。
1. [為共同SSL設定Dispatcher網頁伺服器模組](dispatcher-ssl.md#main-pars-title-4) 。

### 建立或取得CA簽署憑證 {#creating-or-obtaining-ca-signed-certificates}

建立或取得驗證發佈例項和Dispatcher的CA簽署憑證。

#### 建立您的CA {#creating-your-ca}

如果您是CA，請使用 [OpenSSL](https://www.openssl.org/) 建立憑證授權機構，以簽署伺服器和用戶端憑證。(您必須安裝OpenSSL程式庫)。如果您使用第三方CA，請勿執行此程序。

1. 開啓終端機，並將目前目錄變更為銜接CA. sh檔案的目錄，例如 `/usr/local/ssl/misc`。
1. 若要建立CA，請輸入下列命令，然後在promited時提供值：

   ```shell
   ./CA.sh -newca
   ```

   >[!NOTE]
   >
   >opensl. cnf檔案中的數個屬性控制CA. sh指令檔的行為。您必須視需要修改此檔案，才能建立CA。

#### 建立憑證 {#creating-the-certificates}

使用OpenSSL建立憑證要求，以傳送至第三方CA或與CA簽署。

當您建立憑證時，OpenSSL會使用通用名稱屬性來識別憑證持有人。對於演算例項的憑證，如果您要設定Dispatcher只接受「發佈」例項的主機名稱，請使用例項電腦的主機名稱作為「通用名稱」來接受憑證。(請參閱 [DispatchCheckerCN](dispatcher-ssl.md#main-pars-title-11) 屬性)。

1. 開啓終端機並將目前目錄變更為包含OpenSSL程式庫之CH. sh檔案的目錄。
1. 輸入下列命令，並在提示時提供值。如有必要，請使用發佈例項的主機名稱作為通用名稱。主機名稱為DNS位址的DNS可解析名稱：

   ```shell
   ./CA.sh -newreq
   ```

   如果您使用第三方CA，請將newreq. pem檔案傳送至CA以簽署。如果您是CA，請繼續步驟3。

1. 輸入下列命令，使用CA的憑證簽署憑證：

   ```shell
   ./CA.sh -sign
   ```

   包含您的CA管理檔案的目錄中會建立兩個名為newcert. pem和newkey. pem的檔案。這些是轉換電腦的公用憑證和私密金鑰。

1. 重新命名newcert. pem以rendercert. pem，並將newkey. pem重新命名為renderkey. pem。
1. 重復步驟和3，為Dispatcher模組建立新的憑證和新的公開金鑰。請確定您使用的是Dispatcher例項專屬的通用名稱。
1. 重新命名newcert. pem以刪除. pem，並將newkey. pem重新命名為diskey. pem。

### 在Render Computer上設定SSL {#configuring-ssl-on-the-render-computer}

使用rendercert. pem和renderkey. pem檔案在演算例項上設定SSL。

#### 將演算憑證轉換為JKS格式 {#converting-the-render-certificate-to-jks-format}

使用下列逗號並將轉譯憑證轉換為PKCS#12檔案。也包含簽署轉換憑證之CA的憑證：

1. 在終端機介面視窗中，將目前目錄變更為轉譯憑證和私密金鑰的位置。
1. 輸入下列逗號並將轉譯憑證轉換為PKCS#12檔案。也包含簽署轉換憑證之CA的憑證：

   ```shell
   openssl pkcs12 -export -in rendercert.pem -inkey renderkey.pem  -certfile demoCA/cacert.pem -out rendercert.p12
   ```

1. 輸入下列命令，將PKCS#12檔案轉換為Java KeyStore(JKS)格式：

   ```shell
   keytool -importkeystore -srckeystore servercert.p12 -srcstoretype pkcs12 -destkeystore render.keystore
   ```

1. Java Keystore是使用預設別名建立。視需要變更別名：

   ```shell
   keytool -changealias -alias 1 -destalias jettyhttp -keystore render.keystore
   ```

#### 將CA Cert新增至Render&#39;sTruststore {#adding-the-ca-cert-to-the-render-s-truststore}

如果您要當做CA，請將CA憑證匯入鑰匙市。然後，設定執行演算例項的JVM，以信任鑰匙存放區。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2014-08-12T13:11:21.401-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The jetty http service has properties to specify trusted CA certificates for mutual SSL for 6.0. Whether they are operable is undetetermined. See https://issues.adobe.com/browse/DOC-4738.</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;"> </p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For 5.6.1, you would specify the system property javax.net.ssl.trustStore, using the path to cacerts as value.</p>

 -->

1. 使用文字編輯器開啓cacert. pem檔案，並移除所有優先行的文字：

   `-----BEGIN CERTIFICATE-----`

1. 使用下列命令將憑證匯入至鑰匙存放區：

   ```shell
   keytool -import -keystore cacerts.keystore -alias myca -storepass password -file cacert.pem
   ```

1. 若要設定執行演算例項以信任鑰匙存放區的JVM，請使用下列系統屬性：

   ```shell
   -Djavax.net.ssl.trustStore=<location of cacerts.keystore>
   ```

   例如，如果您使用crx-quickstart/bin/Quickstart指令碼來啓動您的發佈例項，您可以修改CQ_ JVM_ OPTS屬性：

   ```shell
   CQ_JVM_OPTS='-server -Xmx2048m -XX:MaxPermSize=512M -Djavax.net.ssl.trustStore=/usr/lib/cq6.0/publish/ssl/cacerts.keystore'
   ```

#### 設定演算例項 {#configuring-the-render-instance}

使用「發佈例項 ** 」區段中「啓用SSL」中的指示，使用演算憑證來設定演算例項的HTTP服務，以使用SSL：

* AEM6.2： [啓用HTTP over SSL](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/config-ssl.html)
* AEM6.1： [啓用HTTP over SSL](https://docs.adobe.com/content/docs/en/aem/6-1/deploy/configuring/config-ssl.html)
* 較舊的AEM版本：檢視 [此頁面。](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)

### 設定Dispatcher模組的SSL {#configuring-ssl-for-the-dispatcher-module}

若要設定Dispatcher使用共同SSL，請準備Dispatcher憑證，然後設定Web伺服器模組。

### 建立統一的Dispatcher憑證 {#creating-a-unified-dispatcher-certificate}

將dispatcher憑證和未加密的私密金鑰結合為單一PEM檔案。使用文字編輯器或 `cat` 命令建立類似下列範例的檔案：

1. 開啓終端機並將目前目錄變更為diskey. pem檔案的位置。
1. 若要解密私密金鑰，請輸入下列命令：

   ```shell
   openssl rsa -in dispkey.pem -out dispkey_unencrypted.pem
   ```

1. 使用文字編輯器或 `cat` 命令，將未加密的私密金鑰和憑證合併為類似下列範例的單一檔案：

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

### 指定要用於Dispatcher的憑證 {#specifying-the-certificate-to-use-for-dispatcher}

將下列屬性新增至 [Dispatcher模組組態(](dispatcher-install.md#main-pars-55-35-1022) in `httpd.conf`)：

* `DispatcherCertificateFile`：Dispatcher統一憑證檔案的路徑，包含公開憑證和未加密的私密金鑰。當SSL伺服器要求Dispatcher用戶端憑證時，就會使用此檔案。
* `DispatcherCACertificateFile`：通往CA憑證檔案的路徑，如果SSL伺服器呈現不受根授權信任的CA。
* `DispatcherCheckPeerCN`：是否啓用( `On`)或停用( `Off`)主機名稱檢查遠端伺服器憑證。

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

