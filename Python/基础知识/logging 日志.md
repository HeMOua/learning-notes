# logging 日志
## 输出格式
+ `%(asctime)s - %(levelname)s - %(message)s`：普通
+ `%(asctime)s - %(name)s - %(levelname)s - %(message)s`
+ `%(asctime)s %(levelname)s [%(filename)s,%(lineno)d] - %(message)s`：包含文件名与行号

## # 不同颜色日志输出
```python
def get_logger(name: str, level: int = logging.INFO):  
    logger = logging.getLogger(name)  
    logger.setLevel(level)  
    # 创建控制台日志处理器  
    console_handler = logging.StreamHandler()  
    console_handler.setLevel(level)  
    # 定义颜色输出格式  
    color_formatter = colorlog.ColoredFormatter(  
        '%(log_color)s%(levelname)s: %(message)s',  
        log_colors={  
            'DEBUG': 'cyan',  
            'INFO': 'green',  
            'WARNING': 'yellow',  
            'ERROR': 'red',  
            'CRITICAL': 'red,bg_white',  
        }  
    )  
    # 将颜色输出格式添加到控制台日志处理器  
    console_handler.setFormatter(color_formatter)  
    # 移除默认的handler  
    for handler in logger.handlers:  
        logger.removeHandler(handler)  
    # 将控制台日志处理器添加到logger对象  
    logger.addHandler(console_handler)  
    return logger
```
