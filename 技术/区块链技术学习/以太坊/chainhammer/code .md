# code 
## 配置文件
	vim config.py
	
	#!/usr/bin/env python3
	"""
	@summary: settings
	@version: v58 (12/April/2019)
	@since:   8/May/2018
	@organization:
	@author:  https://github.com/drandreaskrueger
	@see:     https://github.com/drandreaskrueger/chainhammer for updates
	"""
	
	##########
	# Choices:
	
	# 启动eth客户端，最好保持相同端口 :8545 
	#  docker-compose.yml 文件, 查看 ../networks/quorum-configure.sh
	RPCaddress='http://localhost:8545'
	RPCaddress2='http://localhost:8545'
	
	# 使用它与 TestRPCProvider 进行单元测试
	# RPCaddress, RPCaddress2 = None, None
	# 或者这个用于实验运行或使用真实的以太坊节点进行单元测试：
	# RPCaddress, RPCaddress2 = 'http://localhost:22000', 'http://localhost:22001' # 使用两个不同的 Quorum 节点进行读写
	# RPCaddress, RPCaddress2 = 'http://localhost:22001', 'http://localhost:22002' # crux dockerized, 查看 https://github.com/blk-io/crux/blob/master/README.md#4-node-quorum-network-with-crux
	# RPCaddress, RPCaddress2 = 'http://localhost:8545', 'http://localhost:8545'  # orbita-center_parity-poa-playground; parity-deploy
	# RPCaddress, RPCaddress2 = 'http://localhost:8545', 'http://localhost:8546'  # javahippie_geth-dev; 使用两个不同的节点读写
	
	#  在 send.py 发送多少 tx
	# 现在可以使用  ./send.py 命令行工具参数设置
	# NUMBER_OF_TRANSACTIONS = 5000
	
	# 过时信息
	# 最初合约是使用 ./runscript.sh private-contract.js 手动部署的，wait-for-something-to-happen 由链移动与否触发（仅适用于 raft 和 instantseal）。
	# 现在已经过时了，因为我们总是可以简单地先部署我们自己的合约，请参阅 deploy.py
	
	# OLD:
	# 如果共识算法是 Quorum raft，那么 --> True
	# 现在可以自动化了 ... from clienttype import clientType
	# TODO: 用客户端类型 CONSENSUS 查询替换这个常量
	# RAFT=False
	# RAFT=True
	
	
	# 部署等待时间，对于 Quorum 客户端可能需要一分钟，
	# 那么默认的 timeout=120 可能太短了。 给更多时间：
	
	TIMEOUT_DEPLOY = 300
	
	## 通过 web3 或直接通过 RPC 提交交易
	
	ROUTE = "RPC"  # "web3" "RPC"
	
	# parity's 特质:
	# '时间锁仅在 --geth 兼容模式下支持.'
	# 详细阅读 https://github.com/drandreaskrueger/chainhammer/blob/master/parity.md#run-14 for why and how
	# 对于 either 的每笔交易使用 --geth 或 unlockAccount() , 然后设置为 True:
	
	PARITY_UNLOCK_EACH_TRANSACTION=False
	
	# 为了避免 parity 的 -geth 开关，可以在启动时解锁 parity，例如 
	# parity --unlock 0x39c6ad93dfb708143322d8bbf4c35734f6480249 --password /home/parity/password
	# 例 通过 ./parity-deploy.sh --config aura --nodes 3 手动编辑 docker-compose.yml 文件 services: --> host1: --> command: --> add this: (地址在 deployment/1/address.txt)
	#  --unlock 0x39c6ad93dfb708143322d8bbf4c35734f6480249 --password /home/parity/password
	
	PARITY_ALREADY_UNLOCKED=False
	
	
	#抱歉，但无论如何，节省时间的 RPC 优化毫无意义：
	if PARITY_UNLOCK_EACH_TRANSACTION and ROUTE=="RPC":
	    print ("Sorry, this parameter combination is not implemented:")
	    print ('PARITY_UNLOCK_EACH_TRANSACTION and ROUTE=="RPC"')
	    print ('change config.py, then restart.')
	    exit(0)
	
	
	# 为交易提供的气体 .set(x) 
	# 注意：必须不同于（即高于）成功交易中最终使用的气体； 因为在那些还没有 “transactionReceipt.status” 字段的客户端的情况下，差异被用作交易失败的标志
	GAS_FOR_SET_CALL = 90000
	
	# 仅适用于 Quorum ：
	# 将此设置为 privateFor-transactions 的公钥列表或设置为 None 用于公共交易
	PRIVATE_FOR = ["ROAZBWtSacxXQrOe3FGAqJDyJjFePR5ce4TSIzmJ0Bc="]
	PRIVATE_FOR = None
	
	# 合约  7nodes 例 (可能是 'SimpleStorage.sol'?)
	EXAMPLE_ABI = [{"constant":True,"inputs":[],"name":"storedData","outputs":[{"name":"","type":"uint256"}],"payable":False,"type":"function"},{"constant":False,"inputs":[{"name":"x","type":"uint256"}],"name":"set","outputs":[],"payable":False,"type":"function"},{"constant":True,"inputs":[],"name":"get","outputs":[{"name":"retVal","type":"uint256"}],"payable":False,"type":"function"},{"inputs":[{"name":"initVal","type":"uint256"}],"type":"constructor"}];
	
	EXAMPLE_BIN = "0x6060604052341561000f57600080fd5b604051602080610149833981016040528080519060200190919050505b806000819055505b505b610104806100456000396000f30060606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680632a1afcd914605157806360fe47b11460775780636d4ce63c146097575b600080fd5b3415605b57600080fd5b606160bd565b6040518082815260200191505060405180910390f35b3415608157600080fd5b6095600480803590602001909190505060c3565b005b341560a157600080fd5b60a760ce565b6040518082815260200191505060405180910390f35b60005481565b806000819055505b50565b6000805490505b905600a165627a7a72305820d5851baab720bba574474de3d09dbeaabc674a15f4dd93b974908476542c23f00029"
	
	# 合约文件:
	FILE_CONTRACT_SOURCE  = "contract.sol"
	FILE_CONTRACT_ABI     = "contract-abi.json"
	FILE_CONTRACT_ADDRESS = "contract-address.json"
	
	# 账户密码
	FILE_PASSPHRASE = "account-passphrase.txt"
	
	#上次实验数据
	FILE_LAST_EXPERIMENT = "last-experiment.json"
	
	# 如果为 True，则新写入的文件 FILE_LAST_EXPERIMENT 将停止 tps.py 中的循环
	AUTOSTOP_TPS = True
	
	# 在最后一笔交易被挖出后，在实验结束前再提供 10 个区块
	EMPTY_BLOCKS_AT_END = 10
	
	# 用于遍历所有块的 DB 文件
	DBFILE="allblocks.db"
	
	if __name__ == '__main__':
	    print ("Do not run this. Like you just did. Don't.")
	    
