# cached_file
```
cached_file 
返回缓存文件目录, 如果没有缓存，下载并缓存到硬盘上

函数输入 
模型名 pretrained_model_name_or_path
'Qwen/Qwen1.5-0.5B-Chat'

文件名 TOKENIZER_CONFIG_FILE
'tokenizer_config.json'

如果加载 tokenizer, TOKENIZER_CONFIG_FILE == 'tokenizer_config.json'
如果加载 model, CONFIG_NAME == 'config.json'

resolved_config_file = cached_file(
    pretrained_model_name_or_path,
    TOKENIZER_CONFIG_FILE
    ...其他参数为默认)

输出 resolved_config_file
'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer_config.json'
```
## 运行流程
```
transformers\utils\hub.py 266行
def cached_file(

运行至  
transformers\utils\hub.py 398行
cache_dir = TRANSFORMERS_CACHE
这个 TRANSFORMERS_CACHE 是怎么来的呢 见 [HF_HOME 与 TRANSFORMERS_CACHE 位置]
默认 TRANSFORMERS_CACHE == "~/.cache/huggingface/hub/"
这里 TRANSFORMERS_CACHE == 'D:/.cache/huggingface/hub'

运行至 调用hf_hub_download
transformers\utils\hub.py 397行
# Load from URL or cache if already cached
resolved_file = hf_hub_download(
    
    进入
    huggingface_hub\file_download.py 1014行
    def hf_hub_download(
    
    运行至 拿到repo_type
    huggingface_hub\file_download.py 1219行
    repo_type = "model"
    
    运行至 拼storage_folder目录
    huggingface_hub\file_download.py 1223行
    storage_folder = os.path.join(cache_dir, repo_folder_name(repo_id=repo_id, repo_type=repo_type))
    
        进入 
        huggingface_hub\file_download.py 977行
        def repo_folder_name(*, repo_id: str, repo_type: str) -> str:
        这里拼子目录名 'models--Qwen--Qwen1.5-0.5B-Chat'
    
    出来把cache_dir拼上
    storage_folder == 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat'
    
    运行至  
    huggingface_hub\file_download.py 1226行
    relative_filename = os.path.join(*filename.split("/"))
    如果加载 tokenizer, relative_filename == 'tokenizer_config.json'
    如果加载 model, relative_filename == 'config.json'
    
    运行至 拼url
    huggingface_hub\file_download.py 1243行
    url = hf_hub_url(repo_id, filename, repo_type=repo_type, revision=revision, endpoint=endpoint)
    url == 'https://hf-mirror.com/Qwen/Qwen1.5-0.5B-Chat/resolve/main/tokenizer_config.json'
    这里用了 hf-mirror.com 代理
    
    运行至 拿到url_to_download
    huggingface_hub\file_download.py 1253行
    url_to_download = url
    
    运行至 获取metadata 对url使用HEAD方法
    huggingface_hub\file_download.py 1261行
    metadata = get_hf_file_metadata(
    
        使用request库， method="HEAD"
        commit_hash、etag、location、size 都在返回的headers里
        
        return HfFileMetadata(
            commit_hash=r.headers.get(HUGGINGFACE_HEADER_X_REPO_COMMIT),
            # We favor a custom header indicating the etag of the linked resource, and
            # we fallback to the regular etag header.
            etag=_normalize_etag(r.headers.get(HUGGINGFACE_HEADER_X_LINKED_ETAG) or r.headers.get("ETag")),
            # Either from response headers (if redirected) or defaults to request url
            # Do not use directly `url`, as `_request_wrapper` might have followed relative
            # redirects.
            location=r.headers.get("Location") or r.request.url,  # type: ignore
            size=_int_or_none(r.headers.get(HUGGINGFACE_HEADER_X_LINKED_SIZE) or r.headers.get("Content-Length")),
        )
    
    metadata
    HfFileMetadata(commit_hash='f82bd3692de0283f4a4b31e06d164dd8467fb52e', etag='b1797ada900fbcebfe7388cefbf28e8e5b0cfe21', location='https://hf-mirror.com/Qwen/Qwen1.5-0.5B-Chat/resolve/main/tokenizer_config.json', size=1286)

    运行至 拿到commit_hash
    huggingface_hub\file_download.py 1281行
    commit_hash = metadata.commit_hash
    commit_hash
    'f82bd3692de0283f4a4b31e06d164dd8467fb52e'
    
    运行至 拿到etag
    huggingface_hub\file_download.py 1290行
    etag = metadata.etag
    etag
    'b1797ada900fbcebfe7388cefbf28e8e5b0cfe21'
    
    运行至 拿到expected_size
    huggingface_hub\file_download.py 1299行
    # Expected (uncompressed) size
    expected_size = metadata.size
    expected_size
    1286
    
    运行至 拼缓存文件目录 
    huggingface_hub\file_download.py 1290行
    blob_path = os.path.join(storage_folder, "blobs", etag)
    blob_path 
    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\blobs\\b1797ada900fbcebfe7388cefbf28e8e5b0cfe21'
    后面的那一串hash是etag
    
    运行至 创建缓存目录
    huggingface_hub\file_download.py 1418行
    os.makedirs(os.path.dirname(blob_path), exist_ok=True)
    
    运行至 
    判断软链接目录存不存在
    huggingface_hub\file_download.py 1425行
    if os.path.exists(pointer_path) and not force_download:
        ...
        return pointer_path
    
    判断 blob 目录存不存在， huggingface是很喜欢创建软链 
    huggingface_hub\file_download.py 1430行
    if os.path.exists(blob_path) and not force_download:
        ...
        return pointer_path
    
    
    运行至 拼下载锁文件目录
    huggingface_hub\file_download.py 1438行
    # Prevent parallel downloads of the same file with a lock.
    # etag could be duplicated across repos,
    lock_path = os.path.join(locks_dir, repo_folder_name(repo_id=repo_id, repo_type=repo_type), f"{etag}.lock")
    lock_path
    'D:/.cache/huggingface/hub\\.locks\\models--Qwen--Qwen1.5-0.5B-Chat\\b1797ada900fbcebfe7388cefbf28e8e5b0cfe21.lock'
    
    运行至 创建下载锁
    huggingface_hub\file_download.py 1450行
    Path(lock_path).parent.mkdir(parents=True, exist_ok=True)
    with WeakFileLock(lock_path):
        
        进入
        huggingface_hub\utils\_fixes.py 80行
        @contextlib.contextmanager
        def WeakFileLock(lock_file: Union[str, Path]) -> Generator[BaseFileLock, None, None]:
            lock = FileLock(lock_file)
            lock.acquire()
            ...
        
        FileLock 实现使用的这个库
        pip install filelock   
        https://pypi.org/project/filelock/ 
    
    运行至 创建临时文件
    huggingface_hub\file_download.py 1473行
    temp_file_manager = partial(  # type: ignore
        tempfile.NamedTemporaryFile, mode="wb", dir=cache_dir, delete=False
    )
    
    运行至 使用临时文件
    huggingface_hub\file_download.py 1480行
    # Download to temporary file, then copy to cache dir once finished.
    # Otherwise you get corrupt cache entries if the download gets interrupted.
    with temp_file_manager() as temp_file:
        ....
        
    运行至 下载文件到临时文件
    huggingface_hub\file_download.py 1492行
    http_get(
        默认使用 request
        HF_HUB_ENABLE_HF_TRANSFER=1 使用 hf_transfer
    
    运行至 将临时文件重命名到正式文件，并建立软链接 huggingface是很喜欢创建软链 
    huggingface_hub\file_download.py 1502行
    if local_dir is None:
        logger.debug(f"Storing {url} in cache at {blob_path}")
        _chmod_and_replace(temp_file.name, blob_path)
        _create_symlink(blob_path, pointer_path, new_blob=True)
    temp_file.name
    'D:\\.cache\\huggingface\\hub\\tmpe2cp3599'
    blob_path
    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\blobs\\b1797ada900fbcebfe7388cefbf28e8e5b0cfe21'
    
    运行至 返回 huggingface是很喜欢创建软链
    huggingface_hub\file_download.py 1528行
    return pointer_path
    pointer_path
    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer_config.json'
    
接上 resolved_file = hf_hub_download(
return resolved_file
```


