# 什么是阿里云 OSS？

- **对象存储服务 (Object Storage Service, OSS)** 是阿里云提供的一种海量、安全、低成本、高可靠的云存储服务。可以把它想象成一个无限大的网络硬盘，可以随时随地通过互联网 API 存储和访问任何类型的文件，包括图片、音视频、日志文件、文档等。
  - 与传统的文件存储（如本地硬盘或 FTP）不同，OSS 不是以文件目录树的结构来组织文件，而是将每个文件作为一个独立的对象 (Object) 来管理。每个对象都包含三个部分：
    1. **Key (键)**：对象的唯一标识符，可以理解为文件名，例如 `images/avatar/user01.jpg`。
    2. **Data (数据)**：文件本身的内容。
    3. **Metadata (元数据)**：描述对象信息的键值对，例如文件大小、最后修改时间、内容类型 (Content-Type) 等。



# 核心概念

| 概念         | 英文          | 描述                                                         |
| ------------ | ------------- | ------------------------------------------------------------ |
| **存储空间** | **Bucket**    | 存储对象的容器，是 OSS 的全局命名空间。**Bucket 名称在整个阿里云 OSS 中必须是唯一的**。每个 Bucket 都有自己的访问权限、地域等属性。你可以把它理解为顶级文件夹。 |
| **对象**     | **Object**    | OSS 中存储的基本单元，就是我们上传的文件。每个 Object 都由 Key、Data 和 Metadata 组成。 |
| **地域**     | **Region**    | 指 Bucket 所在的数据中心的物理位置，例如 `华东1 (杭州)`、`华北2 (北京)` 等。选择靠近你用户所在地的地域可以降低访问延迟。**一旦 Bucket 创建成功，其地域不可更改**。 |
| **访问域名** | **Endpoint**  | OSS 对外服务的访问地址，每个 Region 都有一个唯一的 Endpoint。<br />         例如，杭州地域的 Endpoint 是 `oss-cn-hangzhou.aliyuncs.com`。 |
| **访问密钥** | **AccessKey** | 访问阿里云 API 的身份验证凭证，由 **AccessKey ID** 和 **AccessKey Secret** 组成，代表了你的账户身份和权限。**要妥善保管这个，不能泄露**。 |



# 主要功能与优势

## 1. 多种存储类型

- OSS 提供了多种存储类型，以满足不同场景下的成本和性能要求：

  - **标准存储 (Standard)**：默认类型。高可靠、高可用、高性能，适合需要频繁访问和读写的数据，如网站图片、热点视频等。

  - **低频访问存储 (Infrequent Access, IA)**：存储单价低于标准型，但会产生数据取回费用。适合不经常访问（平均每月访问频率低于1次）的数据，如监控视频、日志备份。

  - **归档存储 (Archive)**：存储单价极低，但数据需要先“解冻”（通常需要几分钟）才能访问。适合需要长期保存（至少180天）的冷数据，如医疗影像、财务档案。

  - **冷归档存储 (Cold Archive)**：比归档存储更便宜，但解冻时间更长（通常需要数小时）。适合需要极长期保存的极冷数据。
  - **深度冷归档存储**：......



## 2. 强大的安全性

- **访问控制**：通过 Bucket/Object 的 ACL (访问控制列表)、Bucket Policy 和 RAM (访问控制) 策略，可以实现精细化的权限管理。
- **加密机制**：支持服务端加密 (SSE-OSS/SSE-KMS) 和客户端加密，保护数据在传输和存储过程中的安全。
- **日志审计**：记录所有对 OSS 的访问请求，方便进行安全分析和合规审计。
- **防盗链**：通过 Referer 白名单设置，可以防止你的 OSS 资源被其他网站非法盗用。



## 3. 海量数据处理能力

- **图片处理 (IMG)**：可以对图片进行缩放、裁剪、旋转、加水印等多种操作，只需在图片 URL 后附加参数即可实时处理，非常方便。
- **音视频处理 (MTS)**：可以进行视频转码、截图、拼接等操作。
- **文件解压缩**：可以直接在云端对压缩包进行解压。



## 4. 高可用与高可靠

- **数据冗余**：数据在同一地域内的多个数据中心（可用区）之间进行冗余存储，确保数据持久性高达 99.9999999999% (12个9)。
- **服务可用性**：服务可用性高达 99.995%。
- **跨区域复制**：可以将数据自动、异步地复制到其他地域的 Bucket，实现异地容灾。



