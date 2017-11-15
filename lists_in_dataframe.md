Introduction
============

Whether you've read some data from JSON, or otherwise, you've ended up
with these weird lists in columns. What do you do?

We'll use a snippet of data from the Yelp challenge dataset. We've got
some information on a business level (business\_id). One of those
columns is a list of attributes for the business. Now, you're a good
data scientist so you'll be itching to extract these out into columns so
you can use them as features for something cool and interesting.

    attributes <- readRDS("attributes.rds")

    library(tidyverse)

Data format
===========

The data look like this:

    attributes %>% glimpse

    ## Observations: 10
    ## Variables: 16
    ## $ business_id  <chr> "hovoWva_UjbnyLWEbnFvBw", "F53MSa5SYzO9BG8c_JhskQ...
    ## $ name         <chr> "Thai Gourmet", "Pho Viet", "Taste of India", "Ba...
    ## $ neighborhood <chr> "", "", "Southeast", "Northwest", "University Cit...
    ## $ address      <chr> "3732 Darrow Rd Ste 5", "3557 W Dunlap Ave", "497...
    ## $ city         <chr> "Stow", "Phoenix", "Las Vegas", "Las Vegas", "Cha...
    ## $ state        <chr> "OH", "AZ", "NV", "NV", "NC", "AZ", "OH", "AZ", "...
    ## $ postal_code  <chr> "44224", "85051", "89119", "89130", "28213", "853...
    ## $ latitude     <dbl> 41.16571, 33.56705, 36.09914, 36.23846, 35.29493,...
    ## $ longitude    <dbl> -81.44137, -112.13614, -115.13619, -115.23195, -8...
    ## $ stars        <dbl> 3.5, 2.5, 3.5, 3.5, 4.0, 4.0, 4.0, 4.0, 3.5, 3.5
    ## $ review_count <int> 68, 3, 33, 9, 5, 273, 63, 51, 7, 34
    ## $ is_open      <int> 1, 0, 1, 0, 0, 1, 1, 1, 0, 0
    ## $ attributes   <list> [<"Alcohol: full_bar", "Ambience: {'romantic': F...
    ## $ categories   <list> [<"Thai", "Restaurants">, <"Vietnamese", "Restau...
    ## $ hours        <list> [<"Monday 11:0-21:30", "Tuesday 11:0-21:30", "We...
    ## $ type         <chr> "business", "business", "business", "business", "...

In order to focus on the key columns of interest, we'll just grab the
business\_id and attributes columns to reduce the clutter. They're the
bits we're interested in and we can always join the data back up later
anyway.

    # and key things we're interested in is attributes for business_id:
    attributes %>% 
        select(business_id, attributes) 

    ## # A tibble: 10 x 2
    ##               business_id attributes
    ##                     <chr>     <list>
    ##  1 hovoWva_UjbnyLWEbnFvBw <chr [19]>
    ##  2 F53MSa5SYzO9BG8c_JhskQ     <NULL>
    ##  3 hMh9XOwNQcu31NAOCqhAEw <chr [19]>
    ##  4 kUUBBLBHCasOl2a5nW9nAw <chr [14]>
    ##  5 2rgQ1TULwVoY7TnUlnH7Yw <chr [14]>
    ##  6 2px99IppAcnxR238eq_8_w <chr [21]>
    ##  7 Eq3qA7F5uZBUbcYXROzntA <chr [19]>
    ##  8 Ld2hhA3q3cdkptwS1fsYEg <chr [19]>
    ##  9 -vb_yx5QnIhpXUIdPVD2og <chr [19]>
    ## 10 3rkxTx8DoZSl7_FryhXCVQ <chr [17]>

    # let's start testing ideas on the first element
    tmp <- attributes$attributes[1]

    # extracting successive elements
    tmp %>% map_chr(1)

    ## [1] "Alcohol: full_bar"

    tmp %>% map_chr(2)

    ## [1] "Ambience: {'romantic': False, 'intimate': False, 'classy': False, 'hipster': False, 'divey': False, 'touristy': False, 'trendy': False, 'upscale': False, 'casual': True}"

    tmp %>% map_chr(3)

    ## [1] "BikeParking: True"

    tmp %>% map_chr(4)

    ## [1] "BusinessAcceptsCreditCards: True"

