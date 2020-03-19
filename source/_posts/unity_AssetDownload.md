---
title: Unity 并行下载方案
date: 2020-03-19 15:06:42
author: kongwz
tags:
  - hexo
categories:
  - Unity
comments: true

最近工作调整从游戏开发部门，调岗到平台支持部门，工作内容自然也有变动。现在主要工作内容是将游戏内所有收益活动拿出来做一个SDK，类似腾讯[潘多拉](https://gcloud.qq.com/increment/pandora),主要功能是 集成游戏内所有收益活动，后续公司的游戏只需要接入这个SDK就可以控制，所有收益活活动，其中也包括数据的采集分析等，对于项目来说好处还是很明显的，因为这块单独拿出来做，从开发 到数据分析都不需要开发组去做了，而且可以根据采集的数据进行道具的投放。

很久没写过东西了，主要是因为懒……最近情绪一直不是很高，又不知道怎么解决，希望疫情快点过去吧

---

进入正题吧

项目采用Slua +C# 除了view层的代码，基本还都是用C#来写，然后导出Lua接口供Lua调用。 最近写了一个并行下载的模块，就记录一下吧，还是希望自己能够坚持写下去

---
> 需求

	
- 根据提供的下载地址和保存地址，进行下载文件并保存
- 根据服务器提供的MD5码校验下载的文件是否正确
- 根据MD5码和保存路径判断本地是否有当前文件
- 设置下载超时功能，超时后判定当前下载失败
- 单个文件下载次数设置，每个文件下载三次均失败则不再下载
- 提供每个文件下载进度返回，下载完成返回
- 并行下载功能，目前设置是三个并行下载

---

> 思路

1. 需要参数：下载地址、保存路径、MD5、下载进度回调函数、下载完成回调函数。
2. 因为每个下载进度都要进行返回，所以写一个内部类AssetDLItem，储存当前文件的下载信息，比如下载地址、保存路径、和回调函数等。
3. 创建两个List<AssetDLItem>列表，一个用来保存所有要下载的资源m_AssetDLList，另一个保存当前正在下载的资源m_TempParallelDownloadList，且当开始下载的时候从m_AssetDLList移除 并添加进m_TempParallelDownloadList列表中，下载完成或者下载三次均失败则从m_TempParallelDownloadList中移除。
4. 根据文件下载状态和下载次数判断当前文件是否要继续下载
5. 在Update中调用下载函数，并判断两个列表中下载文件的状态执行下载还是等待。
6. 下载完成后保存文件并校验MD5并返回下载完成回调函数


---

> 代码


```bash

public delegate void AssetDownloadProgressBack(float _progress);
    public class AssetDownloadManager : MonoBehaviour
    {

        #region  对接 lua 接口
        public static AssetDownloadManager Create() {
            GameObject obj = new GameObject();
            DontDestroyOnLoad(obj);
            obj.name = "AssetDownloadManager";
            return obj.AddComponent<AssetDownloadManager>();
        }
        public LuaCallback OnFinishLuaCallback = new LuaCallback();

        public static void Remove(AssetDownloadManager downLoad)
        {
            Destroy(downLoad.gameObject);
        }

        public static void ExtractFile(string zipFile, string extractDir)
        {
            try
            {
                if (Directory.Exists(extractDir))
                {
                    Directory.Delete(extractDir, true);
                }
                Directory.CreateDirectory(extractDir);
                ZipFile.ExtractToDirectory(zipFile, extractDir);
            }
            catch (Exception e)
            {
                Loger.LogError(e.StackTrace);
            }
        }

        public void DownloadAsset(string downloadPath ,string savePath , string _md5 , AssetDownloadProgressBack _progressBack = null , Action<bool> _finishCallBack = null) {
            RequestLoad(downloadPath, savePath, _md5 , _progressBack, _finishCallBack);
        }
        #endregion

        private class AssetDLItem
        {
            public UnityWebRequest cache_obj;
            public string url;		
            public string savePath;
            public float unload_delay;	
            public int retry_count;		
            public bool is_ready;
            public string md5;
            public AssetDownloadProgressBack callBack;
            public Action<bool> finishBack;
            public AssetDLItem( string filePath , string savePath , string _md5 , AssetDownloadProgressBack callBack = null , Action<bool> finishCallBack = null) {
                url = filePath;
                retry_count = 1;
                cache_obj = null;
                is_ready = false;
                md5 = _md5;
                finishBack = finishCallBack;
                this.callBack = callBack;
                this.savePath = savePath;
            }
        }
        public string _ServerURL = "";
        private static AssetDownloadManager _instance = null;
        private List<AssetDLItem> m_AssetDLList = new List<AssetDLItem>();
        private State m_state;
        public const int AUTO_RETRY_COUNT = 3;
        public const int parallel_downloadNum = 2;
        private List<AssetDLItem> m_TempParallelDownloadList = new List<AssetDLItem>();
        private enum State
        {
            WAIT = 0,	
            INIT,		
            SETUP,		
            RUN,		
            ERROR,		
            EXIT,		

            NUM
        }



        private void Awake()
        {
            m_state = State.RUN;
        }

        public void RequestLoad(string path, string savePath, string _md5 , AssetDownloadProgressBack callBack = null , Action<bool> _finishCallBack = null) {
            if (string.IsNullOrEmpty(path) || string.IsNullOrEmpty(savePath)) {
                return;
            }
            if (File.Exists(savePath ))
            {
                Loger.Log(" 下载 文件已经存在  " + savePath);
                if (GetMd5(savePath) == _md5 || _md5==null)
                {
                    Loger.Log(" 要下载的文件 MD5 与本地文件相同  无需下载" + savePath);
                    //本地文件 和要下载的文件 相同  无需下载  直接 调用 下载完成 函数
                    _finishCallBack?.Invoke(true);
                    return;
                }
                File.Delete(savePath);
                return;
            }
            ///检测 是否有相同文件 正在下载列表
            if (m_AssetDLList != null)
            {
                foreach (var tmp in m_AssetDLList)
                {
                    if (tmp.url == path && tmp.savePath == savePath)
                    {
                        Debug.Log("要下载的文件 已在下载列表  无需再次下载 " + path);
                        return;
                    }
                }
            }
            if (m_TempParallelDownloadList != null)
            {
                foreach (var tmp in m_TempParallelDownloadList)
                {
                    if (tmp.url == path && tmp.savePath == savePath)
                    {
                        Debug.Log("要下载的文件 已在下载列表  无需再次下载 1111  " + path);
                        return;
                    }
                }
            }
            AssetDLItem item = new AssetDLItem(path , savePath , _md5 , callBack , _finishCallBack);
            m_AssetDLList.Add(item);
        }

        public void Update()
        {
            switch (m_state) {
                case State.WAIT:
                    break;
                case State.INIT:

                    break;
                case State.SETUP:

                    break;
                case State.RUN:
                    if (m_AssetDLList.Count > 0 || m_TempParallelDownloadList.Count > 0) {
                        //执行下载、
                        RunDownload();
                    }
                    break;
                case State.ERROR:

                    break;
            }
        }

        private void RunDownload() {

            if (m_TempParallelDownloadList.Count <= parallel_downloadNum)
            {
                //当前 正在下载数量 没有到达上限 添加新的 下载流程
                if (m_AssetDLList.Count > 0) {
                    //Loger.LogError("开始 下载 文件  " + m_AssetDLList[0].savePath);
                    m_TempParallelDownloadList.Add(m_AssetDLList[0]);
                    m_AssetDLList.RemoveAt(0); //删除 已经在下载中的 数据
                }
            }
            if (m_TempParallelDownloadList.Count > 0)
            {
                //有 正在下载中的数据 
                for (int i = 0; i < m_TempParallelDownloadList.Count; i++)
                {
                    if (m_TempParallelDownloadList[i].cache_obj != null)
                    {
                        if (m_TempParallelDownloadList[i].is_ready)
                        {
                            //Loger.LogError("Asset Download Finish " + m_TempParallelDownloadList[i].savePath);
                            m_TempParallelDownloadList.RemoveAt(i);
                            break;
                        }
                    }
                    else
                    {
                        if (m_TempParallelDownloadList[i].retry_count > AUTO_RETRY_COUNT)
                        {
                            m_TempParallelDownloadList[i].retry_count = 1;
                            Loger.LogError("资源下载3次失败 " + m_TempParallelDownloadList[i].savePath);
                            //下载三次 失败或者 下载文件 md5 码不匹配
                            //添加 下载失败 弹窗提示  并提示是否重新下载
                            m_TempParallelDownloadList[i].finishBack?.Invoke(false);
                            m_TempParallelDownloadList.RemoveAt(i);
                            break;
                        }
                        else
                        {
                            //Loger.Log("下载资源 准备 " + m_TempParallelDownloadList[i].savePath + "   下载 次数 " + m_TempParallelDownloadList[i].retry_count);
                            StartDownload(m_TempParallelDownloadList[i]);
                        }
                    }
                }
            }
        }

        private void StartDownload(AssetDLItem item) {
            ++item.retry_count;
            //验证 下载文件是否正确                       
            StartCoroutine(DownloadAsset(item));
        }

        private IEnumerator<object> DownloadAsset(AssetDLItem item)
        {
            const float WAIT_TIME = 0.125f;
            const float TIMEOUT_TIME = 30.0f;
            float pre_progress = 0.0f;
            float timeout = 0.0f;
            bool is_error = false;
            UnityWebRequest m_webRequest = UnityWebRequest.Get(item.url);
            item.cache_obj = m_webRequest;
            m_webRequest.SendWebRequest();

            while (!m_webRequest.isDone) {

                yield return new WaitForSeconds(WAIT_TIME);
                Loger.Log(m_webRequest.downloadProgress);
                item.callBack?.Invoke(m_webRequest.downloadProgress);
                //Loger.Log("正在下载 文件  " + item.savePath);
                if (pre_progress == m_webRequest.downloadProgress)
                {
                    timeout += WAIT_TIME;
                }
                else {
                    timeout = 0.0f;
                    pre_progress = m_webRequest.downloadProgress;
                }
                if (timeout >= TIMEOUT_TIME) {
                    Loger.LogError("链接超时  无响应");
                    is_error = true;
                    break;
                }
            }

            if (m_webRequest.isNetworkError || m_webRequest.isHttpError || is_error)
            {
                Loger.LogError("AssetDownload Error " + m_webRequest.error);
                is_error = true;
                item.cache_obj = null;
            }
            else {
                //Loger.Log("下载 没有出错 " + m_webRequest.downloadProgress);
                if (m_webRequest.isDone) {
                    using (FileStream fs = File.OpenWrite(item.savePath ))
                    {
                        fs.Write(m_webRequest.downloadHandler.data, 0, m_webRequest.downloadHandler.data.Length);
                    }
                    if (GetMd5(item.savePath) == item.md5 || item.md5 == null)
                    {
                        //Loger.LogError("下载完成  保存到了  " + item.savePath);
                        item.finishBack?.Invoke(true);
                        item.is_ready = true;
                    }
                    else {
                        // 服务器 储存文件 和 要下载的 文件 MD5 码不匹配
                        Loger.LogError("下载失败 服务器文件md5码 不匹配  即将再次下载 " + item.savePath);
                        item.cache_obj = null;
                        File.Delete(item.savePath);
                    }
                }
               
            }
        }

        public static string GetMd5(string fileName)
        {
            byte[] newBuffer;
            using (FileStream fs = File.OpenRead(fileName))
            {
                newBuffer = new MD5CryptoServiceProvider().ComputeHash(fs);
                fs.Close();
            }
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < newBuffer.Length; i++)
            {
                sb.Append(newBuffer[i].ToString("x2"));
            }

            string result = sb.ToString();
            // Loger.LogError("计算 的 md5 是 = " + result + "    计算文件 =   " + fileName);
            return result;
        }

      
    }
```


---

> 最后

其实很简单，上面就是全部代码了，最后思考一下，如果客户端本地保存了一张本地资源的表，然后根据本地资源表 进行和服务器的表比较，然后确定哪些资源需要下载，然后执行下载，那么就需要写一个 资源表的读取，和服务器表的比较，其实也比较简单，我写了一部分后来用不到 就注释了，因为没有测试就不贴出来了。