# HF_HOME 与 TRANSFORMERS_CACHE 位置
```
这个 TRANSFORMERS_CACHE 是怎么来的呢
transformers\utils\hub.py 96行
PYTORCH_PRETRAINED_BERT_CACHE = os.getenv("PYTORCH_PRETRAINED_BERT_CACHE", constants.HF_HUB_CACHE)
PYTORCH_TRANSFORMERS_CACHE = os.getenv("PYTORCH_TRANSFORMERS_CACHE", PYTORCH_PRETRAINED_BERT_CACHE)
TRANSFORMERS_CACHE = os.getenv("TRANSFORMERS_CACHE", PYTORCH_TRANSFORMERS_CACHE)

这个 constants.HF_HUB_CACHE  是怎么来的呢
huggingface_hub\constants.py 114 行
HF_HUB_CACHE = os.getenv("HF_HUB_CACHE", HUGGINGFACE_HUB_CACHE)

huggingface_hub\constants.py 110 行
HUGGINGFACE_HUB_CACHE = os.getenv("HUGGINGFACE_HUB_CACHE", default_cache_path)

huggingface_hub\constants.py 106 行
default_cache_path = os.path.join(HF_HOME, "hub")

huggingface_hub\constants.py 97 行
default_home = os.path.join(os.path.expanduser("~"), ".cache")
HF_HOME = os.path.expanduser(
    os.getenv(
        "HF_HOME",
        os.path.join(os.getenv("XDG_CACHE_HOME", default_home), "huggingface"),
    )
)
hf_cache_home = HF_HOME  # for backward compatibility. TODO: remove this in 1.0.0

所以 HF_HOME 环境变量可以控制 huggingface 本地缓存位置 的位置
```