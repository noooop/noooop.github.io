# AutoTokenizer 加载 Tokenizer
1. 默认 use_fast
2. 通过 get_tokenizer_config， 内部使用了 cached_file, 得到了对应的tokenizer_config.json文件, 读出来就是tokenizer_config
3. tokenizer_config 里面的 'tokenizer_class': 'Qwen2Tokenizer', 通过 tokenizer_class_from_name， 内部跟TOKENIZER_MAPPING_NAMES 一个一个的硬比, 得到真正的 tokenizer_class
4. tokenizer_class.from_pretrained(pretrained_model_name_or_path, *inputs, **kwargs) 加载

```
model.py 20行
tokenizer = AutoTokenizer.from_pretrained(self.model_name)
    
    进入
    transformers\models\auto\tokenization_auto.py 718行 
    use_auth_token = kwargs.pop("use_auth_token", None)
    
        其中 AutoTokenizer 类定义于
        transformers\models\auto\tokenization_auto.py 627行 
        class AutoTokenizer:
        
        from_pretrained 方法定义于
        transformers\models\auto\tokenization_auto.py 643行 
        def from_pretrained(cls, pretrained_model_name_or_path, *inputs, **kwargs):

    运行至 
    transformers\models\auto\tokenization_auto.py 732行 
    use_fast = kwargs.pop("use_fast", True)
    默认 use_fast

    运行至
    transformers\models\auto\tokenization_auto.py 767行
    tokenizer_config = get_tokenizer_config(pretrained_model_name_or_path, **kwargs)

        进入 
        transformers\models\auto\tokenization_auto.py 517行 
        def get_tokenizer_config(

        运行至 拿到tokenizer_config.json缓存目录
        transformers\models\auto\tokenization_auto.py 600行 
        resolved_config_file = cached_file(
            其中参数 TOKENIZER_CONFIG_FILE == 'tokenizer_config.json'
            cached_file 函数运行流程见 [download_and_cache_file]
        resolved_config_file  
        'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer_config.json'
        
        运行至 读json文件
        transformers\models\auto\tokenization_auto.py 621行 
        with open(resolved_config_file, encoding="utf-8") as reader:
            result = json.load(reader)
        
        运行至 返回
        transformers\models\auto\tokenization_auto.py 624行 
        return result
    
    这个 tokenizer_config 信息还是比较多
    tokenizer_config
    {'add_prefix_space': False,
     'added_tokens_decoder': {'151643': {'content': '<|endoftext|>',
       'lstrip': False,
       'normalized': False,
       'rstrip': False,
       'single_word': False,
       'special': True},
      '151644': {'content': '<|im_start|>',
       'lstrip': False,
       'normalized': False,
       'rstrip': False,
       'single_word': False,
       'special': True},
      '151645': {'content': '<|im_end|>',
       'lstrip': False,
       'normalized': False,
       'rstrip': False,
       'single_word': False,
       'special': True}},
     'additional_special_tokens': ['<|im_start|>', '<|im_end|>'],
     'bos_token': None,
     'chat_template': "{% for message in messages %}{% if loop.first and messages[0]['role'] != 'system' %}{{ '<|im_start|>system\nYou are a helpful assistant<|im_end|>\n' }}{% endif %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}",
     'clean_up_tokenization_spaces': False,
     'eos_token': '<|im_end|>',
     'errors': 'replace',
     'model_max_length': 32768,
     'pad_token': '<|endoftext|>',
     'split_special_tokens': False,
     'tokenizer_class': 'Qwen2Tokenizer',
     'unk_token': None,
     '_commit_hash': 'f82bd3692de0283f4a4b31e06d164dd8467fb52e'}
     
     
     运行至 拿到 config_tokenizer_class
     transformers\models\auto\tokenization_auto.py 770行
     config_tokenizer_class = tokenizer_config.get("tokenizer_class")
     config_tokenizer_class
     'Qwen2Tokenizer'
     
     运行至
     transformers\models\auto\tokenization_auto.py 791行
     tokenizer_class_from_name(config_tokenizer_class)
     transformers.models.qwen2.tokenization_qwen2.Qwen2Tokenizer
     
     tokenizer_class_from_name(config_tokenizer_class + "Fast")
     transformers.models.qwen2.tokenization_qwen2_fast.Qwen2TokenizerFast
     
        进入 
        transformers\models\auto\tokenization_auto.py 489 行
        def tokenizer_class_from_name(class_name: str):
        
        运行至 
        transformers\models\auto\tokenization_auto.py 493 行
        for module_name, tokenizers in TOKENIZER_MAPPING_NAMES.items():
        
        # TOKENIZER_MAPPING_NAMES.keys()
        # odict_keys(['albert', 'align', 'bark', 'bart', 'barthez', 'bartpho', 'bert', 'bert-generation',
        # len(TOKENIZER_MAPPING_NAMES)
        # 161个 TOKENIZER 真的是一个一个模型试
        
            if class_name in tokenizers:
                # 为真的时候
                # class_name == "Qwen2Tokenizer"
                # tokenizers == ('Qwen2Tokenizer', 'Qwen2TokenizerFast')
            
                module_name = model_type_to_module_name(module_name)
                # module_name == 'qwen2'
                
                module = importlib.import_module(f".{module_name}", "transformers.models")
                # 到这里就导入了 transformers.models.qwen2
                
                return getattr(module, class_name)
                # 这里导入了对应的类
                # getattr(module, class_name)
                # transformers.models.qwen2.tokenization_qwen2.Qwen2Tokenizer
                
     运行至
     transformers\models\auto\tokenization_auto.py 815 行
     if use_fast and not config_tokenizer_class.endswith("Fast"):
         tokenizer_class_candidate = f"{config_tokenizer_class}Fast"
         tokenizer_class = tokenizer_class_from_name(tokenizer_class_candidate)
     # use_fast 默认为 True
     # 还是使用 上面的 tokenizer_class_from_name 得到 tokenizer_class
     # tokenizer_class_candidate
     # "Qwen2TokenizerFast"
     # tokenizer_class
     # transformers.models.qwen2.tokenization_qwen2_fast.Qwen2TokenizerFast
     
     运行至 
     transformers\models\auto\tokenization_auto.py 825 行
     return tokenizer_class.from_pretrained(pretrained_model_name_or_path, *inputs, **kwargs) 
     # tokenizer_class.from_pretrained 部分见 [tokenizer_class.from_pretrained]
返回              
```