## 测试合约
	vim contract.sol

	// simplestorage contract
	// chainhammer v46
	
	pragma solidity ^0.4.21;
	
	contract simplestorage {
	  uint public storedData;
	
	  function set(uint x) {
	    storedData = x;        // uses ~26691 gas
	    
	    // try failing transactions:
	    // assert ( 1 == 0 );  // uses up all 90000 given gas
	    // revert();           // uses 41686 gas
	    // throw;              // same as revert(); 
	    // require ( 1 == 0 ); // uses 41714 gas
	  }
	
	  function get() constant returns (uint retVal) {
	    return storedData;
	  }
	}
## 客户端工具
	vim clienttools.py
	
	#!/usr/bin/env python3
	"""
	@summary: 与以太坊客户端节点对话的工具
	@version: v58 (12/April/2019)
	@since:   19/June/2018
	@organization: 
	@author:  https://github.com/drandreaskrueger
	@see: https://github.com/drandreaskrueger/chainhammer for updates
	"""
	
	
	################
	## Dependencies:
	
	import os
	from pprint import pprint
	
	try:
	    from web3 import Web3, HTTPProvider # pip3 install web3
	except:
	    print ("Dependencies unavailable. Start virtualenv first!")
	    exit()
	
	# 为导入扩展 sys.path：
	if __name__ == '__main__' and __package__ is None:
	    from os import sys, path
	    sys.path.append(path.dirname(path.dirname(path.abspath(__file__))))
	
	from hammer.config import RPCaddress 
	from hammer.config import FILE_PASSPHRASE, PARITY_UNLOCK_EACH_TRANSACTION, PARITY_ALREADY_UNLOCKED
	from hammer.clienttype import clientType
	
	################
	## Tools:
	
	
	def printVersions():
	    import sys
	    from web3 import __version__ as web3version 
	    from solc import get_solc_version
	    from testrpc import __version__ as ethtestrpcversion
	    
	    import pkg_resources
	    pysolcversion = pkg_resources.get_distribution("py-solc").version
	    
	    print ("versions: web3 %s, py-solc: %s, solc %s, testrpc %s, python %s" % (web3version, pysolcversion, get_solc_version(), ethtestrpcversion, sys.version.replace("\n", "")))
	
	
	################################################################################
	# 建立联系，并尽可能多地了解
	
	def start_web3connection(RPCaddress=None, account=None):
	    """
	    获取一个 web3 对象，并使其成为全局对象
	    """
	    global w3
	    if RPCaddress:
	        # HTTP provider 
	        # (TODO: also try whether IPC provider is faster, when quorum-outside-vagrant starts working)
	        w3 = Web3(HTTPProvider(RPCaddress, request_kwargs={'timeout': 120}))
	    else:
	        # w3 = Web3(Web3.EthereumTesterProvider()) #不工作！
	        w3 = Web3(Web3.TestRPCProvider()) 
	    
	    print ("web3 connection established, blockNumber =", w3.eth.blockNumber, end=", ")
	    print ("node version string = ", w3.version.node)
	   
	    accountname="chosen"
	    if not account:
	        w3.eth.defaultAccount = w3.eth.accounts[0] # set first account as sender
	        accountname="first"
	    print (accountname + " account of node is", w3.eth.defaultAccount, end=", ")
	    print ("balance is %s Ether" % w3.fromWei(w3.eth.getBalance(w3.eth.defaultAccount), "ether"))
	    
	    return w3
	
	
	def setGlobalVariables_clientType(w3):
	    """
	    设置全局变量。
	    """
	    global NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = clientType(w3)
	    
	    formatter="nodeName: %s, nodeType: %s, nodeVersion: %s, consensus: %s, network: %s, chainName: %s, chainId: %s" 
	    print (formatter % (NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID))
	    
	    return NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID # for when imported into other modules
	
	
	def if_poa_then_bugfix(w3, NODENAME, CHAINNAME, CONSENSUS):
	    """
	    quorum web3.py 问题的错误修复，请参阅
		https://github.com/ethereum/web3.py/issues/898#issuecomment-396701172
		和
		https://web3py.readthedocs.io/en/stable/middleware.html#geth-style-proof-of-authority

		实际上也出现在使用带有 PoA 的 dockerized 标准 geth 节点时
		https://github.com/javahippie/geth-dev (net_version='500')
	    """
	    if NODENAME == "Quorum" or CHAINNAME=='500' or CONSENSUS=='clique':
	        from web3.middleware import geth_poa_middleware
	        # inject the poa compatibility middleware to the innermost layer
	        w3.middleware_stack.inject(geth_poa_middleware, layer=0)
	
	
	# def web3connection(RPCaddress=RPCaddress, account=None):
	def web3connection(RPCaddress=None, account=None):    
	    """
	  	打印依赖版本，启动 web3 连接，识别客户端节点类型，如果 quorum 则错误修复
	    """
	    
	    printVersions()
	    
	    w3 = start_web3connection(RPCaddress=RPCaddress, account=account) 
	
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = setGlobalVariables_clientType(w3)
	
	    if_poa_then_bugfix(w3, NODENAME, CHAINNAME, CONSENSUS)
	    
	    chainInfos = NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    
	    return w3, chainInfos 
	
	
	################################################################################
	# 一般有用的工具
	
	
	def getBlockTransactionCount(w3, blockNumber):
	    """
	    testRPC does not provide this endpoint yet, so replicate its functionality:
	    """
	    block=w3.eth.getBlock(blockNumber)
	    # pprint (block)
	    return len(block["transactions"])
	    
	
	def correctPath(file):
	    """
	    这是 FILE_PASSPHRASE (="account-passphrase.txt") 的破解，用于修复仅在运行测试时出现的 FileNotFound 问题，因为 currentWorkDir 是“chainhammer”而不是“chainhammer/hammer”
		P.S.：如果有一致的解决方案，那么也修复这两个
		测试放入根文件夹的“contract-{abi,address}.json”
	    """
	    # print ("os.getcwd():", os.getcwd())
	    
	    if os.getcwd().split("/")[-1] != "hammer":
	         return os.path.join("hammer", file)
	    else:
	         return file
	     
	
	def unlockAccount(duration=3600, account=None):
	    """
	   解锁一次，然后保持打开状态，以后不要松开解锁时间
	    """
	    
	    if ("TestRPC" in w3.version.node) or (PARITY_ALREADY_UNLOCKED and ("Parity" in w3.version.node)):
	        return True # TestRPC does not need unlocking; or parity can be CLI-switch unlocked when starting
	    
	    if NODENAME=="Quorum":
	        if NETWORKID==1337:
	            passphrase="1234" # Azure Quorum testnet 1337 jtessera
	        else:
	            passphrase="" # Any other Quorum
	    else:
	        # print ("os.getcwd():", os.getcwd())
	        with open(correctPath(FILE_PASSPHRASE), "r") as f:
	            passphrase=f.read().strip()
	
	    if NODENAME=="Geth" and CONSENSUS=="clique" and NETWORKID==500:
	        passphrase="pass" # hardcoded in geth-dev/docker-compose.yml
	
	    # print ("passphrase:", passphrase)
	
	    if not account:
	        account = w3.eth.defaultAccount
	        # print (account)
	
	    if PARITY_UNLOCK_EACH_TRANSACTION:
	        answer = w3.personal.unlockAccount(account=account, 
	                                           passphrase=passphrase)
	    else:
	        if NODETYPE=="Parity": 
	            duration = w3.toHex(duration)
	        answer = w3.personal.unlockAccount(account=account, 
	                                           passphrase=passphrase,
	                                           duration=duration)
	    print ("unlocked:", answer)
	    return answer
	     
	
	
	if __name__ == '__main__':
	
	    # 示例如何调用它：
	    # answer = web3connection()
	    answer = web3connection(RPCaddress=RPCaddress, account=None)
	    
	    w3, chainInfos  = answer
	    
	    global NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = chainInfos
	    
	    # print (type(NETWORKID), NETWORKID)

