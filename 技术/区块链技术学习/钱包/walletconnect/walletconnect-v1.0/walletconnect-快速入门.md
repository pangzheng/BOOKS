# walletconnect-快速入门-Dapp
## 独立客户端
- 安装 walletconnect sdk
	- npm

			npm install --save @walletconnect/client @walletconnect/qrcode-modal
	- yarn

			yarn add @walletconnect/client @walletconnect/qrcode-modal
- 初始化链接

		import WalletConnect from "@walletconnect/client";
		import QRCodeModal from "@walletconnect/qrcode-modal";
		
		//创建一个新链接
		const connector = new WalletConnect({
		  bridge: "https://bridge.walletconnect.org", //必要参数
		  qrcodeModal: QRCodeModal,
		});
		
		// 检查连接是否已经建立
		if (!connector.connected) {
		  // 创建新会话
		  connector.createSession();
		}
		
		//订阅连接事件
		connector.on("connect", (error, payload) => {
		  if (error) {
		    throw error;
		  }
		
		  // 获取提供的账户和 chainId
		  const { accounts, chainId } = payload.params[0];
		});
		
		connector.on("session_update", (error, payload) => {
		  if (error) {
		    throw error;
		  }
		
		  // 获取更新的账户和chainId
		  const { accounts, chainId } = payload.params[0];
		});
		
		connector.on("disconnect", (error, payload) => {
		  if (error) {
		    throw error;
		  }
		
		  // 删除链接
		});
- 发送交易(`eth_sendTransaction`)

		// 交易草稿
		const tx = {
		  from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3", //必要参数
		  to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359", //必要参数(对于非合约部署)
		  data: "0x", //必要参数
		  gasPrice: "0x02540be400", // 可选
		  gas: "0x9c40", // 可选
		  value: "0x00", // 可选
		  nonce: "0x0114", // 可选
		};
		
		// 发送交易
		connector
		  .sendTransaction(tx)
		  .then((result) => {
		    // 返回交易 id = txid; (hash)
		    console.log(result);
		  })
		  .catch((error) => {
		    // 拒绝时返回错误
		    console.error(error);
		  });		
- 签名交易(`eth_signTransaction`)

		// 交易草稿
		const tx = {
		  from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3", //必要参数
		  to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359", //必要参数(对于非合约部署)
		  data: "0x", //必要参数
		  gasPrice: "0x02540be400", // 可选
		  gas: "0x9c40", // 可选
		  value: "0x00", // 可选
		  nonce: "0x0114", // 可选
		};
		
		// 签名交易
		connector
		  .signTransaction(tx)
		  .then((result) => {
		    // 返回签名交易
		    console.log(result);
		  })
		  .catch((error) => {
		    // 拒绝时返回错误
		    console.error(error);
		  });
- 签名个人信息(`personal_sign`)

		// 草稿消息参数
		const message = "My email is john@doe.com - 1537836206101";
		
		const msgParams = [
		  convertUtf8ToHex(message), //必要参数
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3", //必要参数
		];
		
		//签名个人信息
		connector
		  .signPersonalMessage(msgParams)
		  .then((result) => {
		    // 返回签名信息
		    console.log(result);
		  })
		  .catch((error) => {
		    // 拒绝时返回错误
		    console.error(error);
		  });	
- 签名信息(`eth_sign`)

		// 草稿消息参数
		const message = "My email is john@doe.com - 1537836206101";
		
		const msgParams = [
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3",                            //必要参数
		  keccak256("\x19Ethereum Signed Message:\n" + len(message) + message))    //必要参数
		];
		
		
		//签名个人信息
		connector
		  .signMessage(msgParams)
		  .then((result) => {
		    // 返回签名信息
		    console.log(result)
		  })
		  .catch(error => {
		    // 拒绝时返回错误
		    console.error(error);
		  })		  	  
