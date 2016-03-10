---
layout: post
title: "Laravel Eloquent Calculated Fields"
description: "How to select additional fields on the fly with eloquent scopes."
date: 2016-03-10 01:00:00
categories: php
---
In certain situations it's useful to calculate extra fields from within your eloquent models. For instance it may be useful to calculate distance for a model that has coordinate fields.

This can be achieved by using [eloquents scope functions](https://laravel.com/docs/5.1/eloquent#query-scopes) to add the additional field when required.  

First create a model with its base fields.
{% highlight php %}
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Venue extends Model
{
    protected $fillable = [
        'name',
        'lat',
        'lng',
    ];
}
{% endhighlight %}

Then to create the distance field, we can add a scope function that uses the [Haversine formula](http://stackoverflow.com/questions/574691/mysql-great-circle-distance-haversine-formula#answer-574736)
to calculate the distance in miles between two sets of latitude and longitude coordinates.

{% highlight php %}
<?php
//app/Venue.php

public function scopeWithDistance($query, $lat, $lng)
{
    $raw = '(floor(3959 * acos(cos(radians(:lat1)) * cos(radians(lat))
                    * cos(radians(lng) - radians(:lng))
                    + sin(radians(:lng2)) * sin(radians(lat)))
                )) AS distance';

    return $query->selectRaw($raw, [
            'lat1' => $lat,
            'lng' => $lng,
            'lat2' => $lat,
    ]);
}
{% endhighlight %}
But now when the model is called with the distance field the selectRaw query has overwritten the models original select query. As a result the model will now only contain the distance field. 
{% highlight php %}
>>> dd(App\Venue::withDistance(54.23445, -03.23456)->first()->toArray());
array:1 [
    "distance" => "6294"
]
{% endhighlight %}
We can correct this by overwriting the eloquent models [newQuery function](https://laravel.com/api/5.2/Illuminate/Database/Query/Builder.html#method_newQuery)
 which is called at the start of any new query involving the model. Here we can re-add the "select *" query.
{% highlight php %}
<?php
//app/Venue.php

public function newQuery()
{
    return parent::newQuery()->select('venues.*');
}
{% endhighlight %}
Now when the model is called all base and calculated fields are returned. 
{% highlight php %}
>>> dd(App\Venue::withDistance(54.23445, -03.23456)->first()->toArray());
array:7 [
    "id" => 1
    "name" => "The Dublin Castle Camden"
    "lat" => "-13.953238"
    "lng" => "-75.096157"
    "created_at" => "2016-03-10 03:03:11"
    "updated_at" => "2016-03-10 03:03:11"
    "distance" => "6294"
]
{% endhighlight %}