# 与 SpringBoot 集成

- 将 OSS 与 SpringBoot 结合，通常是为了实现文件上传、下载、删除和访问等功能。基本步骤如下：

## 1. 添加依赖

- 在 Spring Boot 项目的 `pom.xml` 文件中，引入阿里云 OSS 的 Java SDK 依赖。

  ```xml
  <dependency>
      <groupId>com.aliyun.oss</groupId>
      <artifactId>aliyun-sdk-oss</artifactId>
      <version>3.15.1</version> <!-- 建议使用较新的稳定版本 -->
  </dependency>
  ```

  - 如果使用的是Java 9及以上的版本，则还需要添加以下JAXB相关依赖

    ```xml
    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
        <version>2.3.1</version>
    </dependency>
    <dependency>
        <groupId>javax.activation</groupId>
        <artifactId>activation</artifactId>
        <version>1.1.1</version>
    </dependency>
    <!-- no more than 2.3.3-->
    <dependency>
        <groupId>org.glassfish.jaxb</groupId>
        <artifactId>jaxb-runtime</artifactId>
        <version>2.3.3</version>
    </dependency>
    ```

    

## 2. 配置文件

- 在 `application.yml` 或 `application.properties` 中配置 OSS 相关的参数，避免硬编码在代码中。

  ```yaml
  aliyun:
    oss:
      endpoint: oss-cn-hangzhou.aliyuncs.com # 你的 Endpoint
      access-key-id: LTAI5t... # 你的 AccessKey ID
      access-key-secret: Gx7r... # 你的 AccessKey Secret
      bucket-name: your-bucket-name # 你的 Bucket 名称
  ```



# OSSClient

## 什么是 OSSClient

- `OSSClient` 是阿里云 OSS Java SDK 中进行所有操作的**核心入口**

- 简单来说，它是 Java 应用程序与阿里云 OSS 服务进行通信的**唯一客户端**。代码中的所有指令，如上传、下载、删除等，都必须通过它来进行



## OSSClient 的核心职责

### 1. **身份认证**

- 在创建 `OSSClient` 实例时，必须提供三样东西：`Endpoint`、`AccessKey ID` 和 `AccessKey Secret`。之后，它会**自动处理所有与 OSS 服务器交互时的签名和身份验证**，确保每一次操作都是安全合法的。
  - `Endpoint` 表示连接到哪个地域的 OSS 服务
  - `AccessKey ID` 和 `AccessKey Secret` ：阿里云提供的**身份验证凭证**，专门用于通过程序（如 Java SDK、Python脚本等）访问云服务 API



### 2. **对象操作**

- 这是最核心的一组功能，用于管理存储在 Bucket 中的数据实体（Object）

  | 核心方法            | 功能描述                                                     |
  | ------------------- | ------------------------------------------------------------ |
  | `putObject()`       | 上传一个对象，支持输入流、字节数组或本地文件等多种形式。     |
  | `getObject()`       | 下载一个对象，返回一个包含对象元数据和内容输入流的 `OSSObject`。 |
  | `deleteObject()`    | 删除一个或多个指定对象。                                     |
  | `listObjects()`     | 分页列出指定 Bucket 内的对象列表。                           |
  | `doesObjectExist()` | 检查指定名称的对象是否存在于 Bucket 中。                     |



### 3. **存储空间操作**

- 用于管理 OSS 的顶级容器，即 Bucket

  | 核心方法         | 功能描述                      |
  | ---------------- | ----------------------------- |
  | `createBucket()` | 创建一个新的 Bucket。         |
  | `deleteBucket()` | 删除一个空的 Bucket。         |
  | `listBuckets()`  | 列出调用者拥有的所有 Bucket。 |



### 4. **预签名 URL 生成**

- 用于实现对私有资源的安全临时访问授权

  | 核心方法                 | 功能描述                                                     |
  | ------------------------ | ------------------------------------------------------------ |
  | `generatePresignedUrl()` | 为私有对象生成一个带有时效性签名的 URL。<br />任何持有该 URL 的用户都可以在过期前拥有对该对象的临时访问权限（如读或写）。 |



## 关键设计与最佳实践

### 1. **生命周期管理：单例模式**