- 签名类型数据(`eth_signTypedData`)

		// 草稿消息参数
		const typedData = {
		  types: {
		    EIP712Domain: [
		      { name: "name", type: "string" },
		      { name: "version", type: "string" },
		      { name: "chainId", type: "uint256" },
		      { name: "verifyingContract", type: "address" },
		    ],
		    Person: [
		      { name: "name", type: "string" },
		      { name: "account", type: "address" },
		    ],
		    Mail: [
		      { name: "from", type: "Person" },
		      { name: "to", type: "Person" },
		      { name: "contents", type: "string" },
		    ],
		  },
		  primaryType: "Mail",
		  domain: {
		    name: "Example Dapp",
		    version: "1.0",
		    chainId: 1,
		    verifyingContract: "0x0000000000000000000000000000000000000000",
		  },
		  message: {
		    from: {
		      name: "Alice",
		      account: "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
		    },
		    to: {
		      name: "Bob",
		      account: "0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
		    },
		    contents: "Hey, Bob!",
		  },
		};
		
		const msgParams = [
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3", //必要参数
		  typedData, //必要参数
		];
		
		// 签名数据类型
		connector
		  .signTypedData(msgParams)
		  .then((result) => {
		     // 返回结果信息
		    console.log(result);
		  })
		  .catch((error) => {
		     // 拒绝时返回错误
		    console.error(error);
		  });
- 发送自定义请求

		// 自定义请求草稿
		const customRequest = {
		  id: 1337,
		  jsonrpc: "2.0",
		  method: "eth_signTransaction",
		  params: [
		    {
		      from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3",
		      to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359",
		      data: "0x",
		      gasPrice: "0x02540be400",
		      gas: "0x9c40",
		      value: "0x00",
		      nonce: "0x0114",
		    },
		  ],
		};
		
		// 发送自定义请求
		connector
		  .sendCustomRequest(customRequest)
		  .then((result) => {
		    // 返回结果信息
		    console.log(result);
		  })
		  .catch((error) => {
		     // 拒绝时返回错误
		    console.error(error);
		  });		  					

## NodeJS 客户端
- 安装
	- npm

			npm install --save @walletconnect/node @walletconnect/qrcode-modal
	- yarn

			yarn add @walletconnect/node @walletconnect/qrcode-modal
- 初始化链接

			import NodeWalletConnect from "@walletconnect/node";
			import WalletConnectQRCodeModal from "@walletconnect/qrcode-modal";
			
			// 创建链接
			const walletConnector = new NodeWalletConnect(
			  {
			    bridge: "https://bridge.walletconnect.org", // 必要参数
			  },
			  {
			    clientMeta: {
			      description: "WalletConnect NodeJS Client",
			      url: "https://nodejs.org/en/",
			      icons: ["https://nodejs.org/static/images/logo.svg"],
			      name: "WalletConnect",
			    },
			  }
			);
			
			// 检查连接是否已经建立
			if (!walletConnector.connected) {
			  // 创建新会话
			  walletConnector.createSession().then(() => {
			    // 获取 QR 二维码码模式的 uri
			    const uri = walletConnector.uri;
			    // 显示静态二维码
			    WalletConnectQRCodeModal.open(
			      uri,
			      () => {
			        console.log("QR Code Modal closed");
			      },
			      true // isNode = true
			    );
			  });
			}
			
			// 订阅连接事件
			walletConnector.on("connect", (error, payload) => {
			  if (error) {
			    throw error;
			  }
			
			  // 关闭二维码模式
			  WalletConnectQRCodeModal.close(
			    true // isNode = true
			  );
			
			  // 获取提供的账户和 chainId
			  const { accounts, chainId } = payload.params[0];
			});
			
			walletConnector.on("session_update", (error, payload) => {
			  if (error) {
			    throw error;
			  }
			
			  // 获取更新的账户和 chainId
			  const { accounts, chainId } = payload.params[0];
			});
			
			walletConnector.on("disconnect", (error, payload) => {
			  if (error) {
			    throw error;
			  }
			
			  // 删除钱包链接
			});
- 发送交易(`eth_sendTransaction`)

		// 交易草稿
		const tx = {
		  from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3", // 必要参数		  to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359", // 必要参数(对于非合约部署)
		  data: "0x", // 必要参数
		  gasPrice: "0x02540be400", // 选填
		  gas: "0x9c40", // 选填
		  value: "0x00", // 选填
		  nonce: "0x0114", // 选填
		};
		
		//发送交易
		walletConnector
		  .sendTransaction(tx)
		  .then((result) => {
		    // 返回交易 id= txid= (hash)
		    console.log(result);
		  })
		  .catch((error) => {
		   // 拒绝时返回错误
		    console.error(error);
		  });	
