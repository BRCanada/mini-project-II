# Miniproject 2

### [Assignment](assignment.md)

## Project/Goals
The goal of this project is to use location APIs like Foursquare, Yelp and Google Places to return 
the closest locations (within 1km) to a specific location pins latitude/longtitude position. I will be creating a program
that uses this process to return and store a list of the closest music venues with their rating (out of 5) by locals. I am 
building this project under the assumption that my clients would be indie musicians looking for a fast and easy way to return a list
of venues that are specifically branded as "Music Venue", so the musicians would have a better idea of venue prospects
in the area of choice. This will help musicians by equipping them with a powerful tool which they can use to target venues in 
areas they know has a stronger fanbase.

## Process
### Step 1: API/Parameter Initialization
* First I imported my Libraries. The libraries I used were: **Pandas, sqlite3, os, requests** and **json**. I also imported the *json_normalize* method from Pandas so I could have more data cleaning possibilities. 
* I used Foursquare and Yelp APIs, and began by establishing accounts on those sites, and saving the api keys to my operating systems environment variables. 
* I also set up my 'Static Variables' at this stage, assigning the radius(1000m) and the latitude/longtitude from a Google Maps pin on Young-Dundas Square to their respective variable names. 

### Step 2: Function
* I defined a function called get_venues, which will take the parameter **urlstr**, a concatenation of my core api urls and the required queries for my project goals.
* Used the requests module to get the data from the api using **urlstr**
* Applied the *raise_for_status()* function from the **requests** library, which raises a pre-written exception should the code returned be 400 or 404.
* Used an if statement that checks remaining status codes. If the status code is not 204, the statement returns the response from the api call.

``` 
def get_venues(urlstr, head):
    """
    Requests information from the parameter url
    Responds with status code
    if the code is not 204, return the response.
    """
    response = requests.get(urlstr, headers=head)
    response.raise_for_status()
    if response.status_code != 204:
        return response
```

### Step 3: URLS
* Both APIs followed roughly the same query structure. I was able to delineate the lat/long parameters within the url, and the radius from that point was measured in meters for both of the APIs. 
* Foursquare API listed their venue types as categories, so I input 10039(Music Venues) into the query using *&categories=10039*
* Yelp used string tags for their categories, so this query was slightly different, input as *&categories=musicvenues*

### Step 4: Data Clean
* Both requests had their data converted to a dataframe, Where I used json_normalize() to organize the data by the 'businesses' column.
* Some column names like *location.display_address* were cleaned up using *.rename()*
* Foursquare uses a 10 point rating system, yelp uses a 5 star rating system. Foursquare's rating values were multiplied by 0.5 to align the ratings.
* Both results tables were sorted by rating and the index was reset using *reset_index(drop=True)*

### Step 5: To SQL
* SQL file was created with *sqlite3.connect()* method, aptly named 'eda_data'. It was created in /data/ folder. 
* The cleaned DataFrames were shipped to the sql file using a simple *to_sql()* method, the table name was set to 'locations'
* Stored data was read using *pd.read_sql()*
    * Select rating and name from locations, order by rating in descending order, and limit results by 10.
    
## Results
I found the coverage was quite similar across both the Foursquare API and Yelp API. 
* Foursquare returned 9 results, Yelp returned 11
<br>
After sorting through the database, it was found that the top music venue based on rating was:
* 1. Bar Cathedral - 54 The Esplanade - 5.0

![bar_cathedral](/images/bar_cathedral.jpg)

## Challenges 
* Yelp data had address field values stored in a list (i.e. ['54 The Esplanade', 'Toronto, ON M5E 1A6', 'Canada'])
    * This returned "InterfaceError: Error binding parameter 1 - probably unsupported type."
    * I had to use a lambda function to convert it to a string, but the end result was the original list encased in string quotations.
    * This could pose an issue in future versions of the application, though most are aesthetic.

## Future Goals
I currently have some loops and alternate functions written outside of this repository. 
#### Short Term
I intend to build a more versatile query building function for the API request urls. I also want a more plug and play user experience, so I would like the user to be able to input an address, and have the application pull that location pin from Maps API, then run it through the venue crawler functions.
#### Long Term
Have the application iterate through a list of addresses/location pins, run the 1km radius functions on all three APIs (Google Places will be initialized by then), and return a list of the top 5 locations based on consumer rating.