- `OSSClient` 被设计为**线程安全**的对象。其实例化和销毁是重量级操作，因为它涉及到内部 HTTP 连接池的初始化和释放。

  - **✅ 正确实践**: 在整个应用程序的生命周期内，应**维护一个 `OSSClient` 的单例实例**。在 Spring/SpringBoot 框架中，最佳实践是将其声明为一个单例作用域的 Bean，并通过依赖注入在需要的地方使用。

  - **❌ 错误实践**: 在每次请求或方法调用中都 `new OSSClient()`。这种做法会导致连接池被频繁创建和销毁，从而引发严重的性能瓶颈和潜在的套接字资源耗尽。



### 2. **资源释放：优雅关闭**

- 当应用程序关闭时，必须显式调用 `ossClient.shutdown()` 方法

  - **作用**: 该方法会平滑地关闭内部的 HTTP 连接池，并释放所有关联的网络资源，防止内存和连接泄漏。

  - **实现**: 在 Spring 框架中，可以通过 Bean 的 `destroy-method` 属性或实现 `DisposableBean` 接口，在容器销毁 Bean 时自动调用此方法





# 阿里云OSS的一般使用步骤

![image-20250825160738800](./assets/image-20250825160738800.png)



# 具体细节编码

- 参考官方文档实现

  阿里云OSS官方文档地址：https://help.aliyun.com/zh/oss/





# 阿里云模板代码示例解析

## 模板代码示例

```java
package com.avexVertex;

import com.aliyun.oss.*;
import com.aliyun.oss.common.auth.*;
import com.aliyun.oss.common.comm.SignVersion;
import com.aliyun.oss.model.PutObjectRequest;
import com.aliyun.oss.model.PutObjectResult;

import java.io.File;

public class Demo {

    public static void main(String[] args) throws Exception {
        // Endpoint以华东1（杭州）为例，其它 Region 请按实际情况填写。
        String endpoint = "https://oss-cn-shanghai.aliyuncs.com";
        // 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
        EnvironmentVariableCredentialsProvider credentialsProvider = CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider();
        // 填写Bucket名称，例如examplebucket
        String bucketName = "campus-link-bucket";
        // 填写Object完整路径，完整路径中不能包含Bucket名称，例如exampledir/exampleobject.txt
        String objectName = "001.jpg";
        // 填写本地文件的完整路径，例如D:\\localpath\\examplefile.txt
        // 如果未指定本地路径，则默认从示例程序所属项目对应本地路径中上传文件
        String filePath = "F:\\Photo\\Background\\yourname.png";
        // 填写Bucket所在地域。以华东1（杭州）为例，Region填写为cn-hangzhou
        String region = "cn-shanghai";

        // 创建OSSClient实例。
        // 当OSSClient实例不再使用时，调用shutdown方法以释放资源。
        ClientBuilderConfiguration clientBuilderConfiguration = new ClientBuilderConfiguration();
        clientBuilderConfiguration.setSignatureVersion(SignVersion.V4);
        OSS ossClient = OSSClientBuilder.create()
                .endpoint(endpoint)
                .credentialsProvider(credentialsProvider)
                .clientConfiguration(clientBuilderConfiguration)
                .region(region)
                .build();

        try {
            // 创建PutObjectRequest对象。
            PutObjectRequest putObjectRequest = new PutObjectRequest(bucketName, objectName, new File(filePath));
            // 如果需要上传时设置存储类型和访问权限，请参考以下示例代码。
            // ObjectMetadata metadata = new ObjectMetadata();
            // metadata.setHeader(OSSHeaders.OSS_STORAGE_CLASS, StorageClass.Standard.toString());
            // metadata.setObjectAcl(CannedAccessControlList.Private);
            // putObjectRequest.setMetadata(metadata);

            // 上传文件。
            PutObjectResult result = ossClient.putObject(putObjectRequest);
        } catch (OSSException oe) {
            System.out.println("Caught an OSSException, which means your request made it to OSS, "
                    + "but was rejected with an error response for some reason.");
            System.out.println("Error Message:" + oe.getErrorMessage());
            System.out.println("Error Code:" + oe.getErrorCode());
            System.out.println("Request ID:" + oe.getRequestId());
            System.out.println("Host ID:" + oe.getHostId());
        } catch (ClientException ce) {
            System.out.println("Caught an ClientException, which means the client encountered "
                    + "a serious internal problem while trying to communicate with OSS, "
                    + "such as not being able to access the network.");
            System.out.println("Error Message:" + ce.getMessage());
        } finally {
            if (ossClient != null) {
                ossClient.shutdown();
            }
        }
    }
}
```



