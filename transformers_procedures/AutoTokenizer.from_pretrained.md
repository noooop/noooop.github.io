# 



# 加载 Tokenizer
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
     
     
    
```

