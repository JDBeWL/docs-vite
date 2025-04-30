---
title: BA Assets解析
date: 2025-04-22
tags:
  - Python
  - Json
  - Base64
head:
  - - meta
    - name: description
      content: 只是关于unity的研究分享，不可用于其他商业用途
  - - meta
    - name: keywords
      content: BlueArchive Python
---

BA Assets解析思路分享，不可用于其他商业用途

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

## 关于代码

部分代码来源于[Blue-Archive-Asset-Downloader](https://github.com/ZM-Kimu/Blue-Archive-Asset-Downloader)，请到原作者的仓库star支持作者。
这里仅仅只是对于作者提供的思路进行对unity工程的逆向分析。

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
                    url = self.__decode_server_url(url_obj[0].read().m_Script.encode("utf-8", "surrogateescape"))  # type: ignore
                    notice(f"Get URL successfully: {url}")
                # 搜索Unity包中的PlayerSettings
                if version_obj := UnityUtils.search_unity_pack(
                    path.join(dir, file), ["PlayerSettings"]
                ):
                    try:
                        version = version_obj[0].read().bundleVersion  # type: ignore
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
5. 使用下载器对获取到的资源进行下载。
6. 解析表格、媒体、资源包目录整理成资源清单。

> 在获取资源清单前请一定确保你取得的Json地址正确。

资源清单获取的方法示例

```python
    resources = JPResource()
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

按照该步骤即可对目前的资源进行处理，下面将解释详细的解析方法。

# 如何进行解析

> 在正式解析资产之前，我声明对于解析之后的所有后果都与作者和我无关，请不要将其用于不正确的用途。

1. 上面定义了一个**JPCatalogDecoder**类，里面封装有**Reader**方法，这个方法用于对实际取得的数据进行转换成可读数据结构。

Reader方法示例

```python
    # 定义Reader类，用于读取二进制数据
    class Reader:
        def __init__(self, initial_bytes) -> None:
            # 初始化读取器，将初始字节流存储在BytesIO对象中
            self.io = BytesIO(initial_bytes)

        def read(self, fmt: str) -> Any:
            # 使用struct模块根据给定的格式字符读取字节
            # struct.calcsize(fmt)计算格式字符fmt所需的字节数
            # self.io.read(struct.calcsize(fmt))从字节流中读取相应数量的字节
            # struct.unpack(fmt, ...)将读取的字节解析为指定格式的数据
            # [0]取出解析后的第一个元素
            return struct.unpack(fmt, self.io.read(struct.calcsize(fmt)))[0]

        def read_string(self) -> str:
            # 读取字符串
            # 首先读取字符串的长度（读32位）
            # 然后根据长度读取相应数量的字节
            # 使用utf-8编码将字节解码为字符串，如果遇到无法解码的字符则替换为'?'（errors="replace"）
            return self.io.read(self.read(I32)).decode(
                encoding="utf-8", errors="replace"
            )

        def read_table_includes(self) -> list[str]:
            # 读取表包含的字符串列表
            # 首先读取列表的大小（32位大小）
            size = self.read(I32)
            if size == -1:
                # 如果大小为-1，表示没有包含任何字符串，返回空列表
                return []
            # 读取一个32位的数据
            self.read(I32)
            includes = []
            for i in range(size):
                # 依次读取每个字符串并添加到列表中
                includes.append(self.read_string())
                if i != size - 1:
                    # 在读取每个字符串之间，读取一个32位的数据
                    self.read(I32)
            return includes
```

2. 接下来实现对资源清单的中的资源解析

Decoder实现

```python
    @staticmethod
    def decode_to_manifest(
        # 定义一个函数参数列表，包含三个参数：raw_data, container, type
        # raw_data: bytes 类型，表示原始数据，以字节形式传递
        # container: JPResource 类型，表示资源容器，用于存储和管理资源
        # type: Literal["table", "media"] 类型，表示类型参数，只能是 "table" 或 "media" 中的一个
        # 函数返回值类型为 dict[str, object]，表示返回一个字典，键为字符串，值为任意对象
        raw_data: bytes, container: JPResource, type: Literal["table", "media"]
    ) -> dict[str, object]:
        # 使用JPCatalogDecoder的Reader方法读取原始数据
        data = JPCatalogDecoder.Reader(raw_data)

        # 初始化一个字典用于存储解析后的清单数据
        manifest: dict[str, object] = {}

        # 读取一个8位整数
        data.read(I8)
        # 读取一个32位整数，表示接下来有多少个条目（item）
        item_num = data.read(I32)

        # 遍历每个条目
        for _ in range(item_num):
            # 判断类型是否为"media"
            if type == "media":
                # 如果是媒体类型，调用私有方法__decode_media_manifest进行解析
                key, obj = JPCatalogDecoder.__decode_media_manifest(data, container)
            else:
                # 如果不是媒体类型，调用私有方法__decode_table_manifest进行解析
                key, obj = JPCatalogDecoder.__decode_table_manifest(data, container)
            # 将解析得到的键值对添加到清单字典中
            manifest[key] = obj
        # 返回解析后的清单字典
        return manifest

    @classmethod
    def __decode_media_manifest(
        cls, data: Reader, container: JPResource
    ) -> tuple[str, dict[str, object]]:
        # 读取一个32位整数
        data.read(I32)
        # 读取一个字符串，作为资源的键
        key = data.read_string()
        # 读取一个8位整数
        data.read(I8)
        # 再次读取一个32位整数
        data.read(I32)
        # 读取一个字符串，作为资源的路径
        path = data.read_string()
        # 再次读取一个32位整数
        data.read(I32)
        # 读取一个字符串，作为资源的文件名
        file_name = data.read_string()
        # 读取一个64位整数，表示资源的字节数
        bytes = data.read(I64)
        # 读取一个64位整数，表示资源的CRC校验码
        crc = data.read(I64)
        # 读取一个布尔值，表示资源是否为序章
        is_prologue = data.read(BOOL)
        # 读取一个布尔值，表示资源是否为分块下载
        is_split_download = data.read(BOOL)
        # 读取一个32位整数，表示资源的媒体类型
        media_type = data.read(I32)

        # 将路径中的反斜杠替换为正斜杠，以统一路径格式
        path = path.replace("\\", "/")
        # 将解析出的资源信息添加到容器中
        container.add_media_resource(
            key, path, file_name, media_type, bytes, crc, is_prologue, is_split_download
        )

        # 返回资源的键和详细信息字典
        return key, {
            "path": path,
            "file_name": file_name,
            "type": media_type,
            "bytes": bytes,
            "crc": crc,
            "is_prologue": is_prologue,
            "is_split_download": is_split_download,
        }

    @classmethod
    def __decode_table_manifest(
        cls, data: Reader, container: JPResource
    ) -> tuple[str, dict[str, object]]:
        # 读取一个32位整数
        data.read(I32)
        # 读取一个字符串，作为表的键
        key = data.read_string()
        # 读取一个8位整数
        data.read(I8)
        # 读取一个32位整数
        data.read(I32)
        # 读取一个字符串，作为表的名称
        name = data.read_string()
        # 读取一个64位整数，表示表的大小
        size = data.read(I64)
        # 读取一个64位整数，表示表的CRC校验值
        crc = data.read(I64)
        # 读取一个布尔值，表示表是否在构建中
        is_in_build = data.read(BOOL)
        # 读取一个布尔值，表示表是否已更改
        is_changed = data.read(BOOL)
        # 读取一个布尔值，表示表是否为序言
        is_prologue = data.read(BOOL)
        # 读取一个布尔值，表示表是否为分下载
        is_split_download = data.read(BOOL)
        # 读取表的包含信息
        includes = data.read_table_includes()

        # 将读取到的表资源信息添加到容器中
        container.add_table_resource(
            key,
            name,
            size,
            crc,
            is_in_build,
            is_changed,
            is_prologue,
            is_split_download,
            includes,
        )

        # 返回表的键和相关信息组成的元组
        return key, {
            "name": name,
            "size": size,
            "crc": crc,
            "is_in_build": is_in_build,
            "is_changed": is_changed,
            "is_prologue": is_prologue,
            "is_split_download": is_split_download,
            "includes": includes,
        }

```

至此，基本的解包流程已经完成，我不会详细去介绍关于Blue Archive的数据如何存储数据。