## 部署接口
	#!/usr/bin/env python3
	"""
	@summary: 部署合约
	@version: v46 (03/January/2019)
	@since:   2/May/2018
	@organization: 
	@author:  https://github.com/drandreaskrueger
	@see:     https://github.com/drandreaskrueger/chainhammer for updates
	"""
	
	
	################
	## Dependencies:
	
	import sys, time, json
	from pprint import pprint
	
	import requests # pip3 install requests
	
	try:
	    from web3 import Web3, HTTPProvider # pip3 install web3
	    from solc import compile_source # pip install py-solc
	except:
	    print ("Dependencies unavailable. Start virtualenv first!")
	    exit()
	
	
	# 扩展导入路径：
	if __name__ == '__main__' and __package__ is None:
	    from os import sys, path
	    sys.path.append(path.dirname(path.dirname(path.abspath(__file__))))
	
	from hammer.config import RPCaddress, TIMEOUT_DEPLOY, PARITY_UNLOCK_EACH_TRANSACTION
	from hammer.config import FILE_CONTRACT_SOURCE, FILE_CONTRACT_ABI, FILE_CONTRACT_ADDRESS
	from hammer.config import GAS_FOR_SET_CALL
	
	from hammer.clienttools import web3connection, unlockAccount
	
	
	###############################################################################
	## 从部署示例
	## http://web3py.readthedocs.io/en/latest/examples.html#working-with-contracts
	## 当“最新”为 4.2.0 
	###############################################################################
	
	
	def compileContract(contract_source_file):
	    """
	   	读取文件，编译，返回合约名称和接口
	    """
	    with open(contract_source_file, "r") as f:
	        contract_source_code = f.read()
	    compiled_sol = compile_source(contract_source_code) # Compiled source code
	    assert(len(compiled_sol)==1) # assert source file has only one contract object
	    
	    contractName = list(compiled_sol.keys())[0] 
	    contract_interface = compiled_sol[contractName]
	    return contractName.replace("<stdin>:", ""), contract_interface 
	
	
	def deployContract(contract_interface, ifPrint=True, timeout=TIMEOUT_DEPLOY):
	    """
	    部署合约，等待接收，返回地址
	    """
	    before=time.time()
	    myContract = w3.eth.contract(abi=contract_interface['abi'], 
	                                 bytecode=contract_interface['bin'])
	
	    tx_hash = w3.toHex( myContract.constructor().transact() )
	    print ("tx_hash = ", tx_hash, "--> waiting for receipt (timeout=%d) ..." % timeout)
	    sys.stdout.flush()
	    tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash, timeout=timeout)
	    print ("Receipt arrived. Took %.1f seconds." % (time.time()-before))
	    
	    contractAddress = tx_receipt["contractAddress"]
	    if ifPrint:
	        line = "Deployed. gasUsed={gasUsed} contractAddress={contractAddress}"
	        print ( line.format(**tx_receipt) )
	    return contractAddress 
	
	    
	def contractObject(contractAddress, abi):
	    """
	    当给定链上地址和 ABI 时重新创建 myContract 对象
	    """
	    # 使用新部署的地址创建合约实例
	    
	    myContract = w3.eth.contract(address=contractAddress,
	                                 abi=abi)
	    return myContract
	    
	
	##########################
	## 额外的基本任务：
	##########################
	
	def saveToDisk(contractAddress, abi):
	    """
	    保存地址和 abi，用于其他脚本
	    """
	    json.dump({"address": contractAddress}, open(FILE_CONTRACT_ADDRESS, 'w'))
	    json.dump(abi, open(FILE_CONTRACT_ABI, 'w'))
	
	
	def loadFromDisk():
	    """
	    从之前运行的“contract_CompileDeploySave”加载地址和 abi
	    """
	    contractAddress = json.load(open(FILE_CONTRACT_ADDRESS, 'r'))
	    abi = json.load(open(FILE_CONTRACT_ABI, 'r'))
	    return contractAddress["address"], abi
	
	
	def contract_CompileDeploySave(contract_source_file):
	    """
	    编译、部署、保存
	    """
	    contractName, contract_interface = compileContract(contract_source_file)
	    print ("unlock: ", unlockAccount())
	    contractAddress = deployContract(contract_interface)
	    saveToDisk(contractAddress, abi=contract_interface["abi"])
	    return contractName, contract_interface, contractAddress
	
	
	def trySmartContractMethods(myContract, gasForSetCall=GAS_FOR_SET_CALL):
	    """
	    只是测试合同的方法是否有效
		--> 调用 getter 然后调用 setter 然后调用 getter
	    """
	
	    # get
	    answer1 = myContract.functions.get().call()
	    print('.get(): {}'.format(answer1))
	    
	    # set
	    if PARITY_UNLOCK_EACH_TRANSACTION:
	        print ("unlockAccount:", unlockAccount())
	    print('.set()')
	    txParameters = {'from': w3.eth.defaultAccount,
	                    'gas' : gasForSetCall}
	    tx = myContract.functions.set(answer1 + 1).transact(txParameters)
	    tx_hash = w3.toHex( tx )
	    print ("transaction", tx_hash, "... "); sys.stdout.flush()
	    tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)
	    print ("... mined. Receipt --> gasUsed={gasUsed}". format(**tx_receipt) )
	    
	    # get
	    answer2 = myContract.functions.get().call()
	    print('.get(): {}'.format(answer2))
	
	    return answer1, tx_receipt, answer2
	
	if __name__ == '__main__':
	
	    global w3, NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    w3, chainInfos = web3connection(RPCaddress=RPCaddress, account=None)
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = chainInfos
	
	    contract_CompileDeploySave(contract_source_file=FILE_CONTRACT_SOURCE)
	    
	    # argument "test" runs the .set() test transaction
	    if len(sys.argv)>1 and sys.argv[1]=="andtests":
	        contractAddress, abi = loadFromDisk()
	        myContract = contractObject(contractAddress, abi)
	        trySmartContractMethods(myContract)

