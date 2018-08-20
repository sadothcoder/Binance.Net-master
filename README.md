
A .Net wrapper for the Binance API as described on [Binance](https://www.binance.com/restapipub.html), including all features the API provides using clear and readable objects.


## Installation
![Nuget version](https://img.shields.io/nuget/v/binance.net.svg)  ![Nuget downloads](https://img.shields.io/nuget/dt/Binance.Net.svg)
Available on [Nuget](https://www.nuget.org/packages/Binance.Net/).
```
pm> Install-Package Binance.Net
```
To get started with Binance.Net first you will need to get the library itself. The easiest way to do this is to install the package into your project using  [NuGet](https://www.nuget.org/packages/Binance.Net/). Using Visual Studio this can be done in two ways.

### Using the package manager
In Visual Studio right click on your solution and select 'Manage NuGet Packages for solution...'. A screen will appear which initially shows the currently installed packages. In the top bit select 'Browse'. This will let you download net package from the NuGet server. In the search box type 'Binance.Net' and hit enter. The Binance.Net package should come up in the results. After selecting the package you can then on the right hand side select in which projects in your solution the package should install. After you've selected all project you wish to install and use Binance.Net in hit 'Install' and the package will be downloaded and added to you projects.

### Using the package manager console
In Visual Studio in the top menu select 'Tools' -> 'NuGet Package Manager' -> 'Package Manager Console'. This should open up a command line interface. On top of the interface there is a dropdown menu where you can select the Default Project. This is the project that Binance.Net will be installed in. After selecting the correct project type  `Install-Package Binance.Net`  in the command line interface. This should install the latest version of the package in your project.

After doing either of above steps you should now be ready to actually start using Binance.Net.
## Getting started
After installing it's time to actually use it. To get started we have to add the Binance.Net namespace:  `using Binance.Net;`.

Binance.Net provides two clients to interact with the Binance API. The  `BinanceClient`  provides all rest API calls. The  `BinanceSocketClient`  provides functions to interact with the websocket provided by the Binance API. Both clients are disposable and as such can be used in a  `using`statement.

Most API methods are available in two flavors, sync and async:
````C#
public void NonAsyncMethod()
{
    using(var client = new BinanceClient())
    {
        var result = client.Ping();
    }
}

public async Task AsyncMethod()
{
    using(var client = new BinanceClient())
    {
        var result2 = await client.PingAsync();
    }
}
````

## Examples
Examples can be found in the Examples folder.


## Response handling
All API requests will respond with an CallResult object. This object contains whether the call was successful, the data returned from the call and an error if the call wasn't successful. As such, one should always check the Success flag when processing a response.
For example:
```C#
using(var client = new BinanceClient())
{
	var result = client.GetServerTime();
	if (result.Success)
		Console.WriteLine($"Server time: {result.Data}");
	else
		Console.WriteLine($"Error: {result.Error.Message}");
}
```
## Options & Authentication
The default behavior of the clients can be changed by providing options to the constructor, or using the `SetDefaultOptions` before creating a new client. Api credentials can be provided in the options.

## Websockets
The Binance.Net socket client provides several socket endpoint to which can be subscribed.

**Public socket endpoints:**
```C#
using(var client = new BinanceSocketClient())
{
	var successDepth = client.SubscribeToDepthStream("bnbbtc", (data) =>
	{
		// handle data
	});
	var successTrades = client.SubscribeToTradesStream("bnbbtc", (data) =>
	{
		// handle data
	});
	var successKline = client.SubscribeToKlineStream("bnbbtc", KlineInterval.OneMinute, (data) =>
	{
		// handle data
	});
	var successSymbol = client.SubscribeToSymbolTicker("bnbbtc", (data) =>
	{
		// handle data
	});
	var successSymbols = client.SubscribeToAllSymbolTicker((data) =>
	{
		// handle data
	});
	var successOrderBook = client.SubscribeToPartialBookDepthStream("bnbbtc", 10, (data) =>
	{
		// handle data
	});
}
```

**Private socket endpoints:**

For the private endpoint a user stream has to be started on the Binance server. This can be done using the `StartUserStream()` method in the `BinanceClient`. This command will return a listen key which can then be provided to the private socket subscription:
```C#
using(var client = new BinanceSocketClient())
{
	var successOrderBook = client.SubscribeToUserStream(listenKey, 
	(accountInfoUpdate) =>
	{
		// handle account info update
	},
	(orderInfoUpdate) =>
	{
		// handle order info update
	});
}
```

**Handling socket events**

Subscribing to a socket stream returns a BinanceStreamSubscription object. This object can be used to be notified when a socket closes or an error occures:
````C#
var sub = client.SubscribeToAllSymbolTicker(data =>
{
	Console.WriteLine("Reveived list update");
});

sub.Data.Closed += () =>
{
	Console.WriteLine("Socket closed");
};

sub.Data.Error += (e) =>
{
	Console.WriteLine("Socket error " + e.Message);
};
````

**Unsubscribing from socket endpoints:**

Sockets streams can be unsubscribed by using the `client.UnsubscribeFromStream` method in combination with the stream subscription received from subscribing:
```C#
using(var client = new BinanceSocketClient())
{
	var successDepth = client.SubscribeToDepthStream("bnbbtc", (data) =>
	{
		// handle data
	});

	client.UnsubscribeFromStream(successDepth.Data);
}
```

Additionaly, all sockets can be closed with the `UnsubscribeAllStreams` method. Beware that when a client is disposed the sockets are automatically disposed. This means that if the code is no longer in the using statement the eventhandler won't fire anymore. To prevent this from happening make sure the code doesn't leave the using statement or don't use the socket client in a using statement:
```C#
// Doesn't leave the using block
using(var client = new BinanceSocketClient())
{
	var successDepth = client.SubscribeToDepthStream("bnbbtc", (data) =>
	{
		// handle data
	});

	Console.ReadLine();
}

// Without using block
var client = new BinanceSocketClient();
client.SubscribeToDepthStream("bnbbtc", (data) =>
{
	// handle data
});
```