Okay, so we might be onto something here. We seem to have been able to
extract key:value pairs from the list. It's a start! But we don't want
to have to manually specify list index for each row. We want to end up
automatically generating a set of column features from whatever we've
got in our list attributes.

Getting there
=============

We might first think of unnest. Great, this will do what we want, right?

    try(
    attributes %>% 
        select(business_id, attributes) %>% 
        unnest )

Okay, it didn't like that. What went wrong? We're not taking into
account the presence of some empty attribute lists. Can we filter on the
length of each? Let's try creating a length feature

    attributes %>% 
        select(business_id, attributes) %>% 
        mutate(Num_attributes = length(attributes))

    ## # A tibble: 10 x 3
    ##               business_id attributes Num_attributes
    ##                     <chr>     <list>          <int>
    ##  1 hovoWva_UjbnyLWEbnFvBw <chr [19]>             10
    ##  2 F53MSa5SYzO9BG8c_JhskQ     <NULL>             10
    ##  3 hMh9XOwNQcu31NAOCqhAEw <chr [19]>             10
    ##  4 kUUBBLBHCasOl2a5nW9nAw <chr [14]>             10
    ##  5 2rgQ1TULwVoY7TnUlnH7Yw <chr [14]>             10
    ##  6 2px99IppAcnxR238eq_8_w <chr [21]>             10
    ##  7 Eq3qA7F5uZBUbcYXROzntA <chr [19]>             10
    ##  8 Ld2hhA3q3cdkptwS1fsYEg <chr [19]>             10
    ##  9 -vb_yx5QnIhpXUIdPVD2og <chr [19]>             10
    ## 10 3rkxTx8DoZSl7_FryhXCVQ <chr [17]>             10

Well they're not all length 10. We know that. We've just calculated the
length of the whole column (10 rows). Let's try again, this time using
map\_int to return the expected integers

    attributes %>% 
        select(business_id, attributes) %>% 
        mutate(num_atts = map_int(attributes, length)) 

    ## # A tibble: 10 x 3
    ##               business_id attributes num_atts
    ##                     <chr>     <list>    <int>
    ##  1 hovoWva_UjbnyLWEbnFvBw <chr [19]>       19
    ##  2 F53MSa5SYzO9BG8c_JhskQ     <NULL>        0
    ##  3 hMh9XOwNQcu31NAOCqhAEw <chr [19]>       19
    ##  4 kUUBBLBHCasOl2a5nW9nAw <chr [14]>       14
    ##  5 2rgQ1TULwVoY7TnUlnH7Yw <chr [14]>       14
    ##  6 2px99IppAcnxR238eq_8_w <chr [21]>       21
    ##  7 Eq3qA7F5uZBUbcYXROzntA <chr [19]>       19
    ##  8 Ld2hhA3q3cdkptwS1fsYEg <chr [19]>       19
    ##  9 -vb_yx5QnIhpXUIdPVD2og <chr [19]>       19
    ## 10 3rkxTx8DoZSl7_FryhXCVQ <chr [17]>       17

Now that's more like it! Let's try unnest again now we can filter out
that pesky null attribute.

    attributes %>% 
        select(business_id, attributes) %>% 
        mutate(num_atts = map_int(attributes, length)) %>% 
        filter(num_atts > 0) %>% 
        unnest %>% 
        separate(attributes, into = c("key", "value"), extra = "merge") 

    ## # A tibble: 161 x 4
    ##               business_id num_atts                        key
    ##  *                  <chr>    <int>                      <chr>
    ##  1 hovoWva_UjbnyLWEbnFvBw       19                    Alcohol
    ##  2 hovoWva_UjbnyLWEbnFvBw       19                   Ambience
    ##  3 hovoWva_UjbnyLWEbnFvBw       19                BikeParking
    ##  4 hovoWva_UjbnyLWEbnFvBw       19 BusinessAcceptsCreditCards
    ##  5 hovoWva_UjbnyLWEbnFvBw       19            BusinessParking
    ##  6 hovoWva_UjbnyLWEbnFvBw       19                GoodForKids
    ##  7 hovoWva_UjbnyLWEbnFvBw       19                GoodForMeal
    ##  8 hovoWva_UjbnyLWEbnFvBw       19                      HasTV
    ##  9 hovoWva_UjbnyLWEbnFvBw       19                 NoiseLevel
    ## 10 hovoWva_UjbnyLWEbnFvBw       19             OutdoorSeating
    ## # ... with 151 more rows, and 1 more variables: value <chr>

