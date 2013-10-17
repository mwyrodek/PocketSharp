![PocketSharp](https://raw.github.com/ceee/PocketSharp/master/PocketSharp.Website/Assets/Images/github-header.png)

**PocketSharp** is a C#.NET class library, that integrates the [Pocket API v3](http://getpocket.com/developer).

**Website:** [pocketsharp.frontendplay.com](http://pocketsharp.frontendplay.com/)
<br>
**NuGet:** [nuget.org/packages/PocketSharp](https://www.nuget.org/packages/PocketSharp/)


## Install using NuGet

```
Install-Package PocketSharp
```

## Components

- [Get](#get) - Retrieve and search for items
- [Modify](#modify) - Modify items and its tags
- [Add](#add) - Add new items
- [Authentication](#account-authentication) - Authenticate users
- [Registration](#account-registration) - Register new users
- [Statistics](#statistics) - Retrieve account statistics

## Supported platforms

PocketSharp is a **Portable Class Library** (since 1.0.0), therefore it's compatible with multiple platforms:

- **.NET** >= 4.0.3 (including WPF)
- **Silverlight** >= 4
- **Windows Phone** >= 7.5
- **Windows Store**

You can find examples for Silverlight 5, WP8 and WPF in the `PocketSharp.Examples` ([@github](https://github.com/ceee/PocketSharp/tree/master/PocketSharp.Examples)) folder.

## Async Support

All public methods, which communicate with the Pocket servers are asynchronous. So they should be called with the preceding `await` keyword, and the encapsulated method marked with `async`.

_In PocketSharp versions < 1.0.0 all methods lacked async support and were blocking._

## What's next

Next is Reading API support (with NReadability implementation). Release: Mid October.

## Usage Example

Request a [Consumer Key on Pocket.](http://getpocket.com/developer/apps/)

Include the PocketSharp namespace and it's associated models (you will need them later):

```csharp
using PocketSharp;
using PocketSharp.Models;
```

Initialize PocketClient with:

```csharp
PocketClient _client = new PocketClient("[YOUR_CONSUMER_KEY]", "[YOUR_ACCESS_CODE]");
```

Do a simple request - e.g. a search for `CSS`:

```csharp
var items = await _client.Search("css");
items.ForEach(
	item => Debug.WriteLine(item.ID + " | " + item.Title)
);
```

Which will output:

    330361896 | CSS Front-end Frameworks with comparison : By usabli.ca
    345541438 | Editr - HTML, CSS, JavaScript playground
    251743431 | CSS Architecture
    343693149 | CSS3 Transitions - Thank God We Have A Specification!
	...

## Create an instance

Constructor:

```csharp
PocketClient(string consumerKey, string accessCode = null, Uri callbackUri = null)
```

`consumerKey`: The API key
<br>
`accessCode`: Provide an access code if the user is already authenticated
<br>
`callbackUri`: The callback URL is called by Pocket after authentication

Example:

```csharp
PocketClient _client = new PocketClient(
	consumerKey: "123498237423498723498723",
	callbackUri: new Uri("http://ceecore.com"),
	accessCode: "097809-oi987-izi8-jk98-oiuu89"
);
```

You can change the _Access Code_ after initialization:

```csharp
_client.AccessCode = "[YOU_ACCESS_CODE]";
```

**Before authentication** you will need to provide the `callbackUri` for authentication requests.
<br>
**After authentication** you will need to provide the `accessCode`.

## Account Authentication

In order to communicate with a Pocket User, you will need a consumer key (which is generated by [creating a new application on Pocket](http://getpocket.com/developer/apps/)) and an **Access Code**.

The authentication is a 3-step process:

#### 1) Generate authentication URI

Receive the **request code** and **authentication URI** from the library by calling `string GetRequestCode()`:

```csharp
string requestCode = await _client.GetRequestCode();
// 0f453f2d-1605-8584-28fd-39af8e
Uri authenticationUri = _client.GenerateAuthenticationUri();
// https://getpocket.com/auth/authorize?request_token=0f453f2d-1605-8584-28fd-39af8e&redirect_uri=http%253a%252f%252fceecore.com
```

The _request code_ is stored internally, but you can also provide it as param in `GenerateAuthenticationUri(string requestCode = null)`.

#### 2) Redirect to Pocket
Next you need to redirect the user to the `authenticationUri`, which displays a prompt to grant permissions for the application (see image). After the user granted or denied, he/she is redirected to the `callbackUri`.

![authentication screen](https://raw.github.com/ceee/PocketSharp/master/PocketSharp/authentication-screen.png)

#### 3) Get Access Code
Call `Task<string> GetAccessCode(string requestCode = null)`

```csharp
string accessCode = await _client.GetAccessCode();
// fa8bfc16-69b3-4d22-7db7-84a58d
```

Again, the received _access code_ is stored internally.
Note that `GetAccessCode` can only be called with an existing _request code_. If you need to re-authenticate a user, start again with Step 1).

#### Important

**Be sure to permanently store the _Access Code_ for your user.**
<br>
Without it you would always have to redo the authentication process.

## Account Registration

The `RegisterAccount` method exists in Pocket API v3, but is currently undocumented.
<br>The user cannot authenticate directly after registration, as the account has to be activated via an opt-in link, which is sent to the e-mail address.

```csharp
bool isSuccess = await _client.RegisterAccount("myUsername", "me@mymail.com", "mypassword");
```

After registration you have to remind the user to check his/her mail account.

## Get

Get list of all items:

```csharp
List<PocketItem> items = await _client.Get();
// equivalent to: await _client.Get(RetrieveFilter.All)
```

Get a list with specific parameters (explanation in the [Pocket Docs](http://getpocket.com/developer/docs/v3/retrieve)):

```csharp
List<PocketItem> items = await _client.Get(
  State? state = null,
  bool? favorite = null,
  string tag = null,
  ContentType? contentType = null,
  Sort? sort = null,
  string search = null,
  string domain = null,
  DateTime? since = null,
  int? count = null,
  int? offset = null 
);
```

It's best to use parameters as _named parameters_, to avoid typing `null` values:

```csharp
List<PocketItem> items = await _client.Get(count: 10, offset: 20, sort: Sort.oldest);
```

Get item by ID:

```csharp
PocketItem item = await _client.Get(1298198);
```

Find items by a tag:

```csharp
List<PocketItem> items = await _client.SearchByTag("tutorial");
```

Find items by a search string.
<br>PocketSharp uses an internal search, which is significantly faster than the Search API by Pocket.

```csharp
List<PocketItem> items = await _client.Search("css");
```

Find items by a search string by already available items:

```csharp
List<PocketItem> items = await _client.Search(myPocketItemList, "css");
```

Get all tags:

```csharp
List<PocketTag> items = await _client.GetTags();
```

Get a filtered list:

```csharp
List<PocketItem> items = await _client.Get(RetrieveFilter.Favorite);
// returns favorites only
```

The RetrieveFilter Enum is specified as follows:

```csharp
enum RetrieveFilter { All, Unread, Archive, Favorite, Article, Video, Image }
```

## Add

Adds a new item to your pocket list.
Accepts four parameters, with `uri` being required.

```csharp
Task<PocketItem> Add(Uri uri, string[] tags = null, string title = null, string tweetID = null)
```

Example:

```csharp
PocketItem newItem = await _client.Add(
	new Uri("http://www.neowin.net/news/full-build-2013-conference-sessions-listing-revealed"),
	new string[] { "microsoft", "neowin", "build" }
);
```

The title can be included for cases where an item does not have a title, which is typical for image or PDF URLs. If Pocket detects a title from the content of the page, this parameter will be ignored.

If you are adding Pocket support to a Twitter client, please send along a reference to the tweet status id (with the tweetID). This allows Pocket to show the original tweet alongside the article.

## Modify

All Modify methods accept either the itemID (as int) or a `PocketItem` as parameter.

Archive the specified item:

```csharp
bool isSuccess = await _client.Archive(myPocketItem);
```

Un-archive the specified item:

```csharp
bool isSuccess = await _client.Unarchive(myPocketItem);
```

Favorites the specified item:

```csharp
bool isSuccess = await _client.Favorite(myPocketItem);
```

Un-favorites the specified item:

```csharp
bool isSuccess = await _client.Unfavorite(myPocketItem);
```

Deletes the specified item:

```csharp
bool isSuccess = await _client.Delete(myPocketItem);
```

#### Modify tags

Add tags to the specified item:

```csharp
bool isSuccess = await _client.AddTags(myPocketItem, new string[] { "css", "2013" });
```

Remove tags from the specified item:

```csharp
bool isSuccess = await _client.RemoveTags(myPocketItem, new string[] { "css", "2013" });
```

Remove all tags from the specified item:

```csharp
bool isSuccess = await _client.RemoveTags(myPocketItem);
```

Replaces all existing tags with new ones for the specified item:

```csharp
bool isSuccess = await _client.ReplaceTags(myPocketItem, new string[] { "css", "2013" });
```

Renames a tag for the specified item:

```csharp
bool isSuccess = await _client.RenameTag(myPocketItem, "oldTagName", "newTagName");
```

## Statistics

The Pocket API supports retrieval of a simple statistics object.

```csharp
PocketStatistics statistics = await client.Statistics();
// PocketStatistics: [CountAll], [CountRead], [CountUnread]
```

---

## Release History

- **1.5.1** (2013-09-30) `RetrieveFilter.All` didn't work; improve search speed
- **1.5.0** (2013-09-28) add statistics and registration API
- **1.4.0** (2013-09-21) rename `Retrieve` to `Get` + update IntelliSense documentation + add `GetTags` method
- **1.3.0** (2013-09-19) get Item by ID + tag modification bugfixes
- **1.2.1** (2013-09-18) correct parameter conversion for DateTime and Boolean
- **1.2.0** (2013-09-17) simplified retrieve methods
- **1.1.0** (2013-09-17) fix modification requests
- **1.0.0** (2013-09-15) convert to PCL & implement async
- **0.3.2** (2013-08-16) tag modification fixed and full retrieval of items for Retrieve method
- **0.3.1** (2013-07-07) authentication fixes
- **0.3.0** (2013-07-02) update authentication process 
- **0.2.0** (2013-06-27) add, modify item & modify tags
- **0.1.0** (2013-06-26) authentication & retrieve functionality

## Dependencies

- [Microsoft.Bcl.Async](https://www.nuget.org/packages/Microsoft.Bcl.Async/)
- [Microsoft.Net.Http](https://www.nuget.org/packages/Microsoft.Net.Http/)
- [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/)

## Contributors
| [![twitter/artistandsocial](http://gravatar.com/avatar/9c61b1f4307425f12f05d3adb930ba66?s=70)](http://twitter.com/artistandsocial "Follow @artistandsocial on Twitter") |
|---|
| [Tobias Klika @ceee](https://github.com/ceee) |

## License

[MIT License](https://github.com/ceee/PocketSharp/blob/master/LICENSE-MIT)