## 交易发送接口
	vim send.py
	
	#!/usr/bin/env python3
	"""
	@summary: 向示例合约提交许多 contract.set(arg) 交易
	@version: v52 (22/January/2019)
	@since:   17/April/2018
	@author:  https://github.com/drandreaskrueger
	@see:     https://github.com/drandreaskrueger/chainhammer for updates
	"""
	
	# 为导入扩展 sys.path：
	if __name__ == '__main__' and __package__ is None:
	    from os import sys, path
	    sys.path.append(path.dirname(path.dirname(path.abspath(__file__))))
	
	################
	## 依赖项：
      # 标准库：
      
	import sys, time, random, json
	from threading import Thread
	from queue import Queue
	from pprint import pprint
	
	# pypi:
	import requests # pip3 install requests
	import web3
	from web3 import Web3, HTTPProvider # pip3 install web3
	from web3.utils.abi import filter_by_name, abi_to_signature
	from web3.utils.encoding import pad_hex
	
	# chainhammer:
	## 配置
	from hammer.config import RPCaddress, ROUTE, PRIVATE_FOR, EXAMPLE_ABI
	from hammer.config import PARITY_UNLOCK_EACH_TRANSACTION
	from hammer.config import GAS_FOR_SET_CALL
	from hammer.config import FILE_LAST_EXPERIMENT, EMPTY_BLOCKS_AT_END
	## 	部署
	from hammer.deploy import loadFromDisk
	##  客户端工具
	from hammer.clienttools import web3connection, unlockAccount
	
	
	##########################
	## 合约相关
	
	
	def initialize_fromAddress():
	    """
	    由 deploy.py 存储在磁盘文件中的数据，从地址初始化合约对象，
	    """
	    contractAddress, abi = loadFromDisk()
	    myContract = w3.eth.contract(address=contractAddress,
	                                 abi=abi)
	    return myContract
	    
	
	def contract_set_via_web3(contract, arg, hashes = None, privateFor=PRIVATE_FOR, gas=GAS_FOR_SET_CALL):
	    """
	    使用 web3 方法 调用 .set(arg) 方法，可能使用 'privateFor' tx-property
	    """
	    txParameters = {'from': w3.eth.defaultAccount,
	                    'gas' : gas}
	    if privateFor:
	        txParameters['privateFor'] = privateFor  # untested
	        
	    # pprint (txParameters)
	    
	    if PARITY_UNLOCK_EACH_TRANSACTION:
	        unlockAccount()
	        
	    tx = contract.functions.set( x=arg ).transact(txParameters)
	    # print ("[sent via web3]", end=" ")  
	    # TODO: 不在此处打印，而是在开始时打印
	    print (".", end=" ")  # TODO:不在此处打印，而是在开始时打印
	    tx = w3.toHex(tx)
	    
	    if not hashes==None:
	        hashes.append(tx)
	    return tx
	
	
	def try_contract_set_via_web3(contract, arg=42):
	    """
	   测试相关
	    """
	    tx = contract_set_via_web3(contract, arg=arg)
	    print (tx)
	    tx_receipt = w3.eth.waitForTransactionReceipt(tx)
	    storedData = contract.functions.get().call()
	    print (storedData) 
	    return storedData
	
	
	## 手动构建和提交事务，即不通过 web3（@jpmsam 的希望是会加快速度）
	## 请注意，数据编译步骤已经实现为 myContract.functions.myMethod(*args, **kwargs).buildTransaction(transaction) 但以下完全绕过 web3.py！
	
	
	def contract_method_ID(methodname, abi):
	    """
	   从 abi 和方法名构建 4 字节 ID
	    """
	    method_abi = filter_by_name(methodname, abi)
	    assert(len(method_abi)==1)
	    method_abi = method_abi[0]
	    method_signature = abi_to_signature(method_abi) 
	    method_signature_hash_bytes = w3.sha3(text=method_signature) 
	    method_signature_hash_hex = w3.toHex(method_signature_hash_bytes)
	    method_signature_hash_4bytes = method_signature_hash_hex[0:10]
	    return method_signature_hash_4bytes
	
	
	def argument_encoding(contract_method_ID, arg):
	    """
	    连接方法 ID + 填充参数
	    """
	    arg_hex = w3.toHex(arg)
	    arg_hex_padded = pad_hex ( arg_hex, bit_size=256)
	    data = contract_method_ID + arg_hex_padded [2:]
	    return data
	    
	    
	def timeit_argument_encoding():
	    """
	    测试上述内容：
		“这样做 10000 次……花了 0.45 秒”
	    """
	    timer = time.clock()
	    reps = 10000
	    for i in range(reps):
	        method_ID = contract_method_ID("set", ABI)
	        data = argument_encoding(method_ID, 7)
	    timer = time.clock() - timer
	    print (data)
	    # no need to precalculate, it takes near to no time:
	    print ("Doing that %d times ... took %.2f seconds" % (reps, timer) )
	
	
	def contract_set_via_RPC(contract, arg, hashes = None, privateFor=PRIVATE_FOR, gas=GAS_FOR_SET_CALL):
	    """
	    调用 .set(arg) 方法 numTx=10，不通过 web3而直接通过 RPC
		@jpmsam 的建议
		https://github.com/jpmorganchase/quorum/issues/346#issuecomment-382216968
	    """
	    
	    method_ID = contract_method_ID("set", contract.abi) # TODO: 使这个“set”对任何方法名称都灵活
	    data = argument_encoding(method_ID, arg)
	    txParameters = {'from': w3.eth.defaultAccount, 
	                    'to' : contract.address,
	                    'gas' : w3.toHex(gas),
	                    'data' : data} 
	    if privateFor:
	        txParameters['privateFor'] = privateFor  # untested
	    
	    method = 'eth_sendTransaction'
	    payload= {"jsonrpc" : "2.0",
	               "method" : method,
	               "params" : [txParameters],
	               "id"     : 1}
	    headers = {'Content-type' : 'application/json'}
	    response = requests.post(RPCaddress, json=payload, headers=headers)
	    
	    # print('raw json response: {}'.format(response.json()))
	    tx = response.json()['result']
	        
	    # print ("[sent directly via RPC]", end=" ") # TODO: 不在此处打印，而是在开始时打印
	    print (".", end=" ") # TODO: not print this here but at start
	    
	    if not hashes==None:
	        hashes.append(tx)
	    return tx
	
	
	def try_contract_set_via_RPC(contract, steps=3):
	    """
	   测试上面，写3个 transactions，检查 storedData
	    """
	    rand = random.randint(1, 100)
	    for number in range(rand, rand+steps):
	        tx = contract_set_via_RPC(contract, number)
	        print ("after setat(%d) tx" % number, tx, " the storedData now is", end=" ")
	        
	        # TODO: 等待 receipt!
	        storedData = contract.functions.get().call()
	        print (storedData) 
	    
	# 根据常量 ROUTE 选择要选择的路由（web3 / RPC）
	contract_set = contract_set_via_web3   if ROUTE=="web3" else contract_set_via_RPC
	
	
	################################################################
	### 
	### 基准测试路由
	###
	### 0 阻塞
	### 1 异步 
	### 2 async, queue, 可以给多少worker
	### 3 异步, 批处理 (过时)
	###
	################################################################
	
	
	def many_transactions_consecutive(contract, numTx):
	    """
	    原生阻塞方法 --> 15 TPS
	    """
	    print ("send %d transactions, non-async, one after the other:\n" % (numTx))
	    txs = []
	    for i in range(numTx):
	        tx = contract_set(contract, i)
	        print ("set() transaction submitted: ", tx) # Web3.toHex(tx)) # new web3
	        txs.append(tx)
	    return txs
	        
	
	
	def many_transactions_threaded(contract, numTx):
	    """
	    多线程提交许多 transactions。

		注意：1 线程/transactions
		--> 机器可以用完线程，然后崩溃
	    """
	    
	    print ("send %d transactions, multi-threaded, one thread per tx:\n" % (numTx))
	
	    threads = []
	    txs = [] # 保存所有交易哈希的容器
	    for i in range(numTx):
	        t = Thread(target = contract_set,
	                   args   = (contract, i, txs))
	        threads.append(t)
	        print (".", end="")
	    print ("%d transaction threads created." % len(threads))
	
	    for t in threads:
	        t.start()
	        print (".", end="")
	        sys.stdout.flush()
	    print ("all threads started.")
	    
	    for t in threads: 
	        t.join()
	    print ("all threads ended.")
	    
	    return txs
	
	def many_transactions_threaded_Queue(contract, numTx, num_worker_threads=25):
	    """
	    多线程提交许多事务，
		具有大小限制的线程队列
	    """
	
	    line = "send %d transactions, via multi-threading queue with %d workers:\n"
	    print (line % (numTx, num_worker_threads))
	
	    q = Queue()
	    txs = [] # container to keep all transaction hashes
	    
	    def worker():
	        while True:
	            item = q.get()
	            contract_set(contract, item, txs)
	            print ("T", end=""); sys.stdout.flush()
	            q.task_done()
	
	    for i in range(num_worker_threads):
	         t = Thread(target=worker)
	         t.daemon = True
	         t.start()
	         print ("W", end=""); sys.stdout.flush()
	    print ("\n%d worker threads created." % num_worker_threads)
	
	    for i in range(numTx):
	        q.put (i)
	        print ("I", end=""); sys.stdout.flush()
	    print ("\n%d items queued." % numTx)
	
	    q.join()
	    print ("\nall items - done.")
	    
	    return txs
	
	
	def many_transactions_threaded_in_batches(contract, numTx, batchSize=25):
	    """
	    多线程提交许多事务；
		但在数量相当少的批次中。
		
		OBSOLETE <--不比线程2快。 
	    """
	
	    line = "send %d transactions, multi-threaded, one thread per tx, " \
	           "in batches of %d parallel threads:\n"
	    print (line % (numTx, batchSize))
	    
	    txs = [] # container to keep all transaction hashes
	    
	    howManyLeft=numTx
	    while howManyLeft>0:
	    
	        line = "Next batch of %d transactions ... %d left to do"    
	        print (line % (batchSize, howManyLeft))
	        
	        threads = []
	        
	        number = batchSize if howManyLeft>batchSize else howManyLeft 
	        
	        for i in range(number):
	            t = Thread(target = contract_set,
	                       args   = (contract, i, txs))
	            threads.append(t)
	            print (".", end="")
	        print ("\n%d transaction threads created." % len(threads))
	    
	        for t in threads:
	            t.start()
	            print (".", end="")
	            sys.stdout.flush()
	        print ("\nall threads started.")
	        
	        for t in threads: 
	            t.join()
	        print ("\nall threads ended.")
	
	        howManyLeft -= number
	        
	    return txs
	
	
	################################################################
	### 
	### 控制样本：交易是否成功？
	###
	################################################################
	
	
	def hasTxSucceeded(tx_receipt): #, gasGiven=GAS_FOR_SET_CALL):
	    
	    # txReceipt.status 或无
	    status = tx_receipt.get("status", None) 
	    if status == 1:  # 明确答案=交易成功！
	        return True
	    if status == 0:  #明确答案 = 交易失败！
	        return False
	
	    # 不幸的是，并非所有客户端都支持状态字段（例如 testrpc-py、quorum）
	     
	    # 第二种方法是将 gasGiven 与 gasUsed 进行比较：
	    tx_hash=tx_receipt.transactionHash
	    gasGiven = w3.eth.getTransaction(tx_hash)["gas"]
	    gasLeftOver = tx_receipt.gasUsed < gasGiven
	    
	    if not gasLeftOver:
		        # 许多类型的交易失败导致所有给定的gas被用完
				# 例如 一个失败的 assert() 在 solidity 中导致所有 gas 都用完
				# 那么很明显 = 交易失败！
	        return False 
	    
	    if gasLeftOver:
	      # 这是危险的情况，因为
		# 例如 solidity throw / revert() / require() 也返回了一些未使用的气体！
		# 以及成功的交易都返回了一些气体！
		# 但是对于没有状态字段的客户端，这是唯一的指标，所以：
	        return True
	    
	
	def receiptGetter(tx_hash, timeout, resultsDict):
	    try:
	        resultsDict[tx_hash] = w3.eth.waitForTransactionReceipt(tx_hash, timeout)
	    except web3.utils.threads.Timeout:
	        pass
	        
	            
	def getReceipts_multithreaded(tx_hashes, timeout):
	    """
	 	每个 tx_hash 一个线程
	    """
	    
	    tx_receipts = {}
	    print("Waiting for %d transaction receipts, can possibly take a while ..." % len(tx_hashes))
	    threads = []    
	    for tx_hash in tx_hashes:
	        t = Thread(target = receiptGetter,
	                   args   = (tx_hash, timeout, tx_receipts))
	        threads.append(t)
	        t.start()
	    
	    # 等待他们都回来：
	    for t in threads: 
	        t.join()
	    
	    return tx_receipts
	
	
	def controlSample_transactionsSuccessful(txs, sampleSize=50, timeout=100):
	    """
	    确保交易实际上是成功的，并且没有失败，例如 没气等

		要对成功状态更改的速度进行基准测试！！
		
		方法：而不是检查每笔交易，这只是需要一些样本。
		它可能以三种截然不同的方式失败：
		
		* 等待 tx-receipt 时超时，然后尝试提高超时秒数
		* tx_receipt.status == 0 对于任何采样交易。 真正的交易失败！
		*所有给定的气体都用完了。 它只是交易失败的间接指标。
	    """
	    
	    print ("Check control sample.")
	    N = sampleSize if len(txs)>sampleSize else len(txs) 
	    txs_sample = random.sample(txs, N)
	    
	    tx_receipts = getReceipts_multithreaded(tx_hashes=txs_sample, timeout=timeout) 
	    
	    #测试1：所有 receipts 都在这里吗？ 
	    M = len(tx_receipts)
	    if M != N:
	        print ("Bad: Timeout, received receipts only for %d out of %d sampled transactions." % (M, N))
	        success = False 
	    else:
	        print ("Good: No timeout, received the receipts for all %d sampled transactions." % N)
	        success = True
	            
	    # 测试2:每笔交易都成功了吗？
	    badCounter=0
	    for tx_hash, tx_receipt in tx_receipts.items():
	        #status = tx_receipt.get("status", None) # 不幸的是，并不是所有的客户端都支持这个
	        # print ((tx_hash, status, tx_receipt.gasUsed ))
	        if not hasTxSucceeded(tx_receipt):
	            success = False
	            print ("Transaction NOT successful:", tx_hash, tx_receipt)
	            badCounter = badCounter+1 
	    # pprint (dict(tx_receipt))
	
	    if badCounter:
	        print ("Bad: %d out of %d not successful!" % (badCounter, M))
	        
	    print ("Sample of %d transactions checked ... hints at:" % M, end=" ")
	    print( "TOTAL SUCCESS :-)" if success else "-AT LEAST PARTIAL- FAILURE :-(" )
	    
	    return success
	
	
	# Try out the above with
	# pytest tests/test_send.py::test_controlSample_transactionsSuccessful
	
	
	################################################################################
	### 
	###估计块的范围，第一个和最后 100 个transaction哈希
	###
	################################################################################
	
	
	def getReceipts_multithreaded_Queue(tx_hashes, timeout, num_worker_threads=8, ifPrint=False):
	    """
	    通过具有 8 个工作线程的多线程队列查询 RPC。

		优于“getReceipts_multithreaded”的优势：

		也适用于 len(tx_hashes) > 1000
	    """
	    
	    start=time.monotonic()
	    q = Queue()
	    tx_receipts = {}
	    
	    def worker():
	        while True:
	            tx_hash = q.get()
	            receiptGetter(tx_hash, timeout, tx_receipts)
	            q.task_done()
	
	    for i in range(num_worker_threads):
	         t = Thread(target=worker)
	         t.daemon = True
	         t.start()
	
	    for tx in tx_hashes:
	        q.put (tx)
	
	    q.join()
	    
	    if ifPrint:
	        duration = time.monotonic() - start
	        print ("%d lookups took %.1f seconds" % (len(tx_receipts), duration))
	    
	    return tx_receipts
	
	
	def when_last_ones_mined__give_range_of_block_numbers(txs, txRangesSize=100, timeout=60):
	    """
	    也只是一个启发式：
		假设前 100 个和后 100 个事务哈希
		已添加到列表 'txs'
		可以揭示整个实验的最小和最大区块数
	    """
	
	    txs_begin_and_end = txs[:txRangesSize] + txs[-txRangesSize:]
	    tx_receipts = getReceipts_multithreaded_Queue(tx_hashes=txs_begin_and_end, 
	                                                  timeout=timeout) #, ifPrint=True)
	    
	    # 或者实际上，所有这些？ Naaa，太慢了：
		# TestRPC: 2000 次查找耗时 122.1 秒
		# Parity: 2000 次查找耗时 7.2 秒
		# Geth：2000 次查找耗时 8.6 秒
	    # tx_receipts = getReceipts_multithreaded_Queue(tx_hashes=txs,  timeout=timeout, ifPrint=True)
	    
	    blockNumbers = [receipt.blockNumber for receipt in tx_receipts.values()]
	    blockNumbers = sorted(list(set(blockNumbers))) # make unique
	    # print (blockNumbers) 
	    return min(blockNumbers), max(blockNumbers)
	
	    
	def store_experiment_data(success, num_txs, 
	                          block_from, block_to, 
	                          empty_blocks,
	                          filename=FILE_LAST_EXPERIMENT):
	    """
	    关于最后一个实验的最基本数据，
		存储在相同的（覆盖的）文件中。
		目的：图表应该能够计算适当的平均值和选择范围
	    """
	    data = {"send" : {
	                "block_first" : block_from,
	                "block_last": block_to,
	                "empty_blocks": empty_blocks, 
	                "num_txs" : num_txs,
	                "sample_txs_successful": success
	                },
	            "node" : {
	                "rpc_address": RPCaddress,
	                "web3.version.node": w3.version.node,
	                "name" : NODENAME,
	                "type" : NODETYPE,
	                "version" : NODEVERSION, 
	                "consensus" : CONSENSUS, 
	                "network_id" : NETWORKID, 
	                "chain_name" : CHAINNAME, 
	                "chain_id" : CHAINID
	                }
	            }
	            
	    with open(filename, "w") as f:
	        json.dump(data, f)
	    
	
	def wait_some_blocks(waitBlocks=EMPTY_BLOCKS_AT_END, pauseBetweenQueries=0.3):
	    """
	    实际上，这里必须等待，
		因为 ./send.py 比 ./tps.py 启动晚
		所以当 ./send.py 结束时，就可以进行分析了。
	    """
	    blockNumber_start = w3.eth.blockNumber
	    print ("blocknumber now:", blockNumber_start, end=" ")
	    print ("waiting for %d empty blocks:" % waitBlocks)
	    bn_previous=bn_now=blockNumber_start
	    
	    while bn_now < waitBlocks + blockNumber_start:
	        time.sleep(pauseBetweenQueries)
	        bn_now=w3.eth.blockNumber
	        # print (bn_now, waitBlocks + blockNumber_start)
	        if bn_now!=bn_previous:
	            bn_previous=bn_now
	            print (bn_now, end=" "); sys.stdout.flush()
	         
	    print ("Done.")
	
	        
	def finish(txs, success):
	    block_from, block_to = when_last_ones_mined__give_range_of_block_numbers(txs)
	    txt = "Transaction receipts from beginning and end all arrived. Blockrange %d to %d."
	    txt = txt % (block_from, block_to)
	    print (txt)
	    
	    if NODETYPE=="TestRPC" or (NODENAME=="Parity" and CHAINNAME=="developmentchain" and NETWORKID==17):
	        print ("Do not wait for empty blocks, as this is TestRPC, or parity instantseal.")
	        waitBlocks=0
	    else:
	        waitBlocks=EMPTY_BLOCKS_AT_END
	        wait_some_blocks(waitBlocks)
	    
	    store_experiment_data (success, len(txs), block_from, block_to, empty_blocks=waitBlocks)
	    # print ("Data stored. This will trigger tps.py to end in ~ %d blocks." % EMPTY_BLOCKS_AT_END)
	    
	    print ("Data stored. This will trigger tps.py to end.\n"
	           "(Beware: Wait ~0.5s until tps.py stops and writes to same file.)")
	    #          see tps.py --> pauseBetweenQueries=0.3
	
	
	################################################################################
	###
	###选择，取决于 CLI 参数
	###
	################################################################################
	
	def check_CLI_or_syntax_info_and_exit():
	    """
	  	在任何事情之前，检查参数数量是否正常，或打印语法说明
	    """
	    
	    #print ("len(sys.argv)=", len(sys.argv))
	    
	    if not (2 <= len(sys.argv) <= 4):
	        print ("Needs parameters:")
	        print ("%s numTransactions algorithm [workers]" % sys.argv[0])
	        print ("at least numTransactions, e.g.")
	        print ("%s 1000" % sys.argv[0])
	        exit()
	
	
	def sendmany(contract):
	    """
	   向合约发送许多交易。
		根据第二个 CLI 参数选择算法。
	    """
	
	    # TODO: 也许将其扩展到一个完整的 (config.py) 设置打印机？
		# 但在 tps.py 中，因为只有该输出在 run.sh 中可见
	    
	    print("\nCurrent blockNumber = ", w3.eth.blockNumber)
	    numTransactions = int(sys.argv[1])
	    if ROUTE=="RPC": route = "RPC directly" 
	    if ROUTE=="web3": route = "web3 library" 
	    print ("You want me to send %d transactions, via route: %s." % (numTransactions, route))
	    
	    # 根据第二个 CLI 参数选择算法：
	    
	    if len(sys.argv)==2 or sys.argv[2]=="sequential":
	        
	        # 阻塞，非异步
	        txs=many_transactions_consecutive(contract, numTransactions)  
	        
	    elif sys.argv[2]=="threaded1":
	        txs=many_transactions_threaded(contract, numTransactions)
	            
	            
	    elif sys.argv[2]=="threaded2":
	        num_workers = 100
	        if len(sys.argv)==4:
	            try:
	                num_workers = int(sys.argv[3])
	            except:
	                pass
	            
	        txs=many_transactions_threaded_Queue(contract, 
	                                         numTx=numTransactions, 
	                                         num_worker_threads=num_workers)
	        
	    elif sys.argv[2]=="threaded3":
	        batchSize=25
	        txs=many_transactions_threaded_in_batches(contract, 
	                                              numTx=numTransactions, 
	                                              batchSize=batchSize)
	          
	    else:
	        print ("Nope. Choice '%s'" % sys.argv[2], "not recognized.")
	        exit()
	
	        
	    print ("%d transaction hashes recorded, examples: %s" % (len(txs), txs[:2]))
	    
	    return txs
	
	
	if __name__ == '__main__':
	    
	    check_CLI_or_syntax_info_and_exit()
	
	    global w3, NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    w3, chainInfos = web3connection(RPCaddress=RPCaddress, account=None)
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = chainInfos
	    
	    # wait_some_blocks(0); exit()
	    # timeit_argument_encoding(); exit()
	    # try_contract_set_via_web3(contract); exit()
	    # try_contract_set_via_RPC(contract);  exit()
	
	    w3.eth.defaultAccount = w3.eth.accounts[0] # set first account as sender
	    contract = initialize_fromAddress()
	
	    txs = sendmany(contract)
	    sys.stdout.flush() # so that the log files are updated.
	
	    success = controlSample_transactionsSuccessful(txs)
	    sys.stdout.flush()
	
	    finish(txs, success)
	    sys.stdout.flush()

