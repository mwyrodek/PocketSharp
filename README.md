# PocketSharp

PocketSharp is a C#.NET class library, that integrates the [Pocket API v3](http://getpocket.com/developer).

_If you don't know Pocket, be sure to check it out [Pocket](http://getpocket.com). It's an awesome service, which lets you save articles, videos, ... in the cloud and access it from all your devices._

PocketSharp consists of 4 parts:

- Authentication
- Retrieve
- Modify _(work in progress)_
- Add _(work in progress)_

---

## Usage Example

Request a Consumer Key on Pocket: [My Applications on Pocket](http://getpocket.com/developer/apps/)

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
_client.Search("css").ForEach(
	item => Console.WriteLine(item.ID + " | " + item.Title)
);
```

Which will output:

    330361896 | CSS Front-end Frameworks with comparison : By usabli.ca
    345541438 | Editr - HTML, CSS, JavaScript playground
    251743431 | CSS Architecture
    343693149 | CSS3 Transitions - Thank God We Have A Specification!
	...

## Create an instance

There are 4 overloads for `PocketClient`:

```csharp
// used for authentication, when no accessCode is available yet
PocketClient(string consumerKey)

// start the PocketClient with an accessCode
PocketClient(string consumerKey, string accessCode)

// different base URL
PocketClient(string consumerKey, Uri baseUrl)

// accessCode and different base URL
PocketClient(string consumerKey, string accessCode, Uri baseUrl)
```

You can change the _Access Code_ after initialization:

```csharp
_client.AccessCode = "[YOU_ACCESS_CODE]";
```

## Authentication

In order to communicate with a Pocket User, you will need a consumer key (which is generated by [creating a new application on Pocket](http://getpocket.com/developer/apps/)) and an **Access Code**.

This is a 3-step process:

1) Receive the authentication URI from the library by calling `Uri Authenticate(Uri callbackUri)`:

```csharp
Uri authenticationUri = client.Authenticate(new Uri("http://example.com"));
```

The `callbackUri` is the URI which is triggered after the user successfully (or not) authenticated your application. 

In this step not only the `authenticationUri` is generated, but the PocketSharp client also stores a received _Request Code_ internally.

2) Next you need to redirect the user to the `authenticationUri`, which displays a prompt to grant permissions for the application. After the user granted or denied, he/she is redirected to the `callbackUri`.

3) Call `string GetAccessCode()`

```csharp
string accessCode = GetAccessCode();
```

Again, the received _Access Code_ is stored internally.
Note that `GetAccessCode` can only be called with an existing _Request Code_. If you need to re-authenticate a user, start again with Step 1).

#### Note

If you need to re-instantiate PocketClient after the redirect, please provide the _Request Code_:

```csharp
_client.RequestCode = "[YOU_REQUEST_CODE]";
```

After Step 1) you have public access to the RequestCode property and can store it for later usage.

```csharp
string requestCode = _client.RequestCode;
```

#### Important

**Be sure to permanently store the _Access Code_ for your user.**
<br>
Without it you would always have to redo the authentication process.

## Retrieve

Find items by a tag:

```csharp
List<PocketItem> items = _client.SearchByTag("tutorial");
```

Find items by a search string:

```csharp
List<PocketItem> items = _client.Search("css");
```

Find items by a filter:

```csharp
List<PocketItem> items = _client.Retrieve(RetrieveFilter.Favorite); // only favorites
```

The RetrieveFilter Enum is specified as follows:

```csharp
enum RetrieveFilter { All, Unread, Archive, Favorite, Article, Video, Image }
```

#### Custom Parameters

You can create a completely custom parameter list for retrieval with the POCO `RetrieveParameters`:

```csharp
var parameters = new RetrieveParameters()
{
	Count = 50,
	Offset = 100,
	Sort = SortEnum.oldest
	...
};

List<PocketItem> items = _client.Retrieve(parameters);
```

## Add

Adds a new item to your pocket list.
Accepts four parameters, with `uri` being required.

```csharp
PocketItem Add(Uri uri, string[] tags = null, string title = null, string tweetID = null)
```

Example:

```csharp
PocketItem newItem = _client.Add(
	new Uri("http://www.neowin.net/news/full-build-2013-conference-sessions-listing-revealed"),
	new string[] { "microsoft", "neowin", "build" }
);
```

The title can be included for cases where an item does not have a title, which is typical for image or PDF URLs. If Pocket detects a title from the content of the page, this parameter will be ignored.

If you are adding Pocket support to a Twitter client, please send along a reference to the tweet status id (with the tweetID). This allows Pocket to show the original tweet alongside the article.

## Modify

All Modify methods accept either the itemID (as int) or a `PocketItem` as parameter.

Archive the specified item:

	bool isSuccess = _client.Archive(myPocketItem);

Un-archive the specified item:

	bool isSuccess = _client.Unarchive(myPocketItem);

Favorites the specified item:

	bool isSuccess = _client.Favorite(myPocketItem);

Un-favorites the specified item:

	bool isSuccess = _client.Unfavorite(myPocketItem);

Deletes the specified item:

	bool isSuccess = _client.Delete(myPocketItem);

Add tags to the specified item:

	bool isSuccess = _client.AddTags(myPocketItem, new string[] { "css", "2013" });

Remove tags from the specified item:

	bool isSuccess = _client.RemoveTags(myPocketItem, new string[] { "css", "2013" });

Remove all tags from the specified item:

	bool isSuccess = _client.RemoveTags(myPocketItem);

Replaces all existing tags with new ones for the specified item:

	bool isSuccess = _client.ReplaceTags(myPocketItem, new string[] { "css", "2013" });

Renames a tag for the specified item:

	bool isSuccess = _client.RenameTag(myPocketItem, "oldTagName", "newTagName");

---

## Release History

- 2013-06-26 v0.1.0 authentication & retrieve functionality

## Contributors
| [![twitter/artistandsocial](http://gravatar.com/avatar/9c61b1f4307425f12f05d3adb930ba66?s=70)](http://twitter.com/artistandsocial "Follow @artistandsocial on Twitter") |
|---|
| [Tobias Klika @ceee](https://github.com/ceee) |