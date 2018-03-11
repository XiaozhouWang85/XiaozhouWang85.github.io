---
layout: post
title:  "Welcome to Jekyll!"
date:   2018-02-18 13:21:40 +0800
categories: jekyll update
---

### How to consume API data with Python

This tutorial is largely based on the following article: https://www.dataquest.io/blog/python-api-tutorial/

First import the library used for handling HTTP communications.


```python
import requests
```

Use a GET request to obtain information from this API. The API returns the current location of the International Space Station. The data is returned by the requests library as a response object that contains various information.


```python
resp = requests.get("https://p1l1hos9n0.execute-api.us-east-1.amazonaws.com/UAT")
```


```python
resp.content
```




    b'{"message":"Missing Authentication Token"}'



Accessing the status code of the response object shows it has returned 200. The following are a list of response codes and their meaning:

- 200 – everything went okay, and the result has been returned (if any)
- 301 – the server is redirecting you to a different endpoint. This can happen when a company switches domain names, or an endpoint name is changed.
- 401 – the server thinks you’re not authenticated. This happens when you don’t send the right credentials to access an API (we’ll talk about authentication in a later post).
- 400 – the server thinks you made a bad request. This can happen when you don’t send along the right data, among other things.
- 403 – the resource you’re trying to access is forbidden – you don’t have the right permissions to see it.
- 404 – the resource you tried to access wasn’t found on the server.

The data is returned as text can also be accessed from the object.


```python
resp.content
```




    b'{"timestamp": 1508379020, "message": "success", "iss_position": {"longitude": "-176.6575", "latitude": "50.8896"}}'



The second API requires a user input in order to return the data. Submitting a get request in the same manner as before leads to an error code being returned.


```python
resp2 = requests.get("http://api.open-notify.org/iss-pass.json")
```


```python
resp2.status_code
```




    400



The API requires latitude / longitdude to be provided to function


```python
#Hong Kong
Lat=22.3964
Lon=114.1095
```

A clumsy way of using this information


```python
resp2 = requests.get("http://api.open-notify.org/iss-pass.json?lat="+str(Lat)+"&lon="+str(Lon))
resp2.status_code
```




    200



A far more elegant way is provided by requests library to pass parameters to the API. Create a dictionary of all variables and then pass this to get call.


```python
Add='Flat E, Floor 3, Block 9, Site 2, whampoa garden'
key='AIzaSyCD7EGh8kFqy7Oh8GrmUn6HJAn1q70aQpY'
```


```python
parameters = {"address": Add, "key": key}
resp2 = requests.get("https://maps.googleapis.com/maps/api/geocode/json", params=parameters)
```


```python
print(resp2.text)
```

    {
       "results" : [
          {
             "address_components" : [
                {
                   "long_name" : "7",
                   "short_name" : "7",
                   "types" : [ "street_number" ]
                },
                {
                   "long_name" : "Tak On Street",
                   "short_name" : "Tak On St",
                   "types" : [ "route" ]
                },
                {
                   "long_name" : "Hung Hom",
                   "short_name" : "Hung Hom",
                   "types" : [ "neighborhood", "political" ]
                },
                {
                   "long_name" : "Kowloon",
                   "short_name" : "Kowloon",
                   "types" : [ "administrative_area_level_1", "political" ]
                },
                {
                   "long_name" : "Hong Kong",
                   "short_name" : "HK",
                   "types" : [ "country", "political" ]
                }
             ],
             "formatted_address" : "7 Tak On St, Hung Hom, Hong Kong",
             "geometry" : {
                "location" : {
                   "lat" : 22.3040823,
                   "lng" : 114.1888033
                },
                "location_type" : "ROOFTOP",
                "viewport" : {
                   "northeast" : {
                      "lat" : 22.3054312802915,
                      "lng" : 114.1901522802915
                   },
                   "southwest" : {
                      "lat" : 22.3027333197085,
                      "lng" : 114.1874543197085
                   }
                }
             },
             "place_id" : "ChIJg0JlBeAABDQReVxuolVRwLM",
             "types" : [ "establishment", "point_of_interest" ]
          }
       ],
       "status" : "OK"
    }
    
    


```python
import json
```


```python
resp2.json()
```




    {'message': 'success',
     'request': {'altitude': 100,
      'datetime': 1508388038,
      'latitude': 22.3964,
      'longitude': 114.1095,
      'passes': 5},
     'response': [{'duration': 228, 'risetime': 1508401611},
      {'duration': 626, 'risetime': 1508407239},
      {'duration': 488, 'risetime': 1508413077},
      {'duration': 412, 'risetime': 1508455347},
      {'duration': 632, 'risetime': 1508461001}]}



Store results into an object and parse it


```python
loc_j=resp2.json()
```


```python
loc_j.keys()
```




    dict_keys(['message', 'request', 'response'])




```python
count=0
for iss_pass in loc_j['response']:
    count+=1
    print('Pass '+str(count)+', Duration:'+str(iss_pass['duration'])+', Rise time: '+ str(iss_pass['risetime']))
```

    Pass 1, Duration:228, Rise time: 1508401611
    Pass 2, Duration:626, Rise time: 1508407239
    Pass 3, Duration:488, Rise time: 1508413077
    Pass 4, Duration:412, Rise time: 1508455347
    Pass 5, Duration:632, Rise time: 1508461001
    


```python

```


{% highlight python %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
