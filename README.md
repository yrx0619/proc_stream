# ProcStream 简单的流事件处理小框架

<br>

## 目录结构
	.
	├── README.md
	├──/conf			# 配置文件目录
	│   ├── config.json			# 流事件配置
	│   └── run.cfg				# 运行配置
	├──/plugin		# 外置插件，可用户自定义 
	│   ├──/nodes				# 用户自定义 节点 代码
	│   └──/triggers			# 用户自定义 触发器 代码
	├── run			# 启动脚本
	├──/stream		# 程序代码
	│   ├──/build_in_nodes		# 内置 节点 代码
	│   ├──/build_in_triggers	# 内置 触发器代码
	└──/var
	    ├──/log		# 日志目录
	    └──/run		# pidfile目录

<br>

## 处理逻辑
	 
	 Event --> Trigger --> handler_Node1 -> handler_Node2 -> output_Node3 --\
	             ^         \ ***************** Stream ***************** /   |
	             |                                                          |
	             |                                                          |
	             |                                                          |
	             \--------------------------Notice--------------------------/
    									
	
	ProcStream的逻辑单元分为 Event、Trigger、Stream、 Node
	Event（事件）：Trigger收集到的一个最小单元，一般为一条消息，或某个事件的发生
	Trigger（触发器）：负责Event的收集/产生，当一个事件到来/发生时，进行一次触发，并将触发的事件递交给Stream进行处理
	Stream（流）：负责按照既定流程处理Trigger递交的事件，Stream由若干个Node组成
	Node（节点）：Node是处理事件的最小单元，节点依附于Stream存在，不能单独存在。Node存在两种类型：output、handler，
		output-Node：Stream不关心Output—Node的结果返回，所以不会Output—Node的结果进行等待，对在处理流程中，当一个Event递交给output-Node后，会立即将Event传递给下一个Node进行处理。一般output-Node作为输出使用，例如数据入库，通知等。
		handler-Node：Stream关心handler-Node的返回，handler-Node可能对Event进行继续/更新/丢弃等动作。Stream会根据不同的动作采取不同的动作。当handler-Node的处理结果返回，且结果不为丢弃时，才会把Event继续向后传递。
		
<br>		

## 配置文件：
	
conf/run.cfg

	base_dir = '/path_to_your_project_dir'
	STDIN   = "/dev/null"
	STDOUT  = base_dir + '/var/log/err.log'
	STDERR  = base_dir + '/var/log/err.log'
	DEFAULT_STDERR = base_dir + '/var/log/err.log'
	
	PID_FILE = base_dir + '/var/run/sdp_id_engine.pid'
	PID_FILE_TIMEOUT = 3
	
	LOG_FILE = base_dir + '/var/log/run.log'
	LOG_LEVEL = "info"	# debug info warning error
	
	CONF_FILE_PATH = base_dir + '/conf/config.json' # 指定stream conf 位置
	
conf/config.json

	{
		// trigger 模板
	    "TriggerTemplate": { 
	        "testtrigger": {
	            "module": "stream.build_in_triggers.test_reader.TestReader"
	        }
	    },
	    
		// Node 模板
	    "NodeTemplate": {
	        "Build_in_SyncOutput" : {
		        // Node使用的Module地址
	            "module": "plugin.nodes.testnode.SyncOutput", 
	            // 线程池大小，默认为1
	            "pool_size": 1,	
	            // node所需参数
	            "args": { 
	                "msg": "hello world"
	            }
	        }	
	    },
	
	    "Streams": {
	    	// 定义流
	    	"TestStream": {
	           "trigger": "testtrigger",
	           "stream": ["Build_in_SyncOutput"]
	       }，
	       // 当然可以定义多条流
	      	"TestStream": {
	           "trigger": "testtrigger",
	           "stream": ["Build_in_SyncOutput"]
	       }
	    }
	}

<br>

## 如何启动

	./run start|stop|restart [options]
	Options
	  -f            Run as a foreground process, not a daemon.
	  -c            Set configuration file path.
	  -h            Show help.	./run	

<br>

## 先试一下？
	框架中已经有集成好的Test的配置，也有集成好的示例代码。
	为何不直接./run -f跑起来看下效果呢？
	

	
	
	
	
	    
	    
	