## 交易记时
	vim tps.py
	
	#!/usr/bin/env python3
	# from __future__ import print_function
	
	"""
	@summary: 进入链的计时交易
	@version: v46 (03/January/2019)
	@since:   17/April/2018
	@organization: 
	@author:  https://github.com/drandreaskrueger
	@see:     https://github.com/drandreaskrueger/chainhammer for updates
	"""
	
	
	import time, timeit, sys, os, json
	
	from web3 import Web3, HTTPProvider
	
	# 扩展导入路径：
	if __name__ == '__main__' and __package__ is None:
	    from os import sys, path
	    sys.path.append(path.dirname(path.dirname(path.abspath(__file__))))
	    
	from hammer.config import RPCaddress2, FILE_LAST_EXPERIMENT, AUTOSTOP_TPS, EMPTY_BLOCKS_AT_END
	from hammer.deploy import loadFromDisk, FILE_CONTRACT_ADDRESS
	from hammer.clienttools import web3connection, getBlockTransactionCount
	    
	
	def loopUntil_NewContract(query_intervall = 0.1):
	    """
	    等待部署新的智能合约。

		连续轮询文件“FILE_CONTRACT_ADDRESS”。
		当覆盖的文件具有不同的地址或新的文件日期时返回。

		N.B.：实际上再次创建了相同的以太坊合约地址，
		如果区块链被删除，一切都重新开始。 所以：检查文件日期。
	    """
	    
	    address, _ = loadFromDisk()
	    when = os.path.getmtime(FILE_CONTRACT_ADDRESS) 
	    print ("(filedate %d) last contract address: %s" %(when, address)) 
	    
	    while(True):
	        time.sleep(query_intervall)
	        
	        # 检查是否部署了新合约
			# 因为那时新地址已保存到文件中：
	        newAddress, _ = loadFromDisk()
	        newWhen = os.path.getmtime(FILE_CONTRACT_ADDRESS)
	        if (newAddress != address or newWhen != when):
	            print ("(filedate %d) new contract address: %s" %(newWhen, newAddress))  
	            break
	    return
	
	def timestampToSeconds(timestamp, NODENAME, CONSENSUS):
	    """
	  将时间戳转换为（浮点数）秒
		作为一个单独的函数，以便它可以在 blocksDB_create.py 中回收
	    """
	        
	    #大多数以太坊客户端以整秒的形式返回块时间戳：
	    timeunits = 1.0
	    
	    # quorum raft 共识...返回的不是秒而是纳秒？
	    if CONSENSUS=="raft": timeunits = 1000000000.0
	    
	    # testrpc-py 有奇数的时间戳单位 ;-)
		# 检查更新：https://github.com/pipermerriam/eth-testrpc/issues/117
	    if NODENAME=="TestRPC": timeunits = 205.0
	    
	    return timestamp / timeunits
	 
	
	def analyzeNewBlocks(blockNumber, newBlockNumber, txCount, start_time, peakTpsAv):
	    """
	   遍历所有新块，将交易数量加起来,打印状态行
	    """
	    
	    txCount_new = 0
	    for bl in range(blockNumber+1, newBlockNumber+1): # TODO check range again - shift by one? 
	        # txCount_new += w3.eth.getBlockTransactionCount(bl)
	        blktx = getBlockTransactionCount(w3, bl)
	        txCount_new += blktx # TODO
	
	    ts_blockNumber =    w3.eth.getBlock(   blockNumber).timestamp
	    ts_newBlockNumber = w3.eth.getBlock(newBlockNumber).timestamp
	    ts_diff = ts_newBlockNumber - ts_blockNumber
	    
	    blocktimeSeconds = timestampToSeconds(ts_diff, NODENAME, CONSENSUS) 
	
	    try:
	        tps_current = txCount_new / blocktimeSeconds
	    except ZeroDivisionError:
	      # 奇怪： Parity 似乎有整秒的块时间分辨率？
		# 所以如果块来得更快（例如使用即时密封），
		# 然后他们最终的阻塞时间为零。
		# 然后，将 TPS_CURRENT 设置为错误但语法正确的值。
	        tps_current = 0
	
	    txCount += txCount_new
	    elapsed = timeit.default_timer() - start_time
	    tpsAv = txCount / elapsed
	    
	    if tpsAv > peakTpsAv:
	        peakTpsAv = tpsAv 
	    
	    verb = " is" if peakTpsAv==tpsAv else "was"  
	    
	    line = "block %d | new #TX %3d / %4.0f ms = " \
	           "%5.1f TPS_current | total: #TX %4d / %4.1f s = %5.1f TPS_average " \
	           "(peak %s %5.1f TPS_average)" 
	    line = line % ( newBlockNumber, txCount_new, blocktimeSeconds * 1000, 
	                    tps_current, txCount, elapsed, tpsAv, verb, peakTpsAv) 
	    print (line)
	    
	    return txCount, peakTpsAv, tpsAv
	
	
	def sendingEndedFiledate():
	    try:
	        when = os.path.getmtime(FILE_LAST_EXPERIMENT)
	    except FileNotFoundError:
	        when = 0
	    return when
	
	
	def readInfofile(fn=FILE_LAST_EXPERIMENT):
	    with open(fn, "r") as f:
	        data = json.load(f)
	    return data
	    
	
	class CodingError(Exception):
	    pass
	
	
	def getNearestEntry(myDict, myIndex):
	    """
	    因为 
	      finalTpsAv = tpsAv[block_last]
	    有时不能解决，那就选择
	      finalTpsAv = tpsAv[block_last+i]
	   测试增加 i，减少 i
	    """
	    answer = myDict.get(myIndex, None)
	    if answer:
	        return answer
	
	    maxIndex,minIndex = max(myDict.keys()), min(myDict.keys())
	
	    #先看后：
	    i = myIndex
	    while not answer:
	        i += +1
	        if i>maxIndex:
	            break
	        answer = myDict.get(i, None)
	
	    # 然后看早期：
	    i=myIndex
	    while not answer:
	        i += -1
	        if i<minIndex:
	            raise CodingError("Ouch, this should never happen. Info: len(myDict)=%d myIndex=%d" %(len(myDict), myIndex)) 
	        answer = myDict.get(i, None)
	        
	        
	    return answer
	
	
	def measurement(blockNumber, pauseBetweenQueries=0.3, 
	                RELAXATION_ROUNDS=3, empty_blocks_at_end=EMPTY_BLOCKS_AT_END):
	    """
	   当一个（或更多）新块出现时，将它们添加到总数中，然后打印一行。
	    """
	
	    whenBefore = sendingEndedFiledate()
	
	      # 一直在等待的区块已经包含了第一笔交易
		# N.B.：时间测量有轻微的不准确，因为没有测量需要多长时间
	    
	    # txCount=w3.eth.getBlockTransactionCount(blockNumber)
	    txCount=getBlockTransactionCount(w3, blockNumber)
	    
	    start_time = timeit.default_timer()
	    start_epochtime = time.time()
	    # TODO: 也许除了经过的系统时间之外，显示阻塞时间？
	    
	    print('starting timer, at block', blockNumber, 'which has ', 
	          txCount,' transactions; at epochtime', start_epochtime)
	    
	    peakTpsAv = 0
	    counterStart, blocknumberEnd = 0, -1
	    
	    tpsAv = {} # 记住所有这些，所以我们可以在'block_last'返回值
	    
	    while(True):
	        newBlockNumber=w3.eth.blockNumber
	        
	        if(blockNumber!=newBlockNumber): # when a new block appears:
	            args = (blockNumber, newBlockNumber, txCount, start_time, peakTpsAv)
	            txCount, peakTpsAv, tpsAv[newBlockNumber] = analyzeNewBlocks(*args)
	            blockNumber = newBlockNumber
	            
	            
	            # 对于前 3 轮，总是再次重置 peakTpsAv！
	            if counterStart < RELAXATION_ROUNDS:
	                peakTpsAv=0
	            counterStart += 1
	
	        # send.py --> store_experiment_data() 在最后一个 tx 被挖掘之后被调用。
	        # 然后再做10个空块......
	        # 只有然后结束这个：
	        # 如果 AUTOSTOP_TPS and blocknumberEnd==-1 和 sendingEndedFiledate()!=whenBefore:
	        if AUTOSTOP_TPS and sendingEndedFiledate()!=whenBefore:
	            print ("Received signal from send.py = updated INFOFILE.")
	            block_last = readInfofile()['send']['block_last']
	            
	            # finalTpsAv = tpsAv[block_last]
	            finalTpsAv = getNearestEntry(myDict=tpsAv, myIndex=block_last)
	            
	            break
	            # finalTpsAv = tpsAv
	            # blocknumberEnd = newBlockNumber + empty_blocks_at_end
	            # print ("The end is nigh ... after blocknumber", blocknumberEnd)
	            # if NODETYPE=="TestRPC":
	            #   break # TestRPC 中没有空块
	        # if blocknumberEnd>0 and newBlockNumber > blocknumberEnd:
	            # break
	
	        time.sleep(pauseBetweenQueries) # do not query too often; as little side effect on node as possible
	        
	    # print ("end")   # N.B.: it never gets here !
	    txt = "Experiment ended! Current blocknumber = %d"
	    txt = txt % (w3.eth.blockNumber)
	    print (txt)
	    return peakTpsAv, finalTpsAv, start_epochtime
	
	def addMeasurementToFile(peakTpsAv, finalTpsAv, start_epochtime, fn=FILE_LAST_EXPERIMENT):
	    with open(fn, "r") as f:
	        data = json.load(f)
	    data["tps"]={}
	    data["tps"]["peakTpsAv"] = peakTpsAv
	    data["tps"]["finalTpsAv"] = finalTpsAv
	    data["tps"]["start_epochtime"] = start_epochtime
	
	    with open(fn, "w") as f:
	        json.dump(data, f)
	             
	if __name__ == '__main__':
	    
	    global w3, NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID
	    w3, chainInfos = web3connection(RPCaddress=RPCaddress2, account=None)
	    NODENAME, NODETYPE, NODEVERSION, CONSENSUS, NETWORKID, CHAINNAME, CHAINID = chainInfos
	    
	    blockNumber_before = w3.eth.blockNumber
	    print ("\nBlock ",blockNumber_before," - waiting for something to happen") 
	    
	    loopUntil_NewContract()
	    blocknumber_start_here = w3.eth.blockNumber 
	    print ("\nblocknumber_start_here =", blocknumber_start_here)
	    
	    peakTpsAv, finalTpsAv, start_epochtime = measurement( blocknumber_start_here )
	    
	    addMeasurementToFile(peakTpsAv, finalTpsAv, start_epochtime, FILE_LAST_EXPERIMENT)
	    print ("Updated info file:", FILE_LAST_EXPERIMENT, "THE END.")	    	

	    