- 签署交易 (`eth_signTransaction`)	

		// 交易草稿
		const tx = {
		  from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3", // 必要参数
		  to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359", //必要参数(对于非合约部署)
		  data: "0x", // 必要参数
		  gasPrice: "0x02540be400", // 选填
		  gas: "0x9c40", // 选填
		  value: "0x00", // 选填
		  nonce: "0x0114", // 选填
		};
		
		// 签名交易
		walletConnector
		  .signTransaction(tx)
		  .then((result) => {
		    // 返回签名交易
		    console.log(result);
		  })
		  .catch((error) => {
		    // 拒绝时返回错误
		    console.error(error);
		  });	
- 签名个人信息 (`personal_sign`)

		// 信息参数草稿
		const message = "My email is john@doe.com - 1537836206101"
		
		const msgParams = [
		  convertUtf8ToHex(message)                                                 // 必要参数
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3",                             // 必要参数
		];
		
		// 签名个人信息
		walletConnector
		  .signPersonalMessage(msgParams)
		  .then((result) => {
		    // 返回签名
		    console.log(result)
		  })
		  .catch(error => {
		    // 拒绝时返回错误
		    console.error(error);
		  })
- 签名信息 (`eth_sign`)

		// 信息参数草稿
		const message = "My email is john@doe.com - 1537836206101";
		
		const msgParams = [
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3",                            // 必要参数
		  keccak256("\x19Ethereum Signed Message:\n" + len(message) + message))    // 必要参数
		];
		
		
		// 签名信息
		walletConnector
		  .signMessage(msgParams)
		  .then((result) => {
		    // 返回签名
		    console.log(result)
		  })
		  .catch(error => {
		   // 拒绝时返回错误
		    console.error(error);
		  })		  		    					
- 签名类型数据 (`eth_signTypedData`)

		//信息参数草稿
		const typedData = {
		  types: {
		    EIP712Domain: [
		      { name: "name", type: "string" },
		      { name: "version", type: "string" },
		      { name: "chainId", type: "uint256" },
		      { name: "verifyingContract", type: "address" },
		    ],
		    Person: [
		      { name: "name", type: "string" },
		      { name: "account", type: "address" },
		    ],
		    Mail: [
		      { name: "from", type: "Person" },
		      { name: "to", type: "Person" },
		      { name: "contents", type: "string" },
		    ],
		  },
		  primaryType: "Mail",
		  domain: {
		    name: "Example Dapp",
		    version: "1.0",
		    chainId: 1,
		    verifyingContract: "0x0000000000000000000000000000000000000000",
		  },
		  message: {
		    from: {
		      name: "Alice",
		      account: "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
		    },
		    to: {
		      name: "Bob",
		      account: "0xbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
		    },
		    contents: "Hey, Bob!",
		  },
		};
		
		const msgParams = [
		  "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3", // 必要参数
		  typedData, // 必要参数
		];
		
		// 签名类型数据
		walletConnector
		  .signTypedData(msgParams)
		  .then((result) => {
		    // 返回签名
		    console.log(result);
		  })
		  .catch((error) => {
		    // 拒绝时返回错误
		    console.error(error);
		  });
- 发送自定义请求

		//自定义请求草稿
		const customRequest = {
		  id: 1337,
		  jsonrpc: "2.0",
		  method: "eth_signTransaction",
		  params: [
		    {
		      from: "0xbc28Ea04101F03aA7a94C1379bc3AB32E65e62d3",
		      to: "0x89D24A7b4cCB1b6fAA2625Fe562bDd9A23260359",
		      data: "0x",
		      gasPrice: "0x02540be400",
		      gas: "0x9c40",
		      value: "0x00",
		      nonce: "0x0114",
		    },
		  ],
		};
		
		// 发送自定义请求
		walletConnector
		  .sendCustomRequest(customRequest)
		  .then((result) => {
		    // 返回请求结果
		    console.log(result);
		  })
		  .catch((error) => {
		   // 拒绝时返回错误
		    console.error(error);
		  });
		  
