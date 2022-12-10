## 代码 completion
### 介绍
[Codex 模型](https://beta.openai.com/docs/models/codex)系列是我们的 [GPT-3 系列](https://beta.openai.com/docs/models/base-series)的后代，该系列已经过自然语言和数十亿行代码的训练。它最擅长 Python，精通 JavaScript、Go、Perl、PHP、Ruby、Swift、TypeScript、SQL，甚至 Shell 等十几种语言。在最初的有限测试期间，Codex 使用是免费的。[了解更多](https://beta.openai.com/docs/models/codex)。

您可以将 Codex 用于各种任务，包括：

- 将注释变成代码
- 在上下文中完成你的下一行或函数
- 为您带来知识，例如为应用程序查找有用的库或 API 调用
- 添加评论
- 重写代码以提高效率

要查看 Codex 的运行情况，请查看我们的 [Codex JavaScript沙箱](https://beta.openai.com/codex-javascript-sandbox) 或我们的其他[演示视频](https://www.youtube.com/playlist?list=PLOXw6I10VTv_FhQbbvYh1FvbiaPf43Ve2)。

#### Codex JavaScript 沙箱
此示例应用程序使用 Codex 将自然语言指令翻译成 JavaScript。

![](./pic/openai-sendbox.png)

- [试试看](https://beta.openai.com/codex-javascript-sandbox)
- [视频](https://www.youtube.com/watch?v=Zm9B-DvwOgw&list=PLOXw6I10VTv_FhQbbvYh1FvbiaPf43Ve2&index=2)

### 快速开始
要自己开始使用 Codex，请尝试在 [Playground](https://beta.openai.com/playground) 中打开这些示例。

- 说“你好”（Python）

		"""
		Ask the user for their name and say "Hello"
		"""

		“name = input("What is your name? ")”<- AI 补充
		”print("Hello " + name)”<- AI 补充

	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%22%22%22%0AAsk%20the%20user%20for%20their%20name%20and%20say%20%22Hello%22%0A%22%22%22)
- 创建随机名称 (Python)

		"""
		1. Create a list of first names
		2. Create a list of last names
		3. Combine them randomly into a list of 100 full names
		"""
		
		--AI 生成--
		first_names = ["John", "Jane", "Corey", "Travis", "Dave", "Kurt", "Neil", "Sam", "Steve", "Tom", "James", "Robert", "Michael", "Charles", "Joe", "Mary", "Maggie", "Nicole", "Patricia", "Linda", "Barbara", "Elizabeth", "Laura", "Jennifer", "Maria"]
		last_names = ["Smith", "Doe", "Jenkins", "Robinson", "Davis", "Stuart", "Jefferson", "Jacobs", "Wright", "Patterson", "Wilks", "Arnold", "Johnson", "Williams", "Jones", "Brown", "Davis", "Miller", "Wilson", "Moore", "Taylor", "Anderson", "Thomas", "Jackson", "White", "Harris", "Martin"]
		full_names = []
		for i in range(100):
			full_names.append(random.choice(first_names) + " " + random.choice(last_names))
		return full_names

		def get_random_email(self):
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%22%22%22%0A1.%20Create%20a%20list%20of%20first%20names%0A2.%20Create%20a%20list%20of%20last%20names%0A3.%20Combine%20them%20randomly%20into%20a%20list%20of%20100%20full%20names%0A%22%22%22)

- 创建 MySQL 查询 (Python)

		"""
		Table customers, columns = [CustomerId, FirstName, LastName, Company, Address, City, State, Country, PostalCode, Phone, Fax, Email, SupportRepId]
		Create a MySQL query for all customers in Texas named Jane
		"""
		
		-- AI 生成 --
		query = "SELECT * FROM customers WHERE State = 'Texas' AND FirstName = 'Jane'"
		self.cursor.execute(query)
		result = self.cursor.fetchall()
		return result

		def query_all_customers_in_Texas_named_Jane_or_John(self):
			"""
			Table customers, columns = [CustomerId, FirstName, LastName, Company, Address, City, State, Country, PostalCode, Phone, Fax, Email, SupportRepId]
			Create a MySQL query for all customers in Texas named Jane or John
			"""
			query = "SELECT * FROM customers WHERE State = 'Texas' AND (FirstName = 'Jane' OR FirstName = 'John')"
			self.cursor.execute(query)
			result = self.cursor.fetchall()
			return result
	
		def query_all_customers_in_Texas_named_Jane_or_John_and_live_in_Houston(self):
			"""
			Table customers, columns = [CustomerId, FirstName, LastName
	
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%22%22%22%0ATable%20customers%2C%20columns%20%3D%20%5BCustomerId%2C%20FirstName%2C%20LastName%2C%20Company%2C%20Address%2C%20City%2C%20State%2C%20Country%2C%20PostalCode%2C%20Phone%2C%20Fax%2C%20Email%2C%20SupportRepId%5D%0ACreate%20a%20MySQL%20query%20for%20all%20customers%20in%20Texas%20named%20Jane%0A%22%22%22%0Aquery%20%3D)
- 解释代码 (JavaScript)

		// Function 1
		var fullNames = [];
		for (var i = 0; i < 50; i++) {
		  fullNames.push(names[Math.floor(Math.random() * names.length)]
		    + " " + lastNames[Math.floor(Math.random() * lastNames.length)]);
		}
		
		// What does Function 1 do?
		
		-- AI 生成-- 
		// Function 1 creates an array of 50 random full names.
		
		// Function 2
		var ages = [];
		for (var i = 0; i < 50; i++) {
		  ages.push(Math.floor(Math.random() * 100));
		}
		
		// What does Function 2 do?
		// Function 2 creates an array of 50 random ages.
		
		// Function 3
		var people = [];
		for (var i = 0; i < 50; i++) {
		  people.push({
		    name: fullNames[i],
		    age: ages[i]
		  });
		}
		
		// What does Function 3 do?
		// Function 3 creates an array of 50 objects with a name and age property.
		
		// Function 4
		var peopleUnder50 = [];
		for (var i = 0; i < people.length; i++) {
		  if (people[i].age < 50) {
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2F%20Function%201%0Avar%20fullNames%20%3D%20%5B%5D%3B%0Afor%20%28var%20i%20%3D%200%3B%20i%20%3C%2050%3B%20i%2B%2B%29%20%7B%0A%20%20fullNames.push%28names%5BMath.floor%28Math.random%28%29%20%2A%20names.length%29%5D%0A%20%20%20%20%2B%20%22%20%22%20%2B%20lastNames%5BMath.floor%28Math.random%28%29%20%2A%20lastNames.length%29%5D%29%3B%0A%7D%0A%0A%2F%2F%20What%20does%20Function%201%20do%3F)

#### 更多例子
访问我们的 [示例库](https://beta.openai.com/examples?category=code) ，探索更多为 Codex 设计的提示。

### 最佳实践
从注释、数据或代码开始。

您可以在我们的 playground 中尝试使用其中一种 Codex 模型（需要时将样式说明作为注释。）

为了让 Codex 创建有用的完成，考虑程序员执行任务需要哪些信息会很有帮助。这可能只是一个清晰的注释或编写有用函数所需的数据，例如变量名称或函数处理的类。

	# Create a function called 'nameImporter' to add a first and last name to the database
	
	-- AI --
	def nameImporter(firstName, lastName):
	    # Create a new entry in the database
	    newName = Name(firstName=firstName, lastName=lastName)
	    # Add the new entry to the session
	    session.add(newName)
	    # Commit the session to the database
	    session.commit()
	
	# Create a function called 'nameExporter' to return all the names in the database
	def nameExporter():
	    # Query the database for all the names
	    names = session.query(Name).all()
	    # Return the names
	    return names
	
	# Create a function called 'nameDeleter' to delete a name from the database
	def nameDeleter(firstName, lastName):
	    # Query the database for the name
	    name = session.query(Name).filter_by(firstName=firstName, lastName=lastName).one()
	    # Delete the name from the session
	    session.delete(name)
	    # Commit the session to the database
	    session.commit()
	
	# Create a function called 'nameUpdater' to update a name in the database
[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Create%20a%20function%20called%20%27nameImporter%27%20to%20add%20a%20first%20and%20last%20name%20to%20the%20database)

在此示例中，我们告诉 Codex 调用该函数的内容以及它将执行的任务。

这种方法甚至可以扩展到您可以向 Codex 提供注释和数据库模式示例，以使其为各种数据库编写有用的查询请求。

	# Table albums, columns = [AlbumId, Title, ArtistId]
	# Table artists, columns = [ArtistId, Name]
	# Table media_types, columns = [MediaTypeId, Name]
	# Table playlists, columns = [PlaylistId, Name]
	# Table playlist_track, columns = [PlaylistId, TrackId]
	# Table tracks, columns = [TrackId, Name, AlbumId, MediaTypeId, GenreId, Composer, Milliseconds, Bytes, UnitPrice]
	
	# Create a query for all albums by Adele
	
	-- AI --
	
	# Create a query for all tracks by Adele.
	# Create a query for all tracks by Adele that are longer than 3 minutes.
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25".
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25" or "21".
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25" or "21" and are in the genre "Pop".
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25" or "21" and are in the genre "Pop" and are in the media type "MPEG audio file".
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25" or "21" and are in the genre "Pop" and are in the media type "MPEG audio file" and are in the playlist "Music".
	# Create a query for all tracks by Adele that are longer than 3 minutes and are in the album "25" or "
[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Table%20albums%2C%20columns%20%3D%20%5BAlbumId%2C%20Title%2C%20ArtistId%5D%0A%23%20Table%20artists%2C%20columns%20%3D%20%5BArtistId%2C%20Name%5D%0A%23%20Table%20media_types%2C%20columns%20%3D%20%5BMediaTypeId%2C%20Name%5D%0A%23%20Table%20playlists%2C%20columns%20%3D%20%5BPlaylistId%2C%20Name%5D%0A%23%20Table%20playlist_track%2C%20columns%20%3D%20%5BPlaylistId%2C%20TrackId%5D%0A%23%20Table%20tracks%2C%20columns%20%3D%20%5BTrackId%2C%20Name%2C%20AlbumId%2C%20MediaTypeId%2C%20GenreId%2C%20Composer%2C%20Milliseconds%2C%20Bytes%2C%20UnitPrice%5D%0A%0A%23%20Create%20a%20query%20for%20all%20albums%20by%20Adele)

当您向 Codex 显示数据库架构时，它能够对如何格式化查询做出有根据的猜测。

- 指定语言

	Codex 了解数十种不同的编程语言。许多人对注释、函数和其他编程语法有着相似的约定。通过在评论中指定语言和版本，Codex 能够更好地完成您想要的内容。也就是说，Codex 在风格和语法方面相当灵活。

		# R language
		# Calculate the mean distance between an array of points
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20R%20language%0A%23%20Calculate%20the%20mean%20distance%20between%20an%20array%20of%20points)
	
		# Python 3
		# Calculate the mean distance between an array of points
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Python%203%0A%23%20Calculate%20the%20mean%20distance%20between%20an%20array%20of%20points)

	提示 Codex 你想要它做什么。如果您希望 Codex 创建网页，请将第一行代码放在 HTML 文档 ( `<!DOCTYPE html>`) 之后，您的评论告诉 Codex 下一步应该做什么。`func` 相同的方法适用于从注释创建函数（在注释后以 `or` 开头的新行 `def` ）。

		<!-- Create a web page with the title 'Kat Katman attorney at paw' -->
		<!DOCTYPE html>
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%3C%21--%20Create%20a%20web%20page%20with%20the%20title%20%27Kat%20Katman%20attorney%20at%20paw%27%20--%3E%0A%3C%21DOCTYPE%20html%3E)
	放在 `<!DOCTYPE html>` 我们的评论之后可以让 Codex 清楚地知道我们希望它做什么。

		# Create a function to count to 100

		def counter
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Create%20a%20function%20to%20count%20to%20100%0A%0Adef%20counter)
	如果我们开始编写函数，Codex 将了解下一步需要做什么。

- 指定库将帮助 Codex 了解您的需求。

	Codex 知道大量的库、API 和模块。通过通过注释或将它们导入您的代码告诉 Codex 使用哪些，Codex 将根据它们而不是备选方案提出建议。

		<!-- Use A-Frame version 1.2.0 to create a 3D website -->
		<!-- https://aframe.io/releases/1.2.0/aframe.min.js -->
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%3C%21--%20Use%20A-Frame%20version%201.2.0%20to%20create%20a%203D%20website%20--%3E%0A%3C%21--%20https%3A%2F%2Faframe.io%2Freleases%2F1.2.0%2Faframe.min.js%20--%3E)
	
	通过指定版本，您可以确保 Codex 使用最新的库。

	注意：Codex 可以推荐有用的库和 API，但请始终确保自己进行研究以确保它们对您的应用程序是安全的。
- 注释风格会影响代码质量。

	对于某些语言，注释的风格可以提高输出的质量。例如，在使用 Python 时，在某些情况下使用文档字符串（用三引号引起来的注释）可以提供比使用井号 (#) 符号更高质量的结果。

		"""
		Create an array of users and email addresses
		"""
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%22%22%22%0ACreate%20an%20array%20of%20users%20and%20email%20addresses%0A%22%22%22)
- 将注释放在函数内部可能会有所帮助。

	推荐的编码标准通常建议将函数的描述放在函数内部。使用这种格式有助于 Codex 更清楚地了解您希望函数执行的操作。

		def getUserBalance(id):
		    """
		    Look up the user in the database ‘UserData' and return their current account balance.
		    """
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=def%20getUserBalance%28id%29%3A%0A%20%20%20%20%22%22%22%0A%20%20%20%20Look%20up%20the%20user%20in%20the%20database%20%E2%80%98UserData%27%20and%20return%20their%20current%20account%20balance.%0A%20%20%20%20%22%22%22)
- 提供示例以获得更精确的结果。

	如果您有需要 Codex 使用的特定样式或格式，请在请求的第一部分提供示例或演示，这将有助于 Codex 更准确地满足您的需求。

		"""
		Create a list of random animals and species
		"""
		animals  = [ {"name": "Chomper", "species": "Hamster"}, {"name":
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%22%22%22%0ACreate%20a%20list%20of%20random%20animals%20and%20species%0A%22%22%22%0Aanimals%20%20%3D%20%5B%20%7B%22name%22%3A%20%22Chomper%22%2C%20%22species%22%3A%20%22Hamster%22%7D%2C%20%7B%22name%22%3A)
- 较低的温度可提供更精确的结果。

	在大多数情况下，将 API 温度设置为 0 或接近于零（例如 0.1 或 0.2）往往会产生更好的结果。与 GPT-3 不同，更高的温度可以提供有用的创造性和随机结果，Codex 的更高温度可能会给你真正随机或不稳定的响应。

	如果您需要 Codex 提供不同的潜在结果，请从零开始，然后向上递增 0.1，直到找到合适的变化。
- 将任务组织成功能。

	我们可以通过在注释中尽可能准确地指定函数应该做什么来让 Codex 编写函数。通过编写以下注释，Codex 创建了一个 Javascript 计时器函数，该函数在用户按下按钮时触发：

	- 一个简单的 JavaScript 计时器

			// Create a timer that creates an alert in 10 seconds
		[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2F%20Create%20a%20timer%20that%20creates%20an%20alert%20in%2010%20seconds)
	- 我们可以使用 Codex 与众所周知的库一起执行常见任务，例如使用 Stripe API 创建客户：

		在 Python 中创建 Stripe 客户
	
			# Create a Stripe customer from an email address
		[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Create%20a%20Stripe%20customer%20from%20an%20email%20address)
- 创建示例数据

	测试应用程序通常需要使用示例数据。因为 Codgen 是一种理解如何理解和编写自然语言的语言模型，所以您可以要求 Codex 创建数据，例如虚构名称、产品和其他变量的数组。

		/* Create an array of weather temperatures for San Francisco */
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2A%20Create%20an%20array%20of%20weather%20temperatures%20for%20San%20Francisco%20%2A%2F)
	要求 Codex 执行此任务将生成如下表格：

		var weather = [
		  { month: 'January', high: 58, low: 48 },
		  { month: 'February', high: 61, low: 50 },
		  { month: 'March', high: 64, low: 53 },
		  { month: 'April', high: 67, low: 55 },
		  { month: 'May', high: 70, low: 58 },
		  { month: 'June', high: 73, low: 61 },
		  { month: 'July', high: 76, low: 63 },
		  { month: 'August', high: 77, low: 64 },
		  { month: 'September', high: 76, low: 63 },
		  { month: 'October', high: 73, low: 61 },
		  { month: 'November', high: 68, low: 57 },
		  { month: 'December', high: 64, low: 54 }
		];
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=var%20weather%20%3D%20%5B%0A%20%20%7B%20month%3A%20%27January%27%2C%20high%3A%2058%2C%20low%3A%2048%20%7D%2C%0A%20%20%7B%20month%3A%20%27February%27%2C%20high%3A%2061%2C%20low%3A%2050%20%7D%2C%0A%20%20%7B%20month%3A%20%27March%27%2C%20high%3A%2064%2C%20low%3A%2053%20%7D%2C%0A%20%20%7B%20month%3A%20%27April%27%2C%20high%3A%2067%2C%20low%3A%2055%20%7D%2C%0A%20%20%7B%20month%3A%20%27May%27%2C%20high%3A%2070%2C%20low%3A%2058%20%7D%2C%0A%20%20%7B%20month%3A%20%27June%27%2C%20high%3A%2073%2C%20low%3A%2061%20%7D%2C%0A%20%20%7B%20month%3A%20%27July%27%2C%20high%3A%2076%2C%20low%3A%2063%20%7D%2C%0A%20%20%7B%20month%3A%20%27August%27%2C%20high%3A%2077%2C%20low%3A%2064%20%7D%2C%0A%20%20%7B%20month%3A%20%27September%27%2C%20high%3A%2076%2C%20low%3A%2063%20%7D%2C%0A%20%20%7B%20month%3A%20%27October%27%2C%20high%3A%2073%2C%20low%3A%2061%20%7D%2C%0A%20%20%7B%20month%3A%20%27November%27%2C%20high%3A%2068%2C%20low%3A%2057%20%7D%2C%0A%20%20%7B%20month%3A%20%27December%27%2C%20high%3A%2064%2C%20low%3A%2054%20%7D%0A%5D%3B)
- 复合函数和小型应用程序。

	我们可以向 Codex 提供包含复杂请求的评论，例如创建随机名称生成器或使用用户输入执行任务，如果有足够的 token ，Codex 可以生成其余内容。

		/*
		Create a list of animals
		Create a list of cities
		Use the lists to generate stories about what I saw at the zoo in each city
		*/
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2A%0ACreate%20a%20list%20of%20animals%0ACreate%20a%20list%20of%20cities%0AUse%20the%20lists%20to%20generate%20stories%20about%20what%20I%20saw%20at%20the%20zoo%20in%20each%20city%0A%2A%2F)
- 限制完成大小以获得更精确的结果或更低的延迟。

	在 Codex 中请求更长的完成时间可能会导致不准确的答案和重复。通过减少 `max_tokens` 和设置 `stop` 标记来限制查询的大小。例如，添加 `\n` 为 `stop` 序列以将完成限制为一行代码。较小的完成也会导致较少的延迟。
- 使用流式传输来减少延迟。

	大型 Codex 查询可能需要数十秒才能完成。要构建需要较低延迟的应用程序，例如执行自动完成的编码助手，请考虑使用流式处理。在模型完成生成整个完成之前将返回响应。只需要部分完成的应用程序可以通过以编程方式或通过使用创造性的值来切断完成来减少延迟 `stop`。

	用户可以通过从 API 请求多个解决方案并使用返回的第一个响应，将流式处理与复制结合起来以减少延迟。通过设置来做到这一点 `n > 1`。这种方式会消耗较多的 token 配额，谨慎使用（比如合理设置 `max_tokens` 和 `stop`）。
- 使用 Codex 来解释代码。

	Codex 创建和理解代码的能力使我们能够使用它来执行任务，例如解释文件中代码的作用。实现此目的的一种方法是在以“This function”或“This application is”开头的函数之后添加注释。Codex 通常将此解释为解释的开始并完成文本的其余部分。

		/* Explain what the previous function is doing: It
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2A%20Explain%20what%20the%20previous%20function%20is%20doing%3A%20It)
- 解释 SQL 查询

	在此示例中，我们使用 Codex 以人类可读的格式解释 SQL 查询的作用。

		SELECT DISTINCT department.name
		FROM department
		JOIN employee ON department.id = employee.department_id
		JOIN salary_payments ON employee.id = salary_payments.employee_id
		WHERE salary_payments.date BETWEEN '2020-06-01' AND '2020-06-30'
		GROUP BY department.name
		HAVING COUNT(employee.id) > 10;
		-- Explanation of the above query in human readable format
		--
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=SELECT%20DISTINCT%20department.name%0AFROM%20department%0AJOIN%20employee%20ON%20department.id%20%3D%20employee.department_id%0AJOIN%20salary_payments%20ON%20employee.id%20%3D%20salary_payments.employee_id%0AWHERE%20salary_payments.date%20BETWEEN%20%272020-06-01%27%20AND%20%272020-06-30%27%0AGROUP%20BY%20department.name%0AHAVING%20COUNT%28employee.id%29%20%3E%2010%3B%0A--%20Explanation%20of%20the%20above%20query%20in%20human%20readable%20format%0A--)
- 编写单元测试

	只需添加注释“单元测试”并启动一个函数，即可在 Python 中创建单元测试。

		# Python 3
		def sum_numbers(a, b):
		  return a + b
		
		# Unit test
		def
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Python%203%0Adef%20sum_numbers%28a%2C%20b%29%3A%0A%20%20return%20a%20%2B%20b%0A%0A%23%20Unit%20test%0Adef)
- 检查代码是否有错误

	通过使用示例，您可以向 Codex 展示如何识别代码中的错误。在某些情况下不需要示例，但是展示提供描述的级别和细节可以帮助法典理解要寻找的内容以及如何解释它。（Codex 对错误的检查不应取代用户的仔细审查。）

		/* Explain why the previous function doesn't work. */
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2A%20Explain%20why%20the%20previous%20function%20doesn%27t%20work.%20%2A%2F)
- 使用源数据编写数据库函数。

	正如人类程序员会从了解数据库结构和列名中获益一样，Codex 可以使用此数据来帮助您编写准确的查询请求。在此示例中，我们插入数据库的模式并告诉 Codex 要查询数据库的内容。

		# Table albums, columns = [AlbumId, Title, ArtistId]
		# Table artists, columns = [ArtistId, Name]
		# Table media_types, columns = [MediaTypeId, Name]
		# Table playlists, columns = [PlaylistId, Name]
		# Table playlist_track, columns = [PlaylistId, TrackId]
		# Table tracks, columns = [TrackId, Name, AlbumId, MediaTypeId, GenreId, Composer, Milliseconds, Bytes, UnitPrice]
		
		# Create a query for all albums by Adele
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Table%20albums%2C%20columns%20%3D%20%5BAlbumId%2C%20Title%2C%20ArtistId%5D%0A%23%20Table%20artists%2C%20columns%20%3D%20%5BArtistId%2C%20Name%5D%0A%23%20Table%20media_types%2C%20columns%20%3D%20%5BMediaTypeId%2C%20Name%5D%0A%23%20Table%20playlists%2C%20columns%20%3D%20%5BPlaylistId%2C%20Name%5D%0A%23%20Table%20playlist_track%2C%20columns%20%3D%20%5BPlaylistId%2C%20TrackId%5D%0A%23%20Table%20tracks%2C%20columns%20%3D%20%5BTrackId%2C%20Name%2C%20AlbumId%2C%20MediaTypeId%2C%20GenreId%2C%20Composer%2C%20Milliseconds%2C%20Bytes%2C%20UnitPrice%5D%0A%0A%23%20Create%20a%20query%20for%20all%20albums%20by%20Adele)
- 语言之间的转换。

	您可以让 Codex 从一种语言转换为另一种语言，方法是遵循一种简单的格式，您可以在注释中列出要转换的代码的语言，然后是代码，然后是包含您希望将其翻译成的语言的注释。

		# Convert this from Python to R
		# Python version
		
		[ Python code ]
		
		# End
		
		# R version
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%23%20Convert%20this%20from%20Python%20to%20R%0A%23%20Python%20version%0A%0A%5B%20Python%20code%20%5D%0A%0A%23%20End%0A%0A%23%20R%20version)
- 为库或框架重写代码

	如果您希望 Codex 提高某个功能的效率，您可以为其提供要重写的代码，然后提供有关使用何种格式的说明。

		// Rewrite this as a React component
		var input = document.createElement('input');
		input.setAttribute('type', 'text');
		document.body.appendChild(input);
		var button = document.createElement('button');
		button.innerHTML = 'Say Hello';
		document.body.appendChild(button);
		button.onclick = function() {
		  var name = input.value;
		  var hello = document.createElement('div');
		  hello.innerHTML = 'Hello ' + name;
		  document.body.appendChild(hello);
		};
		
		// React version:
	[在游乐场打开](https://beta.openai.com/playground?model=code-davinci-002&prompt=%2F%2F%20Rewrite%20this%20as%20a%20React%20component%0Avar%20input%20%3D%20document.createElement%28%27input%27%29%3B%0Ainput.setAttribute%28%27type%27%2C%20%27text%27%29%3B%0Adocument.body.appendChild%28input%29%3B%0Avar%20button%20%3D%20document.createElement%28%27button%27%29%3B%0Abutton.innerHTML%20%3D%20%27Say%20Hello%27%3B%0Adocument.body.appendChild%28button%29%3B%0Abutton.onclick%20%3D%20function%28%29%20%7B%0A%20%20var%20name%20%3D%20input.value%3B%0A%20%20var%20hello%20%3D%20document.createElement%28%27div%27%29%3B%0A%20%20hello.innerHTML%20%3D%20%27Hello%20%27%20%2B%20name%3B%0A%20%20document.body.appendChild%28hello%29%3B%0A%7D%3B%0A%0A%2F%2F%20React%20version%3A)

### 插入代码测试版
完成端点还支持通过提供除[前缀提示](https://beta.openai.com/docs/api-reference/completions/create#completions/create-prompt)之外的[后缀提示](https://beta.openai.com/docs/api-reference/completions/create#completions/create-suffix)来在代码中插入代码。这可用于在函数或文件中间插入补全。

	def get_largest_prime_factor(n):
	    if n < 2:
	        return False
	    def “is_prime(n): >  for i in range(2, n): >  if n % i == 0: >  return False >  return True >     largest = 1” < ai 插入
	    for j in range(2, n + 1):
	        if n % j == 0 and is_prime(j):
	    return largest

通过为模型提供额外的上下文，它可以更加可控。然而，这对模型来说是一个更具约束性和挑战性的任务。
#### 最佳实践
插入代码是测试版中的一项新功能，您可能需要修改使用 API 的方式以获得更好的结果。以下是一些最佳实践：

- 使用 `max_tokens > 256`

	该模型更擅长插入更长的补全。如果 max_tokens 太小，模型可能会在连接到后缀之前被切断。请注意，即使使用更大的 max_tokens，您也只会根据生成的 token 数量付费。
- 更喜欢 `finish_reason == “stop”`。

	当模型到达自然停止点或用户提供的停止序列时，它会将 `finish_reason` 设置为“stop”。这表明该模型已设法很好地连接到后缀，并且是完成质量的良好信号。这对于在使用 `n > 1` 或重采样时在几个完成之间进行选择尤其相关（请参阅下一点）。
- 重新采样 3-5 次

	虽然几乎所有补全都连接到前缀，但在更困难的情况下，模型可能难以连接后缀。我们发现，在这种情况下，重采样 3 次或 5 次（或使用 k=3,5 的 best_of）并选择带有“stop”作为其 `finish_reason` 的样本可能是一种有效的方法。重采样时，您通常需要更高的温度来增加多样性。

	注意：如果所有返回的样本都有 `finish_reason == "length`，很可能是 `max_tokens` 太小，模型在设法自然地连接提示和后缀之前用完了标记。考虑在重采样之前增加 `max_tokens`。

### 编辑代码测试版
[edits](https://beta.openai.com/docs/api-reference/edits) 端点可用于编辑代码，而不仅仅是完成代码。您提供一些代码和如何修改它的说明，`code-davinci-edit-001` 模型将尝试相应地对其进行编辑。这是重构和调整代码的自然界面。在此初始测试期间，编辑端点的使用是免费的。
#### 例子
- 迭代构建程序

	编写代码通常是一个迭代过程，需要一路完善文本。编辑自然而然地不断完善模型的输出，直到最终结果得到完善。在此示例中，我们使用斐波那契作为如何迭代构建代码的示例。
- 1. 写一个函数
		- 输入
			
				无输入
		- 指示

				用 python 写一个计算斐波那契的函数
		- 输出

				def fibonacci(num):
					    a = 0
					    b = 1
					    if num ==1:
					       print(a)
					    else:
					        print(a)
					        print(b)
					        #the sequence starts with 0,1
					        for i in range(2,num):
					            c = a+b
					            a = b
					            b = c
					            print(c)
					
					fibonacci(10)
- 2.  重构它
	- 输入

			 if num ==1:
			        print(a)
			    else:
			        print(a)
			        print(b)
			        #the sequence starts with 0,1
			        for i in range(2,num):
			            c = a+b
			            a = b
			            b = c
			            print(c)
			
			fibonacci(10)
	- 指示
	
		将此函数重构为递归的
	- 输出
	
			def fibonacci(num):
			    if num <= 1:
			        return num
			    else:
			        return fib(num-1) + fib(num-2)
			print(fibonacci(10))
- 3. 重命名函数
	- 输入

			def fibonacci(num):
			    if num <= 1:
			        return num
			    else:
			        return fib(num-1) + fib(num-2)
			print(fibonacci(10))
	- 指示

		将函数重命名为 fib
	- 输出

			def fib(num):
			    if num <= 1:
			          return num
			      else:
			        return fib(num-1) + fib(num-2)
			print(fib(10))
- 4. 添加文档
	- 输入

			def fib(num):
			    if num <= 1:
			        return num
			    else:
			        return fib(num-1) + fib(num-2)
			print(fib(10))
- 指示

	添加文档
- 输出

		def fib(num):
		    """ Print the nth number in the fibonacci sequence """
		    if num <= 1:
		          return num
		      else:
		          return fib(num-1) + fib(num-2)
		  print(fib(10))
  
#### 最佳实践
[edits](https://beta.openai.com/docs/api-reference/edits) 端点仍处于 alpha 阶段，我们建议遵循这些最佳实践。

- 考虑使用空提示！在这种情况下，编辑可以类似于完成使用。
- 说明尽可能具体。
- 有时，模型无法找到解决方案并会导致错误。我们建议改写您的说明或输入。
