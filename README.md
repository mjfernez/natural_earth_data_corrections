# Correct Natural Earth Shape Files

There are some tiny errors in the data provided by [Natural Earth
Data](https://www.naturalearthdata.com/), which impacted a project I was
working on. I was attempting to import [the
world](https://www.naturalearthdata.com/downloads/10m-cultural-vectors/)
in Elastic, but Elastic has some bug where you can't upload GeoJSON
through the web form, so I had to do it manually, like this:
```bash
NAME,ECONOMY,FORMAL_EN,GDP_MD,ISO_A2
ogr2ogr -f ElasticSearch  -progress \
-select $fields \
-lco NOT_ANALYZED_FIELDS=$fields \
-lco INDEX_NAME=countries \
-lco OVERWRITE_INDEX=YES \
ES:http://localhost:9200 \
/vsizip/./ne_10m_admin_0_countries.zip/ne_10m_admin_0_countries.shp
```

But ogr2ogr yells at you after processing about 170 countries or so. If
you run the same with the `-skipfailures` option, you'll see every
country gets indexed *except* Egypt! Why?

A look a the json output from ogr2ogr (which I will spare you here),
ultimately lead me to:

```code
"Self-intersection at or near point [35.621087106,23.139292914]"
```

So, I opened it in QGIS and well...

![img](./oops.png)

Funny enough the lines in the middle aren't a problem, just this
one point sitting on the border.

Fortunately, QGIS has a Geometry Checker Plugin, but unfortunately, it's
a bit complicated and was a pain to do. If you don't tune it right, you
end up having to sort through lots of "mistakes" which aren't mistakes.

Also there's an [unfixed bug](https://github.com/qgis/QGIS/issues/37527) 
in QGIS which doesn't make shape files correctly.
Don't know how that's possible considering that's
literally what the software's made for, but I could only get geojson
input to work correctly.

For anyone else who might be down this rabbit hole, Egypt is Object ID
161--I promise that will save you time. Or you could just download my
copy of the file here.

Hoping to use this git repo as part of a bug report, once I read their
process on that.
