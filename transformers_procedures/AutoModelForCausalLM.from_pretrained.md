# qwen1.5 单步跟踪

# 加载模型
```
model.py 20行
    model = AutoModelForCausalLM.from_pretrained(
        
        进入
        transformers\models\auto\auto_factory.py 443行
        def from_pretrained(cls, pretrained_model_name_or_path, *model_args, **kwargs):

        运行至 
        transformers\models\auto\auto_factory.py 482行
        resolved_config_file = cached_file(
            
            进入 
            transformers\utils\hub.py 266行
            def cached_file(
            
            运行至
            transformers\utils\hub.py 398行
            cache_dir = TRANSFORMERS_CACHE
            这个 TRANSFORMERS_CACHE 是怎么来的呢 见 [HF_HOME 与 TRANSFORMERS_CACHE 位置]
            
            运行至
            transformers\utils\hub.py 397行
            # Load from URL or cache if already cached
            resolved_file = hf_hub_download(
            
            运行至
            transformers\utils\hub.py 465行
            return resolved_file
            
                这里得到了本地缓存位置
                resolved_file
                'D:/.cache/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\config.json'
            
        运行至
        transformers\models\auto\auto_factory.py 521行
        config, kwargs = AutoConfig.from_pretrained(
            
            进入
            transformers\models\auto\configuration_auto.py 1012行
            def from_pretrained(cls, pretrained_model_name_or_path, **kwargs)
            
            运行至
            transformers\models\auto\configuration_auto.py 1111行
            config_dict, unused_kwargs = PretrainedConfig.get_config_dict(pretrained_model_name_or_path, **kwargs)
            获取 config_dict 见 [get_config_dict]

            运行至  
            transformers\models\auto\configuration_auto.py 1128行      
            config_class = CONFIG_MAPPING[config_dict["model_type"]]
            其中 
                config_dict["model_type"] 是'qwen2' (config_dict 内容见[get_config_dict])
                CONFIG_MAPPING 字典的key len(CONFIG_MAPPING.keys()) 输出是234
                'qwen2' 从 CONFIG_MAPPING 里查出来是 transformers.models.qwen2.configuration_qwen2.Qwen2Config
                所以 CONFIG_MAPPING 是一个 model_type 到 模型config类的字典，transformers内置有234种模型
                这个 CONFIG_MAPPING 怎么构造的就不跟了，不是重点
            
            运行至
            transformers\models\auto\configuration_auto.py 1135行 
            return config_class.from_dict(config_dict, **unused_kwargs)
            
                进入
                transformers\configuration_utils.py 737行 
                def from_dict(cls, config_dict: Dict[str, Any], **kwargs) -> "PretrainedConfig":
            
                    补充一下
                    transformers\configuration_utils.py 49行 
                    class PretrainedConfig(PushToHubMixin):
                    
                    transformers\models\qwen2\configuration_qwen2.py 28行 
                    class Qwen2Config(PretrainedConfig):
                    
                    Qwen2Config继承自PretrainedConfig，所以运行 config_class.from_dict 进入到 configuration_utils.py 737行 
            
                
        
```


## get_config_dict
```              
进入
transformers\configuration_utils.py 614行
def get_config_dict(

运行至
transformers\configuration_utils.py 633行
config_dict, kwargs = cls._get_config_dict(pretrained_model_name_or_path, **kwargs)

    进入
    transformers\configuration_utils.py 647行
    def _get_config_dict(
    
    运行至 
    transformers\configuration_utils.py 688行
    resolved_config_file = cached_file(
    同上面的流程 本地缓存位置
    resolved_config_file
    'D:/.cache/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\config.json'
    
    运行至
    transformers\configuration_utils.py 717行
    # Load config dict
    config_dict = cls._dict_from_json_file(resolved_config_file)
            
            进入
            transformers\configuration_utils.py 814行
            def _dict_from_json_file(cls, json_file: Union[str, os.PathLike]):
                with open(json_file, "r", encoding="utf-8") as reader:
                    text = reader.read()
                return json.loads(text)
            读 config.json 得到 config_dict
    
    这个 config_dict 信息还是比较多
    config_dict
    {'architectures': ['Qwen2ForCausalLM'],
     'attention_dropout': 0.0,
     'bos_token_id': 151643,
     'eos_token_id': 151645,
     'hidden_act': 'silu',
     'hidden_size': 1024,
     'initializer_range': 0.02,
     'intermediate_size': 2816,
     'max_position_embeddings': 32768,
     'max_window_layers': 21,
     'model_type': 'qwen2',
     'num_attention_heads': 16,
     'num_hidden_layers': 24,
     'num_key_value_heads': 16,
     'rms_norm_eps': 1e-06,
     'rope_theta': 1000000.0,
     'sliding_window': 32768,
     'tie_word_embeddings': True,
     'torch_dtype': 'bfloat16',
     'transformers_version': '4.37.0',
     'use_cache': True,
     'use_sliding_window': False,
     'vocab_size': 151936}
     
    运行至
    transformers\configuration_utils.py 728行
    logger.info(f"loading configuration file {configuration_file} from cache at {resolved_config_file}")
    'loading configuration file config.json from cache at D:/.cache/hub\\models--Qwen--Qwen1.5-0.5B-Chat\\snapshots\\f82bd3692de0283f4a4b31e06d164dd8467fb52e\\config.json'
```   