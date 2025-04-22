---
title: BA Assets解包
date: 2025-04-22
tags: [Python,Json,Base64]
head: 
    - - meta
      - name: description
        content: 只是关于unity的研究分享，不可用于其他商业用途
    - - meta
      - name: keywords
        content: BlueArchive Python
---

BA Assets解包思路分享，不可用于其他商业用途

---

# 关于BlueArchive

《Blue Archive》（碧蓝档案/蔚蓝档案）是由韩国Nexon旗下子公司NAT Games开发、Yostar代理发行的一款二次元风格**角色收集RPG**，主打学园题材与战略战斗玩法。

# 如何获取游戏

## 日服/国际服

1. 从Google Play/App Store上获取完整的游戏至手机或模拟器，这里不介绍该方法。
2. 使用第三方平台获取apk，比如APK Pure/Apk Combo，这里使用Apk Combo的直链获取到游戏的完整安装包。

**Apk Combo获取**Blue Archive JP

> 请确保你能链接上apkcombo的服务器并且不会被cloudflare拦截

[apkcombo](https://apkcombo.com)

[apkcombo下的Blue Archive JP](https://apkcombo.com/blue-archive-jp/com.YostarJP.BlueArchive/download/apk)

在**apkcombo**的Blue Archive JP页面中找到位于**Download**和**Old Versions**间的**Blue Archive JP···**，点击右边的下载按钮，即可以获取到完整的安装包，如果你需要制作对应的软件来解决你的问题，可以使用https响应下载或者使用它临时提供的服务器直链下载链接。
## 沙勒服

从上海悠星的官网上获取下载链接或者使用第三方平台提供的渠道服进行下载，这里不做演示。

---
# 如何对下载的安装包处理

这里使用Python来对下载的Xapk安装包处理，需要安装的依赖包如下
>requests
>
>UnityPy==1.20.22
>
>cloudscraper
>
>xxhash
>
>pycryptodome
>
>flatbuffers
>
>pykakasi

解压安装包到你的工作目录，到这一步我们需要在安装包中寻找关于游戏下载资源的链接。

1. 对解压后的文件进行遍历寻找关于assets资产文件夹，bin文件夹，Data文件夹中的GameMainConfig。
2. 提取里面的资源下载的URL链接并对其解码成原始网址便于后续处理。

寻找apk中的URL示例
```python
url = version = ""
# 遍历APK解压文件夹中的文件

        for dir, _, files in os.walk(

            path.join(self.APK_EXTRACT_FOLDER, "assets", "bin", "Data")

        ):

            for file in files:

                # 搜索Unity包中的GameMainConfig

                if url_obj := UnityUtils.search_unity_pack(

                    path.join(dir, file), ["TextAsset"], ["GameMainConfig"], True

                ):

                    # 解码服务器URL

                    url = self.__decode_server_url(url_obj[0].read().m_Script.encode("utf-8", "surrogateescape"))  # type: ignore

                    notice(f"Get URL successfully: {url}")

                # 搜索Unity包中的PlayerSettings

                if version_obj := UnityUtils.search_unity_pack(

                    path.join(dir, file), ["PlayerSettings"]

                ):

                    try:

                        version = version_obj[0].read().bundleVersion  # type: ignore

                    except:

                        version = "unavailable"

                    print(f"The apk version is {version}.")

  

                # 如果URL和版本号都已获取，则退出循环

                if url and version:

                    break
```

解码服务器URL的示例实现

```python
def __decode_server_url(self, data: bytes) -> str:
        # 定义密钥字典

        ciphers = {

            "ServerInfoDataUrl": "X04YXBFqd3ZpTg9cKmpvdmpOElwnamB2eE4cXDZqc3ZgTg==",

            "DefaultConnectionGroup": "tSrfb7xhQRKEKtZvrmFjEp4q1G+0YUUSkirOb7NhTxKfKv1vqGFPEoQqym8=",

            "SkipTutorial": "8AOaQvLC5wj3A4RC78L4CNEDmEL6wvsI",

            "Language": "wL4EWsDv8QX5vgRaye/zBQ==",

        }

        # 对数据进行Base64编码

        b64_data = base64.b64encode(data).decode()

        # 使用密钥转换字符串

        json_str = convert_string(b64_data, create_key("GameMainConfig"))

        # 解析JSON字符串

        obj = json.loads(json_str)

        # 获取加密的URL

        encrypted_url = obj[ciphers["ServerInfoDataUrl"]]

        # 使用密钥转换URL

        url = convert_string(encrypted_url, create_key("ServerInfoDataUrl"))

        return url
```

到这一步，我们获取到了游戏在下载资源使用的原始URL链接。

3. 尝试对获取到的URL链接进行api响应并对获得响应。
4. 正常响应后尝试对目标进行请求获取。
5. 解析表格、媒体、资源包目录整理成资源清单。

资源清单获取的方法示例

```python
esources = JPResource()

        try:

            # 获取资源清单API响应

            if not (api := FileDownloader(server_url).get_response()):

                raise LookupError(

                    "Cannot access resource url. Retry may solve this issue."

                )

  

            # 获取基础URL

            base_url = (

                api.json()["ConnectionGroups"][0]["OverrideConnectionGroups"][-1][

                    "AddressablesCatalogUrlRoot"

                ]

                + "/"

            )

  

            # 设置资源URL

            resources.set_url_link(

                base_url, "Android/", "MediaResources/", "TableBundles/"

            )

  

            # 获取并解密表格目录

            if table_data := FileDownloader(

                urljoin(resources.table_url, "TableCatalog.bytes")

            ).get_bytes():

                JPCatalogDecoder.decode_to_manifest(table_data, resources, "table")

            else:

                notice(

                    "Failed to fetch table catalog. Retry may solve this issue.",

                    "error",

                )

  

            # 获取并解密媒体目录

            if media_data := FileDownloader(

                urljoin(resources.media_url, "Catalog/MediaCatalog.bytes")

            ).get_bytes():

                JPCatalogDecoder.decode_to_manifest(media_data, resources, "media")

            else:

                notice(

                    "Failed to fetch media catalog. Retry may solve this issue.",

                    "error",

                )

  

            # 获取并解析资源包目录

            if (

                bundle_data := FileDownloader(

                    urljoin(resources.bundle_url, "bundleDownloadInfo.json")

                ).get_response()

            ) and bundle_data.headers.get("Content-Type") == "application/json":

                bundle_catalog = bundle_data.json()["BundleFiles"]

                for bundle in bundle_catalog:

                    resources.add_bundle_resource(

                        bundle["Name"],

                        bundle["Size"],

                        bundle["Crc"],

                        bundle["IsPrologue"],

                        bundle["IsSplitDownload"],

                    )

            else:

                notice(

                    "Failed to fetch bundle catalog. Retry may solve this issue.",

                    "error",

                )

  

            # 如果资源清单为空，则抛出异常

            if not resources:

                raise FileNotFoundError("Cannot pull the manifest.")

  

            return resources.to_resource()

  

        except Exception as e:

            raise LookupError(

                f"Encountered the following error while attempting to fetch manifest: {e}."

            ) from e
```

123