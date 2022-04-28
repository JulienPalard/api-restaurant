# The Restaurants API

This API only exposes two routes:

- `/`
- `/restaurants/`


## The slash route

This route is the « API homepage », like for a website it's the page
where users, given the API URL, will land, so it should be usefull to
them.

The bare minimum is to list usefull links here, like:

```json
{
    "restaurants": "/restaurants/"
}
```

If you want to add more information, why not returning a
[json-home](https://mnot.github.io/I-D/draft-nottingham-json-home.txt)?
I'd love it!


## The `/restaurants/` route

This route lists restaurants. All restaurant. From your nearby
pizzeria, to (The Restaurant at the End of the
Universe)[https://en.wikipedia.org/wiki/The_Restaurant_at_the_End_of_the_Universe].

The list is a bit long, though, so it'll have to be limited or
paginated in some way.

This route accepts a few query string parameters:

- `q` to query a restaurant by name, for example `?q=Chardonnet`.
- `lat`, `lon` to query all restaurants around the given point.

To query for a small area around a point, one have to specify a
precise `lat`/`lon` pair, like `?lat=45.4674066&lon=6.9026969`.

To query for bigger areas, one can give less precise coordinates
like `?lat=45.4674&lon=6.9026`, or an even bigger areas using
`?lat=45.4&lon=6.9`, and so on, more precisely:

- `lat=45.4` means `45.4 <= lat < 45.5`
- `lat=45.46` means `45.46 <= lat < 45.47`
- `lat=45.467` means `45.467 <= lat < 45.468`
- ...

In other words you can see `lat` and `lon` parameters as regexes.

So yes, it's possible to query for a rectangle by giving a different
precision for `lat` and `lon`. But no, it is not possible to describe
all possible rectangles using this lean syntax.


### The response

The response should obviously be given in UTF-8 encoded JSON.

It should be an object containing an `items` key, containing a list of
restaurants in the form of an OSM node, like:

```json
{
  "items": [
    {
      "changeset": 2159133,
      "id": 468716078,
      "lat": 45.4674066,
      "lon": 6.9026969,
      "tags": {
        "amenity": "restaurant",
        "name": "Le Chardonnet"
      },
      "timestamp": "2009-08-15T23:34:57Z",
      "type": "node",
      "uid": 87991,
      "user": "andygates",
      "version": 1
    },
    {...},
    {...},
    {...},
  ]
}
```

If you want to implement pagination you're free to add attributes as
needed to the root object.


## Where to get the data?

As you understood, you'll use the `OSM` database.

An easy way to query `OSM` is to use
[overpass](https://wiki.openstreetmap.org/wiki/Overpass).

To play around, you can use [overpass
turbo](https://overpass-turbo.eu/), as an example, to find all
restaurant in `Tignes` I use:

=> https://overpass-turbo.eu/s/156z

`OSM` is a free and open-source gem, but as you imagine the overpass
servers are not funded by an overwhelming amount of advertisment,
privacy infringings, customer data sellings, black hat SEO, and you
name it.

The `overpass` queries can be greedy in resources, so we'll take a lot
of care reducing the number of requests to the vital minimum: we're
good citizens.

As the overpass language allows to ask for a specific kind of
response, it's easy to get some `json` here:

```python
city = 'Tignes'
response = requests.post(
    "https://lz4.overpass-api.de/api/interpreter",
    f"""[out:json];area[name="{city}"];node["amenity"="restaurant"](area);out meta;""".encode("UTF-8")
)
```

But to allow you to start writing code quickly without even hitting
the overpass servers I've already executed the query for Tignes'
restaurants and stored it here:

https://mdk.fr/x/tignes.json

So you can use it during the development phase, again to keep the
number of actual overpass queries to the bare minimum, and if you're
one of the good citizen between the good citizens you'll even download
my file once and for all instead of querying it hundreds of times,
just saying.