## React-Native
一个嵌入式库，可帮助您轻松地将您的 [React Native dapps](https://reactnative.dev/) 连接到 [Android](https://reactnative.dev/)、[iOS](https://reactnative.dev/)和 [Web](https://github.com/necolas/react-native-web) 上的[以太坊](https://ethereum.org/)钱包。

注意：此库假定您已经在应用程序中启用了对 [Web3](https://github.com/ChainSafe/web3.js) 的先决条件支持。这可以通过使用创建一个新项目 [npx create-react-native-dapp](https://github.com/cawfree/create-react-native-dapp) 或者通过使用在现有项目中引入对 Web3 的支持来完成 [npx rn-nodeify --install --hack](https://github.com/tradle/rn-nodeify)。

有关更多详细信息，请查看[文档](https://docs.walletconnect.org/)。
### 安装
要开始，请安装 `@walletconnect/react-native-dapp`

	yarn add @walletconnect/react-native-dapp
如果您还没有安装，您可能还需要 [react-native-svg](https://github.com/react-native-svg/react-native-svg) 与持久存储提供程序一起安装，例如 [@react-native-async-storage/async-storage](https://github.com/react-native-async-storage/async-storage)：

	yarn add react-native-svg @react-native-async-storage/async-storage
### 架构
该库是使用 [React Context API](https://reactjs.org/docs/context.html) 实现的，该 API 用于帮助 [connector](https://docs.walletconnect.org/client-api) 在整个应用程序中创建全局可访问的实例。这允许您在甚至深度嵌套的组件中使用统一实例，并确保您呈现的应用程序始终与连接器状态同步。

- `WalletConnectProvider`

	在应用程序的根目录中，可以声明一个  [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx) ，它控制对[连接器实例](https://docs.walletconnect.org/client-api)的访问和持久性
	
		import * as React from 'react';
		import WalletConnectProvider from '@walletconnect/react-native-dapp';
		import AsyncStorage from '@react-native-async-storage/async-storage';
		
		export default function App(): JSX.Element {
		  return (
		    <WalletConnectProvider
		      redirectUrl={Platform.OS === 'web' ? window.location.origin : 'yourappscheme://'}
		      storageOptions= {{
		        asyncStorage: AsyncStorage,
		      }}>
		      <>{/* awesome app here */}</>
		    </WalletConnectProvider>
		  );
		}
	上面，我们传递了 [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx) 两个必需的参数；`redirectUrl`和`storageOptions`：

	- `redirectUrl` 

		用于帮助控制外部钱包和您的应用程序之间的导航。在上 web 您只需要指定一个有效的申请路线；而在移动平台上，您必须指定[深度链接 URI 方案](https://docs.expo.io/workflow/linking/#universaldeep-links-without-a-custom-scheme)。
	- `storageOptions`

		允许指定必须用于持久会话数据的存储引擎
		
		- 尽管在我们的示例中我们使用了 [@react-native-async-storage/async-storage](https://github.com/react-native-async-storage/async-storage)，但这可以是您喜欢的引擎，只要它符合 [IAsyncStorage](https://github.com/pedrouid/keyvaluestorage) 通用存储接口声明。
	
	值得注意的是，[WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx) 可选地接受接口 `WalletConnect` 定义的配置参数：[IWalletConnectOptions](https://github.com/WalletConnect/walletconnect-monorepo/tree/next/packages/helpers/utils)
		
		import * as React from 'react';
		import WalletConnectProvider from '@walletconnect/react-native-dapp';
		import AsyncStorage from '@react-native-async-storage/async-storage';
			
		export default function App(): JSX.Element {
		  return (
		    <WalletConnectProvider
		      bridge="https://bridge.walletconnect.org"
		      clientMeta={{
		        description: 'Connect with WalletConnect',
		        url: 'https://walletconnect.org',
		        icons: ['https://walletconnect.org/walletconnect-logo.png'],
		        name: 'WalletConnect',
		      }}
		      redirectUrl={Platform.OS === 'web' ? window.location.origin : 'yourappscheme://'}
		      storageOptions= {{
		        asyncStorage: AsyncStorage,
		      }}>
		      <>{/* awesome app here */}</>
		    </WalletConnectProvider>
		  );
		}
	在上面的代码片段中，除了所需的 props 之外，我们还可以看到 [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx).

	提示：您的自定义选项与此默认配置深度合并。因此，可以覆盖单个嵌套属性，而无需定义所有它们。

- `withWalletConnect	`

	手动使用 [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx)，您可以使用 [withWalletConnect](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/hooks/useWalletConnect.ts) 更高阶的组件，它将 [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx) 为您包装您的根应用程序：
	
		import * as React from "react";
		import {
		  withWalletConnect,
		  useWalletConnect,
		} from "@walletconnect/react-native-dapp";
		import AsyncStorage from "@react-native-async-storage/async-storage";
		
		function App(): JSX.Element {
		  const connector = useWalletConnect(); // valid
		  return <>{/* awesome app here */}</>;
		}
		
		export default withWalletConnect(App, {
		  clientMeta: {
		    description: "Connect with WalletConnect",
		  },
		  redirectUrl:
		    Platform.OS === "web" ? window.location.origin : "yourappscheme://",
		  storageOptions: {
		    asyncStorage: AsyncStorage,
		  },
		});
	这在功能上几乎与手动实现 一个相同 [WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx)，关键区别在于我们可以[useWalletConnect](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/hooks/useWalletConnect.ts) 直接从 `App` 组件中调用。相比之下，在前面的例子中，只有子组件[WalletConnectProvider](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/providers/WalletConnectProvider.tsx) 可能能够调用这个钩子。
- `useWalletConnect`

	该 [useWalletConnect](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/hooks/useWalletConnect.ts) 挂钩提供对可在 Android、iOS 和 Web 上访问的实例的访问。这符合原始规范：[WalletConnect](https://docs.walletconnect.org/client-api) `connector`
	
		import AsyncStorage from '@react-native-async-storage/async-storage';
		import { useWalletConnect, withWalletConnect } from '@walletconnect/react-native-dapp';
		import * as React from 'react';
		
		function App(): JSX.Element {
		  const connector = useWalletConnect();
		  if (!connector.connected) {
		    /**
		     *  链接
		     */
		    return <Button title="Connect" onPress={() => connector.connect()} />;
		  }
		  return <Button title="Kill Session" onPress={() => connector.killSession()} />;
		}
		
		export default withWalletConnect(App, {
		  redirectUrl: Platform.OS === 'web' ? window.location.origin : 'yourappscheme://',
		  storageOptions: {
		    asyncStorage: AsyncStorage,
		  },
		});

### 定制
`@walletconnect/react-native-dapp` 还允许您自定义 [QrcodeModal](https://github.com/WalletConnect/walletconnect-docs/tree/074ef6d866a4790726bc2159d80cdc4f35a969ea/quick-start/dapps/src/components/QrcodeModal.tsx). 这是通过将 [Render Callback](https://docs.walletconnect.com/quick-start/dapps/react-native) 属性 ,传递给我们对  `renderQrcodeModal` 的调用 `withWalletConnect` 或实例来实现的 `WalletConnectProvider`。

例如，可以选择使用 `BottomSheet` 相对于一个来呈现钱包选择 `Modal`：

		import AsyncStorage from '@react-native-async-storage/async-storage';
		import BottomSheet from 'react-native-reanimated-bottom-sheet';
		import { Image, Text, TouchableOpacity } from 'react-native';
		import {
		  useWalletConnect,
		  withWalletConnect,
		  RenderQrcodeModalProps,
		  WalletService,
		} from '@walletconnect/react-native-dapp';
		import * as React from 'react';
		
		function CustomBottomSheet({
		  walletServices,
		  visible,
		  connectToWalletService,
		  uri,
		}: RenderQrcodeModalProps): JSX.Element {
		  const renderContent = React.useCallback(() => {
		    return walletServices.map((walletService: WalletService, i: number) => (
		      <TouchableOpacity key={`i${i}`} onPress={() => connectToWalletService(walletService, uri)}>
		        <Image source={{ uri: walletService.logo }} />
		        <Text>{walletService.name}</Text>
		      </TouchableOpacity>
		    ));
		  }, [walletServices, uri]);
		  return <BottomSheet renderContent={renderContent} {...etc} />;
		};
		
		function App(): JSX.Element {
		  const connector = useWalletConnect();
		  return <>{/* awesome custom app here */}</>;
		}
		
		export default withWalletConnect(App, {
		  redirectUrl: Platform.OS === 'web' ? window.location.origin : 'yourappscheme://',
		  storageOptions: {
		    asyncStorage: AsyncStorage,
		  },
		  renderQrcodeModal: (props: RenderQrcodeModalProps): JSX.Element => (
		    <CustomBottomSheet {...props} />
		  ),
		});

## Web3 Provider
- 安装
	- npm

			npm install --save web3 @walletconnect/web3-provider
	- yarn 

			yarn add web3 @walletconnect/web3-provider

首先，使用以下选项实例化您的 `WalletConnect web3-provider:Infura` 或自定义 RPC 映射

- infura

		import WalletConnectProvider from "@walletconnect/web3-provider";
		
		//  Create WalletConnect Provider
		const provider = new WalletConnectProvider({
		  infuraId: "27e484dcd9e3efcfd25a83a78777cdf1",
		});
		
		//  Enable session (triggers QR Code modal)
		await provider.enable(); 
- 自定义 RPC

		import WalletConnectProvider from "@walletconnect/web3-provider";

		//  Create WalletConnect Provider
		const provider = new WalletConnectProvider({
		  rpc: {
		    1: "https://mainnet.mycustomnode.com",
		    3: "https://ropsten.mycustomnode.com",
		    100: "https://dai.poa.network",
		    // ...
		  },
		});
		
		//  Enable session (triggers QR Code modal)
		await provider.enable();
		
然后你可以使用你最喜欢的以太坊库集成你的 dapp：`ethers.js` 或 `web3.js`

- `ethers.js`

		import { providers } from "ethers";
		
		//  Wrap with Web3Provider from ethers.js
		const web3Provider = new providers.Web3Provider(provider);
- `web3.js`	

		import Web3 from "web3";
		
		//  Create Web3 instance
		const web3 = new Web3(provider);

### 事件 EIP-1193
设置您的提供商后，您应该监听 EIP-1193 事件以检测帐户和链更改以及断开连接。

	// Subscribe to accounts change
	provider.on("accountsChanged", (accounts: string[]) => {
	  console.log(accounts);
	});
	
	// Subscribe to chainId change
	provider.on("chainChanged", (chainId: number) => {
	  console.log(chainId);
	});
	
	// Subscribe to session disconnection
	provider.on("disconnect", (code: number, reason: string) => {
	  console.log(code, reason);
	});
### Provider 模式
	interface RequestArguments {
	  method: string;
	  params?: unknown[] | object;
	}
	
	// Send JSON RPC requests
	const result = await provider.request(payload: RequestArguments);
	
	// Close provider session
	await provider.disconnect()
### web3 模式
	//  Get Accounts
	const accounts = await web3.eth.getAccounts();
	
	//  Get Chain Id
	const chainId = await web3.eth.chainId();
	
	//  Get Network Id
	const networkId = await web3.eth.net.getId();
	
	// Send Transaction
	const txHash = await web3.eth.sendTransaction(tx);
	
	// Sign Transaction
	const signedTx = await web3.eth.signTransaction(tx);
	
	// Sign Message
	const signedMessage = await web3.eth.sign(msg);
	
	// Sign Typed Data
	const signedTypedData = await web3.eth.signTypedData(msg);
### 提供者选项(Provider Options)
- 必选

	为了解决非签名请求，您需要提供以下内容之一：
	
	- Infura ID

		infuraId 将支持以下链 ID： Mainnet (1), Ropsten (3), Rinkeby(4), Goerli (5) and Kovan (42)

			const provider = new WalletConnectProvider({
			  infuraId: "27e484dcd9e3efcfd25a83a78777cdf1",
			});
	- RPC URL Mapping

		RPC URL 映射应由 chainId 索引，并且至少需要一个值。

			const provider = new WalletConnectProvider({
			  rpc: {
			    1: "https://mainnet.mycustomnode.com",
			    3: "https://ropsten.mycustomnode.com",
			    100: "https://dai.poa.network",
			    // ...
			  },
			});
- 可选

	您还可以使用以下选项通过提供程序自定义连接器

	- 桥接

		通过提供 url 使用您自己的托管网桥
		
			const provider = new WalletConnectProvider({
			  infuraId: "27e484dcd9e3efcfd25a83a78777cdf1",
			  bridge: "https://bridge.myhostedserver.com",
			});
	- 禁用二维码

		使用您自己的自定义二维码模式并禁用内置的
		
			const provider = new WalletConnectProvider({
			  infuraId: "27e484dcd9e3efcfd25a83a78777cdf1",
			  qrcode: false,
			});
			
			provider.connector.on("display_uri", (err, payload) => {
			  const uri = payload.params[0];
			  CustomQRCodeModal.display(uri);
			});
	- 过滤移动链接

		如果您想减少移动链接选项的数量或自定义其顺序，您可以提供一系列钱包名称
		
			const provider = new WalletConnectProvider({
			  infuraId: "27e484dcd9e3efcfd25a83a78777cdf1",
			  qrcodeModalOptions: {
			    mobileLinks: [
			      "rainbow",
			      "metamask",
			      "argent",
			      "trust",
			      "imtoken",
			      "pillar",
			    ],
			  },
			});				
				

												
									  