This is looking much more hopeful! We have successfully extracted the
first attribute key. The final column, the value part, happily doesn't
show because some are fairly long.

All we need to do now is to use the good old spread function from tidyr
to give us our feature columns for our final result!

Extracting features
===================

    attributes_unnested <- attributes %>% 
        select(business_id, attributes) %>% 
        mutate(num_atts = map_int(attributes, length)) %>% 
        filter(num_atts > 0) %>% 
        unnest %>% 
        separate(attributes, into = c("key", "value"), extra = "merge") %>% 
        spread(key, value) 

    attributes_unnested %>% glimpse

    ## Observations: 9
    ## Variables: 27
    ## $ business_id                <chr> "2px99IppAcnxR238eq_8_w", "2rgQ1TUL...
    ## $ num_atts                   <int> 21, 14, 17, 19, 19, 19, 14, 19, 19
    ## $ Alcohol                    <chr> "beer_and_wine", "full_bar", "none"...
    ## $ Ambience                   <chr> "romantic': False, 'intimate': Fals...
    ## $ BikeParking                <chr> "True", NA, NA, "True", "True", "Tr...
    ## $ BusinessAcceptsCreditCards <chr> "True", "True", "True", "True", "Tr...
    ## $ BusinessParking            <chr> "garage': False, 'street': False, '...
    ## $ Caters                     <chr> "True", NA, "False", "True", "True"...
    ## $ CoatCheck                  <chr> NA, NA, NA, NA, NA, NA, "False", NA...
    ## $ DogsAllowed                <chr> "True", NA, NA, NA, NA, NA, NA, NA, NA
    ## $ GoodForDancing             <chr> NA, NA, NA, NA, NA, NA, "True", NA, NA
    ## $ GoodForKids                <chr> "True", "True", "True", "True", "Tr...
    ## $ GoodForMeal                <chr> "dessert': False, 'latenight': Fals...
    ## $ HappyHour                  <chr> NA, NA, NA, NA, NA, NA, "True", NA, NA
    ## $ HasTV                      <chr> "True", NA, "False", "True", "True"...
    ## $ Music                      <chr> NA, NA, NA, NA, NA, NA, "dj': True,...
    ## $ NoiseLevel                 <chr> "average", NA, "quiet", "quiet", "q...
    ## $ OutdoorSeating             <chr> "True", "True", "False", "False", "...
    ## $ RestaurantsAttire          <chr> "casual", "casual", "casual", "casu...
    ## $ RestaurantsDelivery        <chr> "False", "False", NA, "False", "Tru...
    ## $ RestaurantsGoodForGroups   <chr> "True", "True", "True", "True", "Tr...
    ## $ RestaurantsPriceRange2     <chr> "1", "2", "2", "2", "2", "2", "1", ...
    ## $ RestaurantsReservations    <chr> "False", "True", "False", "True", "...
    ## $ RestaurantsTableService    <chr> "True", "True", "True", "True", "Tr...
    ## $ RestaurantsTakeOut         <chr> "True", "True", "True", "True", "Tr...
    ## $ WheelchairAccessible       <chr> "True", "True", NA, NA, NA, "True",...
    ## $ WiFi                       <chr> "free", NA, "no", "no", "free", "no...

We can now start to get on with our usual data cleaning. For example,
what values do we have for "WiFi"?

    attributes_unnested %>%
        count(WiFi)

    ## # A tibble: 3 x 2
    ##    WiFi     n
    ##   <chr> <int>
    ## 1  free     2
    ## 2    no     5
    ## 3  <NA>     2

Further work
============

As you can see, there are still some features we might want to extract
further, such as Ambience. This is for another day, but you might decide
where you want to go with that is to split the values out into yet more
columns having, perhaps, prepended "Ambience:" to each subkey to
generate "Ambience:romantic" and "Ambience:intimate" features, for
example.