## 变量含义详解

### 1. `endpoint`

- **代码中的值**: `"https://oss-cn-shanghai.aliyuncs.com"`
- **含义**: **OSS 服务的访问域名**。这是你要连接的阿里云 OSS 服务器的总地址。
- **比喻**: 这是**国际快递处理中心**的地址。所有发往“华东上海”区域的包裹，都必须先送到这里。



### 2. `credentialsProvider`

- **代码中的值**: `CredentialsProviderFactory.newEnvironmentVariableCredentialsProvider()`
- **含义**: **凭证提供者**。这是一个对象，它的任务是提供你的 AccessKey ID 和 Secret，用来向 OSS 证明你的身份。这个特定的提供者会去你电脑的**系统环境变量**里寻找 `OSS_ACCESS_KEY_ID` 和 `OSS_ACCESS_KEY_SECRET` 这两个值。
- **比喻**: 你的**身份证和授权书**。你告诉快递员：“我的证件信息已经预先登记在系统里了（环境变量），你自己去查吧。



### 3. `bucketName`

- **代码中的值**: `"campus-link-bucket"`
- **含义**: **存储空间（Bucket）的名称**。Bucket 是你在 OSS 上创建的用来存放文件的“容器”，可以理解为一个根目录。这个名字在阿里云全球范围内必须是唯一的。
- **比喻**: 包裹上的**收件人姓名**。这个名字是独一无二的，确保包裹能送到你专属的储物柜



### 4. `objectName`

- **代码中的值**: `"001.jpg"`
- **含义**: **对象（Object）的完整路径和名称**。
  - 这就是你的文件上传到 Bucket 之后的名字。这个名字不是域名，不包含`https://`开头的那些东西
    - 你也可以包含路径，比如 `"images/avatars/001.jpg"`，OSS 会自动帮你创建对应的文件夹
- **比喻**: 这是写在包裹上的**“物品清单”和“存放位置”**。比如“个人照片/头像/001.jpg”，快递员会按照这个指示把你的物品放到储物柜里对应的位置



### 5. `filePath`

- **代码中的值**: `"F:\\Photo\\Background\\yourname.png"`
- **含义**: **本地文件的完整路径**。它告诉程序，要去你电脑的哪个位置找到那个准备要上传的文件
- **比喻**: 你告诉快递员：“我要寄的那个**包裹就放在我家书房的桌子上**”，这是一个具体的、物理的地址



### 6. `region`

- **代码中的值**: `"cn-shanghai"`
- **含义**: **Bucket 所在的地理区域**。这个值必须和你创建 Bucket 时选择的区域完全一致
- **比喻**: 在快递单上**再次确认收件人所在的城市**是“上海”，帮助系统进行最高效的路由和处理



### 7. `clientBuilderConfiguration` 和 `ossClient`

- **含义**: 这两个和我们之前讨论的一样，是用来**创建最终的 OSS 客户端实例**的。`clientBuilderConfiguration` 用来设置高级选项（如 V4 签名），`ossClient` 则是配置完成后，真正用来执行操作的工具。
- **比喻**: 准备好**特殊寄件要求**（`clientBuilderConfiguration`），然后生成一张包含了所有信息的、可以使用的**快递单**（`ossClient`）。



### 8. `putObjectRequest`

- **代码中的值**: `new PutObjectRequest(bucketName, objectName, new File(filePath))`
- **含义**: **一个封装好的“上传请求”对象**。它把这次上传需要的所有核心信息都打包在了一起：要上传到哪个 Bucket (`bucketName`)，上传后叫什么名字 (`objectName`)，以及要上传的本地文件是什么 (`new File(filePath)`)。
- **比喻**: 快递员把你**要寄的包裹**（`new File(filePath)`）和你**填好的快递单**（包含 `bucketName` 和 `objectName`）**正式绑定在一起**，准备装车。



### 9. `result`

- **代码中的值**: `ossClient.putObject(putObjectRequest)` 的返回值
- **含义**: **上传操作的结果**。如果上传成功，这个对象里会包含一些返回信息，比如文件的 ETag（一个可以用来校验文件完整性的哈希值）。
- **比喻**: 快递寄出去后，快递公司给你的一个**回执单**，上面有追踪号码和签收确认信息。