# tokenizer_class.from_pretrained
1. 使用 cached_file，得到了对应的tokenizer_config.json文件, 读出来就是tokenizer_config
2. 将 Tokenizer 相关的文件，使用 cached_file 都下载缓存，并返回文件地址
3. cls._from_pretrained 才是 Tokenizer 真正的构造过程
4. 处理 '<|endoftext|>' '<|im_start|>' '<|im_end|>'三个特别字符
5. 构造 Qwen2TokenizerFast 第一层
6. Qwen2Tokenizer 使用的是 HuggingFace's *tokenizers* library   byte-level BPE
7. 构造 PreTrainedTokenizerFast， 第二层， fast_tokenizer = TokenizerFast.from_file(fast_tokenizer_file) 初始化 tokenizers库
8. 构造 PreTrainedTokenizerBase， 第三层， 设置model_max_length， chat_template（原来是 Jinja template）
9. 构造 SpecialTokensMixin， 第四层，设置模型共用的特殊Token， SPECIAL_TOKENS_ATTRIBUTES = ["bos_token","eos_token","unk_token","sep_token","pad_token","cls_token","mask_token","additional_special_tokens"]
10. 添加 自用的SpecialTokens， "<|endoftext|>", "<|im_start|>", "<|im_end|>"

```
进入 
transformers\tokenization_utils_base.py 1818行
def from_pretrained(

    补充一下
    transformers\tokenization_utils_base.py 1544行 
    class PreTrainedTokenizerBase(SpecialTokensMixin, PushToHubMixin):
    
    transformers\tokenization_utils_fast.py 78行 
    class PreTrainedTokenizerFast(PreTrainedTokenizerBase):
    
    transformers\models\qwen2\tokenization_qwen2.py 87行 
    class Qwen2Tokenizer(PreTrainedTokenizer):
    
    transformers\models\qwen2\tokenization_qwen2_fast.py 44行 
    class Qwen2TokenizerFast(PreTrainedTokenizerFast):
    
    Qwen2Tokenizer和Qwen2TokenizerFast都继承自PreTrainedTokenizerBase，所以运行 tokenizer_class.from_pretrained 进入到 tokenization_utils_base.py 1818行

运行至
transformers\tokenization_utils_base.py 1968行
fast_tokenizer_file = FULL_TOKENIZER_FILE
resolved_config_file = cached_file(  
# FULL_TOKENIZER_FILE
# 'tokenizer.json'
# resolved_config_file
# 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer_config.json'

运行至 
transformers\tokenization_utils_base.py 1988行
with open(resolved_config_file, encoding="utf-8") as reader:
    tokenizer_config = json.load(reader)
# tokenizer_config.json 又读一遍 

运行至 将 Tokenizer 相关的文件都下载缓存，并返回文件地址
transformers\tokenization_utils_base.py 1997行
for file_id, file_path in vocab_files.items():
    .....
    resolved_vocab_files[file_id] = cached_file(

vocab_files
{'vocab_file': 'vocab.json',
 'merges_file': 'merges.txt',
 'tokenizer_file': 'tokenizer.json',
 'added_tokens_file': 'added_tokens.json',
 'special_tokens_map_file': 'special_tokens_map.json',
 'tokenizer_config_file': 'tokenizer_config.json'}
 
resolved_vocab_files
{'vocab_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\vocab.json',
 'merges_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\merges.txt',
 'tokenizer_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json',
 'added_tokens_file': None,
 'special_tokens_map_file': None,
 'tokenizer_config_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer_config.json'}
 
运行至 
transformers\tokenization_utils_base.py 2048行
return cls._from_pretrained(        
    
    进入 真正的Tokenizer构造
    transformers\tokenization_utils_base.py 2063行
    def _from_pretrained(   

    运行至
    transformers\tokenization_utils_base.py 2098行
    tokenizer_config_file = resolved_vocab_files.pop("tokenizer_config_file", None)
    # resolved_vocab_files 少了 tokenizer_config_file

    运行至
    transformers\tokenization_utils_base.py 2100行
    with open(tokenizer_config_file, encoding="utf-8") as tokenizer_config_handle:
        init_kwargs = json.load(tokenizer_config_handle)

    tokenizer_config.json 又读一遍
    init_kwargs 就是上面的 tokenizer_config

    运行至
    transformers\tokenization_utils_base.py 2103行
    config_tokenizer_class = init_kwargs.get("tokenizer_class")
    config_tokenizer_class
    'Qwen2Tokenizer'

    运行至
    transformers\tokenization_utils_base.py 2195行
    added_tokens_file = resolved_vocab_files.pop("added_tokens_file", None)
    special_tokens_map_file = resolved_vocab_files.pop("special_tokens_map_file", None)
    # resolved_vocab_files 少了 added_tokens_file 和 special_tokens_map_file 都是none
    resolved_vocab_files 剩下
    {'vocab_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\vocab.json',
     'merges_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\merges.txt',
     'tokenizer_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json'}

    运行至
    transformers\tokenization_utils_base.py 2197行
    for args_name, file_path in resolved_vocab_files.items():
        if args_name not in init_kwargs:
         init_kwargs[args_name] = file_path

    运行至
    transformers\tokenization_utils_base.py 2200行
    tokenizer_file = resolved_vocab_files.pop("tokenizer_file", None)
    # resolved_vocab_files 少了 tokenizer_file
    tokenizer_file
    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json'

    运行至 处理 '<|endoftext|>' '<|im_start|>' '<|im_end|>'三个特别字符
    transformers\tokenization_utils_base.py 2206行
    #### Handle tokenizer serialization of added and special tokens
    added_tokens_decoder: Dict[int, AddedToken] = {}
    added_tokens_map: Dict[str, AddedToken] = {}
    if "added_tokens_decoder" in init_kwargs:
        for idx, token in init_kwargs["added_tokens_decoder"].items():
            ...
            token = AddedToken(**token)
            ...
            
            init_kwargs["added_tokens_decoder"]
            {'151643': {'content': '<|endoftext|>',
            'lstrip': False,
            'normalized': False,
            'rstrip': False,
            'single_word': False,
            'special': True},
            '151644': {'content': '<|im_start|>',
            'lstrip': False,
            'normalized': False,
            'rstrip': False,
            'single_word': False,
            'special': True},
            '151645': {'content': '<|im_end|>',
            'lstrip': False,
            'normalized': False,
            'rstrip': False,
            'single_word': False,
            'special': True}}

    运行至
    transformers\tokenization_utils_base.py 2287行
    tokenizer = cls(*init_inputs, **init_kwargs)
    init_inputs
    ()
    init_kwargs
    {'add_prefix_space': False,
    'added_tokens_decoder': {151643: AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                          151644: AddedToken("<|im_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                          151645: AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True)},
    'additional_special_tokens': ['<|im_start|>', '<|im_end|>'],
    'bos_token': None,
    'chat_template': "{% for message in messages %}{% if loop.first and messages[0]['role'] != 'system' %}{{ '<|im_start|>system\nYou are a helpful assistant<|im_end|>\n' }}{% endif %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}",
    'clean_up_tokenization_spaces': False,
    'eos_token': AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
    'errors': 'replace',
    'model_max_length': 32768,
    'pad_token': AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
    'split_special_tokens': False,
    'unk_token': None,
    'vocab_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\vocab.json',
    'merges_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\merges.txt',
    'tokenizer_file': 'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json',
    'name_or_path': 'Qwen/Qwen1.5-0.5B-Chat'}
     
        进入  这才真正进入Qwen2TokenizerFast的构造函数 第一层
        transformers\models\qwen2\tokenization_qwen2_fast.py 92行
        
            注意一下Qwen2TokenizerFast的文档字符串
            transformers\models\qwen2\tokenization_qwen2_fast.py 46行
                Construct a "fast" Qwen2 tokenizer (backed by HuggingFace's *tokenizers* library). Based on byte-level
                Byte-Pair-Encoding.
            
                Same with GPT2Tokenizer, this tokenizer has been trained to treat spaces like parts of the tokens so a word will
                be encoded differently whether it is at the beginning of the sentence (without space) or not:
            Qwen2Tokenizer 使用的是 HuggingFace's *tokenizers* library   byte-level BPE
            byte-level BPE 在 GPT2 第一次引入，变为事实上的标准。
        
        def __init__(
            
            运行至 添加框架需要的特别token
            transformers\models\qwen2\tokenization_qwen2_fast.py 108行
            unk_token="<|endoftext|>"
            bos_token=None
            eos_token="<|endoftext|>"
            pad_token="<|endoftext|>"
            
            运行至 
            transformers\models\qwen2\tokenization_qwen2_fast.py 129行
            super().__init__(

                进入  构造 PreTrainedTokenizerFast 第二层
                transformers\tokenization_utils_fast.py 94行
                def __init__(self, *args, **kwargs):
                
                    vocab_file
                    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\vocab.json'
                    merges_file
                    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\merges.txt'
                    tokenizer_file
                    'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json'
                    kwargs
                    {'add_prefix_space': False,
                    'added_tokens_decoder': {151643: AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                    151644: AddedToken("<|im_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                    151645: AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True)},
                    'additional_special_tokens': ['<|im_start|>', '<|im_end|>'],
                    'chat_template': "{% for message in messages %}{% if loop.first and messages[0]['role'] != 'system' %}{{ '<|im_start|>system\nYou are a helpful assistant<|im_end|>\n' }}{% endif %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}",
                    'clean_up_tokenization_spaces': False,
                    'errors': 'replace',
                    'model_max_length': 32768,
                    'split_special_tokens': False,
                    'name_or_path': 'Qwen/Qwen1.5-0.5B-Chat'}
                
                运行至 
                transformers\tokenization_utils_fast.py 97行
                fast_tokenizer_file = kwargs.pop("tokenizer_file", None)
                fast_tokenizer_file
                'D:/.cache/huggingface/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\tokenizer.json'
                
                运行至 
                transformers\tokenization_utils_fast.py 99行
                added_tokens_decoder = kwargs.pop("added_tokens_decoder", {})
                {151643: AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                 151644: AddedToken("<|im_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                 151645: AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True)}
                
                运行至 
                transformers\tokenization_utils_fast.py 111行
                fast_tokenizer = TokenizerFast.from_file(fast_tokenizer_file)
                    
                    底层是 huggingface tokenizers. huggingface真的是打通了底层逻辑， 创建了生态闭环
                    from tokenizers import Tokenizer as TokenizerFast
                    https://huggingface.co/docs/tokenizers/main/en/index
                
                运行至 
                transformers\tokenization_utils_fast.py 128行
                self._tokenizer = fast_tokenizer
                
                运行至 
                transformers\tokenization_utils_fast.py 128行
                super().__init__(**kwargs)
                
                    进入 构造 PreTrainedTokenizerBase 第三层
                    transformers\tokenization_utils_base.py
                    def __init__(self, **kwargs):
                        kwargs
                        {'unk_token': None,
                         'bos_token': None,
                         'eos_token': AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                         'pad_token': AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                         'add_prefix_space': False,
                         'additional_special_tokens': ['<|im_start|>', '<|im_end|>'],
                         'chat_template': "{% for message in messages %}{% if loop.first and messages[0]['role'] != 'system' %}{{ '<|im_start|>system\nYou are a helpful assistant<|im_end|>\n' }}{% endif %}{{'<|im_start|>' + message['role'] + '\n' + message['content'] + '<|im_end|>' + '\n'}}{% endfor %}{% if add_generation_prompt %}{{ '<|im_start|>assistant\n' }}{% endif %}",
                         'clean_up_tokenization_spaces': False,
                         'errors': 'replace',
                         'model_max_length': 32768,
                         'split_special_tokens': False,
                         'name_or_path': 'Qwen/Qwen1.5-0.5B-Chat'}
                     
                    运行至 
                    transformers\tokenization_utils_base.py 1572
                    model_max_length = kwargs.pop("model_max_length", kwargs.pop("max_len", None))
                    model_max_length
                    32768
                    
                    运行至 
                    transformers\tokenization_utils_base.py 1577
                    self.padding_side = kwargs.pop("padding_side", self.padding_side)
                    self.padding_side
                    'right'                            
                    
                    运行至 
                    transformers\tokenization_utils_base.py 1583
                    self.truncation_side = kwargs.pop("truncation_side", self.truncation_side)
                    self.truncation_side
                    'right'
                
                    运行至 原来是 Jinja template
                    transformers\tokenization_utils_base.py 1600
                    # Stores a Jinja template that formats chat histories into tokenizable strings
                    self.chat_template = kwargs.pop("chat_template", None)
                    
                    运行至 
                    transformers\tokenization_utils_base.py 1603
                    super().__init__(**kwargs)
                
                        进入 构造 SpecialTokensMixin 第四次
                        transformers\tokenization_utils_base.py  834行
                        def __init__(self, verbose=False, **kwargs):
                        kwargs
                        {'unk_token': None,
                         'bos_token': None,
                         'eos_token': AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                         'pad_token': AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                         'add_prefix_space': False,
                         'additional_special_tokens': ['<|im_start|>', '<|im_end|>'],
                         'errors': 'replace'}
                        
                                注意一下 SpecialTokensMixin 属性
                                transformers\tokenization_utils_base.py  823行
                                SPECIAL_TOKENS_ATTRIBUTES = [
                                    "bos_token",
                                    "eos_token",
                                    "unk_token",
                                    "sep_token",
                                    "pad_token",
                                    "cls_token",
                                    "mask_token",
                                    "additional_special_tokens",
                                ]
                                
                         运行至 把 SPECIAL_TOKENS setattr 
                         transformers\tokenization_utils_base.py  850行       
                         for key, value in kwargs.items():   
                            if key in self.SPECIAL_TOKENS_ATTRIBUTES:
                            .....
                                elif isinstance(value, (str, AddedToken)):
                                    setattr(self, key, value)
                
                
                返回 到这层
                
                运行至 添加 added_tokens_decoder
                transformers\tokenization_utils_fast.py 162行
                tokens_to_add = [
                    token
                    for index, token in sorted(added_tokens_decoder.items(), key=lambda x: x[0])
                    if token not in self.added_tokens_decoder
                ]
                encoder = list(self.added_tokens_encoder.keys()) + [str(token) for token in tokens_to_add]
                # if some of the special tokens are strings, we check if we don't already have a token
                tokens_to_add += [
                    token for token in self.all_special_tokens_extended if token not in encoder and token not in tokens_to_add
                ]  
                tokens_to_add
                [AddedToken("<|endoftext|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                 AddedToken("<|im_start|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True),
                 AddedToken("<|im_end|>", rstrip=False, lstrip=False, single_word=False, normalized=False, special=True)]              
                
                运行至 真调用添加   
                transformers\tokenization_utils_fast.py 172行                            
                if len(tokens_to_add) > 0:
                ....
                    if tokens:
                        self._add_tokens(tokens, special_tokens=is_last_special)
    返回 到这层
    
运行至
transformers\tokenization_utils_base.py 2299行
return tokenizer
``` 
