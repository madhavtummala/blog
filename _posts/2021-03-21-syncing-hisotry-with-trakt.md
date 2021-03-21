---
title: "Syncing history with Trakt"
date: 2021-03-21
layout: post
tag: guide
---

This guide will explain scrapping local movie/show collection and syncing with Trakt using Trakt API.
Follow the [API docs](https://trakt.docs.apiary.io/) and [my project](https://github.com/madhavtummala/trakt_sync) 
to see how you can achieve this. My code has been tested to work on macOS (both M1 and Intel). Contributions are welcome.

## Some Background
I am a big fan of movies and shows and try to get my hands on anything
that is popular and entertaining.
Thanks to my new internet connection and a new 1Tb external hard disk,
In 2012, I started maintaining a digital collection of all movies I watched
on TV, in Theaters or just from somewhere on the Internet.
In 2016, this collection grew to 500gb, and I had accidentally deleted all of
it in an OS Installation experiment. Instead of stopping there,
I went ahead and downloaded all of it again 😬

I now hold close to 1.5Tb of movies and shows in an external hard disk of 2Tb.
All with properly formatted folder names and files and yes, if you couldn't
tell already, I take pride in useless things like proper formatting. In the age
of streaming, personal collections are becoming obsolete. I can't possibly keep up with the
catalog offered by Netflix or Youtube Movies. But I have an emotional connection
with the collection I have accumulated over years. After all, this was my childhood,
looking up a movie, downloading it and watching it with my family. And again, what could I
possibly do with my monthly 700gb internet.

## Survey
I wanted to document everything I watched, for myself. I started looking at websites that
allow this like IMDb, Letterboxd, Trakt. As a person from IT background, automation was the first
concern for me. Anything I do must be infinitely repeatable with minimal effort.
If the service on the other end has an API, makes it really easy.
* [IMDb](https://www.imdb.com) is quite famous, and upon creating an account, you can add movies or shows to your
  watchlist, watched history and also rate them on a scale of 10. The reviews and parents guide
  are really useful, and it is easy to discover new content. There is an [API](https://developer.imdb.com), but is quite
  restricted in sense that you need to submit an application for building a legit App in order to access it.
* [Letterboxd](https://letterboxd.com) perfectly has what I need. It's marketed as a social
  media site for film lovers. Films(not shows) can be tracked, added to watchlist, recommended to friends.
  There is an open [API](http://api-docs.letterboxd.com), so this website is great.
* [Trakt](https://trakt.tv/) is a website with the primary goal of tracking, both movies and shows. It
  has an open [API](https://trakt.docs.apiary.io/) and more ads compared to Letterboxd. It has some more interesting
  features such as check-in, scrobble but the website feels a little unpolished with more Ads.
* [TMDb](https://www.themoviedb.org/) is not a social media site, but is used as a definitive source
  for identifying movies and shows by many other websites and apps. It has a very famous [API](https://www.themoviedb.org/documentation/api).
  You can create lists and populate your watch history with it. It will not disappear anytime soon and will only get famous as more and more
  services start using the API.

IMDb was already out of scope, since it doesn't have an API I could use. Letterboxd is great
overall for cinema lovers and website looks great with unobtrusive minimal ads, but the entire
thing is **only** for movies. TMDb is a great choice for an API. If I were to start my own
application that tracks movies and shows, I would use TMDb. That said, TMDb is blocked in India,
and I cannot visit the website without a VPN/proxy which is a huge blocker. 

So, I chose Trakt at the end.

## Trakt API
We will discuss how to
- Use Trakt API in Python
- Scrap file system data in Python
- Sync watch history between local and trakt account
- Other trakt features

All the code used will be in [my project](https://github.com/madhavtummala/trakt_sync) and follow the README
for the initial setup. You need to create a new application [here](https://trakt.tv/oauth/applications). Fill some random url
like `https://localhost:0000/` in `Redirect uri`, we won't be using that. Copy the client secret and client id that get
created and add them to **secrets.py** as required.

### Authentication
I didn't spend enough time on this part, so there might be better solutions out there to do the same. 
The API docs for authentication part was confusing, so I resorted to using a python library [trakt](https://pypi.org/project/trakt/)
for doing this. I am using [device authentication](https://trakt.docs.apiary.io/#reference/authentication-devices)
as mentioned in docs.
 
All you have to do is run **auth.py** and follow the instructions, you will get an authorization json object like this:
```json5
{
  'access_token': '<your_access_token>',
  'token_type': 'Bearer',
  'expires_in': 7776000,
  'refresh_token': '<your_refresh_token>',
  'scope': 'public',
  'created_at': '<your_creation_time>'
}
```
Store the entire thing as an `authorization` object and access_token separately in **secrets.py**. This access_token is 
valid for three months, so this is pretty much a one time thing. In case you need to refresh the token, please follow 
the code in `auth.py` to only call that function, I am working on improvising this code.

### Using the API
I used the standard `requests` library and followed the [API docs](https://trakt.docs.apiary.io/) to achieve just the watched history sync.
Let's look at [get-history](https://trakt.docs.apiary.io/#reference/sync/get-history/get-watched-history)
and [add-to-history](https://trakt.docs.apiary.io/#reference/sync/add-to-history/add-items-to-watched-history) methods in specific.

The function to create the header is:
```python
def prepare_headers(access_token, my_client_id):
	headers = {}
	headers['Content-Type'] = 'application/json'
	headers['Authorization'] = access_token
	headers['trakt-api-version'] = '2'
	headers['trakt-api-key'] = my_client_id
	return headers
```
Then we can get the current watched history from trakt by:
```python
def list_in_cloud(media):
	get_url = "https://api.trakt.tv/sync/history/{}/?limit=1000".format(media+"s")
	headers = prepare_headers(access_token, my_client_id)
	response = requests.get(get_url, headers=headers)
	items = response.json()
	return items
```
Here `media` is either *movie* or *show*. The response is paginated, so I put `limit` at 1000, to get all my results in a
single page. The format will be same as shown as the API docs page.

Each watched history object is a `media item`. It will have associated with it:
- "type" : movie / show / episode / season
- "title" : just a title
- "year" : exact year for movie, first episode release year for show
- "watched_at" : last_watched and last_updated information
- "plays" : # of plays for this movie/show/episode/season

<u>example:</u>
```json
{
  "plays": 4,
  "last_watched_at": "2014-10-11T17:00:54.000Z",
  "last_updated_at": "2014-10-11T17:00:54.000Z",
  "movie": {
    "title": "Batman Begins",
    "year": 2005,
    "ids": {
      "trakt": 6,
      "slug": "batman-begins-2005",
      "imdb": "tt0372784",
      "tmdb": 272
    }
  }
}
```
Now assuming you have all your movies in local that you wish to sync with trakt in this format:
```python
list['title'(string), 'year'(int), 'watched_at'(utc_time string), 'trakt_id'(int)]
```
We can use the following code to add them to watch history.
```python
def prepare_big_json(extra_local, media):
	my_json = {}
	my_json[media+"s"] = []
	for item in extra_local:
		my_obj = {}
		my_obj['watched_at'] = item[2]
		my_obj['title'] = item[0]
		if item[1] != None:
			my_obj['year'] = item[1]
		my_obj['ids'] = {}
		my_obj['ids']['trakt'] = item[3]
		my_json[media+"s"].append(my_obj)

	json_data = json.dumps(my_json)
	return json_data
	
def bulk_add_to_cloud(extra_in_local, media):
	api_url = "https://api.trakt.tv/sync/history"
	headers = prepare_headers(access_token, my_client_id)
	json_data = prepare_big_json(extra_in_local, media)
	print("Uploading Bulk {} items".format(len(extra_in_local)))
	response = requests.post(api_url, data=json_data, headers=headers)
	while(response.status_code != 201):
		time.sleep(0.5)
		response = requests.post(api_url, data=json_data, headers=headers)
```

It's important to note that, if an item is already present in trakt account as watched, adding a new instance
of the same time will add a new play, so the # of plays will be incremented by 1 and last_watched timestamp will be updated.
If you wish only to upload items that are not in cloud, then we could perform and additional check based on trakt_ids 
and post only those that are extra in local
```python
def find_extra_in_local(original_cloud, original_local):
	cloud = [i[i['type']]['ids']['trakt'] for i in original_cloud]
	cloud_ids = list(set(cloud))
	extra_local = [i for i in original_local if i[3] not in cloud_ids]
	return extra_local
```

### Scrapping local collection
This segment is very specific to one's own organisation. If one doesn't have a consistent naming and placement, it 
will be difficult to write a script. In my case, this is the file structure for movies has division based on language,
year before reaching the actual movie folder and for shows has division based on language as shown:
```
<movies_root>
|---english
    |---2009
    |---2021
        |---"<movie_title1> (<year>)"
            |---<some_random_long_name>.mp4
            |---<some_random_long_name>.srt
            |---<additional_files_or_folders>
        |---"<movie_title2> (<year>)"
|---spanish
|---hindi

<shows_root>
|---english
    |---"<show_name>"
        |---"S0<season_number>"
            |---<some_random_long_episode_name>.mp4
            |---<another_episode>
            |---<additional_files_and_subtitles>
|---spanish
```
I only use the information from the **movie folder** - *title, year and creation time* and from the **show folder** - *title and 
creation time*. I am using creation time as watched_time, but one can use their own logic and if passed empty, 
trakt will automatically use the release date of movie/show as your watched_time.
```python
def list_in_local(media):
	if media == "movie":
		path = os.walk("/Volumes/Backup Plus/Movies")
		local_list = []
		for root, directories, files in path:
			if "(" in root and ")" in root.split("/")[-1]:
				try:
					name = root.split("/")[-1]
					title, year = name.split("(")
					year = year.split(")")[0]
					timetup = time.gmtime(os.stat(root).st_birthtime)
					watched_at = datetime(*timetup[:6]).isoformat(timespec='milliseconds') + 'Z'
					local_list.append([title.strip(), int(year.strip()), watched_at])
				except:
				    print(name)
					continue
		return local_list
	else:
		path = os.walk("/Volumes/Backup Plus/TV SERIES")
		local_list = []
		for root, directories, files in path:
			if len(root.split("/")) == 6:
				title = root.split("/")[-1]
				timetup = time.gmtime(os.stat(root).st_birthtime)
				watched_at = datetime(*timetup[:6]).isoformat(timespec='milliseconds') + 'Z'
				# print(title, watched_at)
				local_list.append([title.strip(), None, watched_at])
		return local_list
```

We will now add an **id** information to each item we collected, in addition to the title, year and watched_at information we already 
have. This is necessary to get a definitive item to be able to compare and store. For this, we will use the search endpoint,
using title and year with the following code and then use it for steps mentioned in the previous section.
```python
def add_trakt_id_to_local_list(local_list, media):
	search_url = "https://api.trakt.tv/search/{}/".format(media)
	headers = prepare_headers(access_token, my_client_id)
	local_list_with_id = []
	for item in local_list:
		params = {'query' : item[0]}
		print("Searching {}({})".format(item[0], item[1]))
		response = requests.get(search_url, headers=headers, params = params)
		while response.status_code != 200:
			time.sleep(0.5)
			response = requests.get(search_url, headers=headers, params = params)
		found = False
		# print("Got {} results".format(len(response.json())))
		for i in response.json():
			if i[i['type']]['year'] == item[1]:
				item.append(i[i['type']]['ids']['trakt'])
				local_list_with_id.append(item)
				found = True
				break
		if not found and len(response.json()) > 0:
			i = response.json()[0]
			print("Couldn't find exact.. selecting {}({}) score:{}"
				.format(i[i['type']]['title'], i[i['type']]['year'], i['score']))
			item.append(i[i['type']]['ids']['trakt'])
			local_list_with_id.append(item)
	return local_list_with_id
```

### Other trakt features
Looking at the API docs, already gives us an idea of features supported by trakt.
- Allows you to track shows, episodes yet to be released next week and episodes available to be watched are show in dashboard
- Categorisation of watch history by genres
- One can organise and track content from multiple services like Netflix, Disney+ etc
- Scrobble automatically marks you as watching if you watch through connected apps/services and check-in allows you to 
manually be marked as watching for that item.
- Import a new movie / show from TMDb to Trakt
- Standard collections, lists, comments and reviews
- Widgets allow embedding details of current activities or last watched in other websites (VIP)
- All [other](https://trakt.tv/vip) VIP features

<u>Widget Example(I don't have VIP 😕):</u>
<html>
<a target="_blank" href="https://trakt.tv/users/madhavtummala">
  <img
      alt="MadhavTummala" 
      src="https://widgets.trakt.tv/users/368c3c21a30a412a3e37d1750feaec85/watched/fanart2@2x.jpg" />
</a>
</html>

&nbsp;

## Conclusion
This syncing script was a one-time use for me, just to build up my watch history. Its understandable that not every
item is present in the local collection, but this was just a little experiment I did in the weekends. 
I have been using trakt ever since directly from the website using **check-in** for everything I watch.

Feel free to follow [me](https://trakt.tv/users/madhavtummala) there 
or open a [PR](https://github.com/madhavtummala/trakt_sync) for making scripts better.
