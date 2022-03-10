# walletconnect-快速入门-Wallet
## Kotlin 客户端 (Android)
[代码库](https://github.com/WalletConnect/kotlin-walletconnect-lib)
## Swift 客户端 (iOS)
目前有两个 swift 客户端可用：

- 由 [Trust Wallet](https://trustwallet.com/) 团队维护的 [wallet-connect-swift](https://github.com/WalletConnect/wallet-connect-swift)
- 由 [Gnosis Safe](https://gnosis-safe.io/) 团队维护的 [WalletConnectSwift](https://github.com/WalletConnect/WalletConnectSwift)

## React-Native 客户端
您可以在 [example.walletconnect.org](https://example.walletconnect.org/) 使用示例 Dapp测试您的集成 [源代码](https://github.com/WalletConnect/walletconnect-example-dapp)
### 安装
- npm

		npm install --save @walletconnect/client
- yarn

		yarn add @walletconnect/client

用于 React-Native 的 Polyfill NodeJS 模块

- npm

		npm install --save rn-nodeify
		rn-nodeify --install --hack
- yarn

		yarn add rn-nodeify
		rn-nodeify --install --hack
		
### 初始化链接
	import WalletConnect from "@walletconnect/client";
	
	// Create connector
	const connector = new WalletConnect(
	  {
	    // Required
	    uri: "wc:8a5e5bdc-a0e4-47...TJRNmhWJmoxdFo6UDk2WlhaOyQ5N0U=",
	    // Required
	    clientMeta: {
	      description: "WalletConnect Developer App",
	      url: "https://walletconnect.org",
	      icons: ["https://walletconnect.org/walletconnect-logo.png"],
	      name: "WalletConnect",
	    },
	  },
	  {
	    // Optional
	    url: "<YOUR_PUSH_SERVER_URL>",
	    type: "fcm",
	    token: token,
	    peerMeta: true,
	    language: language,
	  }
	);
	
	// Subscribe to session requests
	connector.on("session_request", (error, payload) => {
	  if (error) {
	    throw error;
	  }
	
	  // Handle Session Request
	
	  /* payload:
	  {
	    id: 1,
	    jsonrpc: '2.0'.
	    method: 'session_request',
	    params: [{
	      peerId: '15d8b6a3-15bd-493e-9358-111e3a4e6ee4',
	      peerMeta: {
	        name: "WalletConnect Example",
	        description: "Try out WalletConnect v1.0",
	        icons: ["https://example.walletconnect.org/favicon.ico"],
	        url: "https://example.walletconnect.org"
	      }
	    }]
	  }
	  */
	});
	
	// Subscribe to call requests
	connector.on("call_request", (error, payload) => {
	  if (error) {
	    throw error;
	  }
	
	  // Handle Call Request
	
	  /* payload:
	  {
	    id: 1,
	    jsonrpc: '2.0'.
	    method: 'eth_sign',
	    params: [
	      "0xbc28ea04101f03ea7a94c1379bc3ab32e65e62d3",
	      "My email is john@doe.com - 1537836206101"
	    ]
	  }
	  */
	});
	
	connector.on("disconnect", (error, payload) => {
	  if (error) {
	    throw error;
	  }
	
	  // Delete connector
	});
### 管理连接
	// Approve Session
	connector.approveSession({
	  accounts: [                 // required
	    '0x4292...931B3',
	    '0xa4a7...784E8',
	    ...
	  ],
	  chainId: 1                  // required
	})
	
	// Reject Session
	connector.rejectSession({
	  message: 'OPTIONAL_ERROR_MESSAGE'       // optional
	})
	
	
	// Kill Session
	connector.killSession()
### 管理调用请求	
	// Approve Call Request
	connector.approveRequest({
	  id: 1,
	  result: "0x41791102999c339c844880b23950704cc43aa840f3739e365323cda4dfa89e7a"
	});
	
	// Reject Call Request
	connector.rejectRequest({
	  id: 1,                                  // required
	  error: {
	    code: "OPTIONAL_ERROR_CODE"           // optional
	    message: "OPTIONAL_ERROR_MESSAGE"     // optional
	  }
	});
				

