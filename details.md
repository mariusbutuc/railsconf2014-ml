# Machine Learning for Fun and Profit (with Ruby) -- The Talk

## Prep

* get names from users so we can use them as gender categories, also their country if its not the US?

## Intro

Machine learning. Big Data. Data science.

Now that many online businesses operate at "web scale" and collect data about every aspect of their operation, the ability to *do something* with all that data is getting a lot of attention. And even after sorting through the hype from vendors, there's certainly value in that data and the tools that unlock it. 

[The Underwear Gnomes: Step 2 is often data science]

There are a *LOT* of tools involved in the data science world and Ruby really isn't one of the major players. Python is probably the most common general purpose language while the more specialized mathematical languages like Julia, R, Octave (FOSS clone of Matlab) play a crucial role. There are also specialized databases (InfluxDB, the new WebScaleSQL engine for MySQL), specialized platforms (Hadoop and friends), and even specialized _hardware_ for data science.

This workshop is going to ease you into the world of data science and machine learning by focusing on something we've all seen in nearly every Rails application -- the Users table. We are going to use Ruby to apply an array of different tools and techniques to our Users table to extract useful data that can guide real business and operational decisions.

### Format
We're going to walk through a set of exercises that starts with a real question from the business team and then find a way to answer the question with the data we have in the Users table. I'm going walk through the steps and then we're each going to implement it, though you're welcome to work together in pairs. 

### Setup

I'm assuming you have a Ruby install and are familiar with bundler. You're welcome to use your own users table, but I'm supplying a simple one in sqlite3 format (along with MySQL and Postgresql and CSV). One of our gems *requires Ruby 1.9.3* -- I use 1.9.3-p194 -- and is sensitive to underlying C bindings. I am not much of a C programmer and since the C then binds to Fortran under the hood, I'm out of my depth dealing with it :)

You should be able to clone the git repository and run bundler in the root directory which should give us everything we need. If you only have ruby 2.0+, you can change the ruby version in your Gemfile and we'll just skip over the problematic gems.

## Exercise 1: Gender

### The problem

Question: We're going to sell logo T-shirts. What proportion of women's cut T-Shirts should we order?

Let's think about your signup form -- do you ask for a gender? [Sidenote: How many? Facebook offers ~50 different options now]. Do you even *want* to? Asking more questions can slow the signup process, reduce your conversion rate, and doesn't help backfill your information.

What are the solutions?

* Start asking the question, add it to profile (completeness %, etc)
* A sample survey, possibly with a bonus/contest
* Other ideas?

Our first question is about value -- while its a no-brainer to gain some goodwill by offering a t-shirt cut for women, we are not an ecommerce company and don't have space in our startup for dealing with a lot of inventory nor do we want to order in small one-off batches. Our vendor knows how to handle the distribution of sizes but we'd like to get a solid guess about the gender difference.

We don't have to have an exact number -- we're probably fine with a percentage that's rounded to the nearest 5 or even 10%. We can assume 50/50, but we'd now we're also curious about the gender ratio for fun.

How can we decide _today_? And without spending a lot of money on a survey, marketing data, or some other expensive or timeconsuming option? Is there an acceptable proxy for gender?

First name might be a good candidate

John => male,
Susan => female

Looking good!

Kim => female

Kim Stanley Robinson is male
Kim Mohan (Dragon Magazine FTW!) is male

Well, they are certainly an exception to the rule, but we don't need precision. But let's not show this information on their public profile! But we can agree Kim is generally a female name.

River => ?
Cedar => ?
Justice => ?

Hmmm, problem is harder than it seemed. 

And another one

Jamie => ?

This one probably depends on where you're from and your perspective on internationalization. Its traditionally a female name in the US but a more masculine name in Great Britain for example. 

So is this going to be useful? Will it be better than a survey? Probably -- survey response rates are terrible and the extrapolation from the sample may be just as bad or worse.

https://github.com/bmuller/sexmachine

### Results

For this data set, 

mostly_male 7
mostly_female 3
female 11
male 86
andy 62

169 users

	* 08% (11+3 / 169) female
	* 55% (86+7 /169) male
	* 37% unknown
	  

There were 50 junk users (demo, test, etc) assigned to the andy group, so after removing them

mostly_male	7
mostly_female	1
female	11
male	86
andy	14

119 users

	* 11% (11+1 / 119) female
	* 78% (86+7 /119) male
	* 11% unknown

Hand coding the remaining andy folks, just for fun

119 users

	* 11% (13/119) female
	* 89% (106/119) male

### Lessons Learned

As far as the data

	* Clean the junk!!
	* extra spaces a prob, chomp!
	* hyphenated, initials,  and compound was a problem (Jean-Louis, John Paul)
	* Lots of Chris, mostly_male

As far as the results

	* Quick and dirty analysis with data said ~10% female.
	* Refined and cleaned data said ~10%
	* Actual data (hand coded by me) ~10%

Thoughts?

## Exercise 2: Geolocation

### Problem
Thanks to explosive subscription growth, our support team is growing! We want to make sure we cover the bulk of our customers with support hours, but we can't afford a round-the-clock solution. 

Question: Where do most of our students live?

What are the possible solutions? We could try the same common set of solutions

* Start asking the question, add it to profile (completeness %, etc)
* A sample survey, possibly with a bonus/contest
* Other ideas?

with the same cost and completeness problems. There's got to be a better way!


How accurate do we have to be? We can start with a first pass analysis to find countries and continents which would show the distribution between US, Europe, and Asia.

--------------------------------------------

Question: We have a great English language site. What countries should we spend marketing dollars and technical on (i18n, payment processors, physical presence)

What are the possible solutions? We could try the same common set of solutions

* Start asking the question, add it to profile (completeness %, etc)
* A sample survey, possibly with a bonus/contest
* sniff browser language preferences
* payment information
* Other ideas?

with the same cost and completeness problems. There's got to be a better way!

Many implementations of user authentication end up storing IP address information in the User table -- specifically the last IP address used by that specific user. We can use geolocation APIs to map the IP to location data and solve our problem.

IP addresses are not a perfect solution thanks to VPNs, mobile access, and travel, but the data is *right there* if we have the tools to handle it. Our solution only needs to assign people to countries or regions so the accuracy doesn't have to be high.

### Code

There are myriad web services for geolocating data as well as free databases. I'm going to use a free service which also is available as open source -- https://github.com/fiorix/freegeoip. It combines the freely available limited version of the commerical MaxMind database with some additional custom data using some Python scripts and runs as a local Go-based webservice. Ployglot for the win!

	`$ cd $GOPATH/src/github.com/fiorix/freegeoip`
	`$ cd db`
	`$ ./updatedb`
	`$ cd ..`
	`$ ./freegeoip`

### Results

	demo.group_and_count(:country_name).all
	=> [{:country_name=>"Canada", :count=>2},
 	{:country_name=>"Germany", :count=>1},
 	{:country_name=>"Reserved", :count=>3},
 	{:country_name=>"United Kingdom", :count=>5},
 	{:country_name=>"United States", :count=>80}]

Looks like everyone is in North America with a few European outliers. We'll be fine with English it seems. Though we should probably look at Spanish since that a significant secondary language in the US. We can also dig in to Canada to see if its Quebec and we should think about French.

	demo.group_and_count(:country_name, :region_name).order(:count).all
	=> [{:country_name=>"Germany", :region_name=>"Hamburg", :count=>1},
	 {:country_name=>"United Kingdom", :region_name=>"Tameside", :count=>1},
	 {:country_name=>"United States", :region_name=>"Idaho", :count=>1},
	 {:country_name=>"United States", :region_name=>"Massachusetts", :count=>1},
	 {:country_name=>"United States", :region_name=>"South Carolina", :count=>1},
	 {:country_name=>"United States", :region_name=>"Tennessee", :count=>1},
	 {:country_name=>"Canada", :region_name=>"Ontario", :count=>2},
	 {:country_name=>"United Kingdom", :region_name=>"", :count=>2},
	 {:country_name=>"United Kingdom", :region_name=>"Manchester", :count=>2},
	 {:country_name=>"United States", :region_name=>"Illinois", :count=>2},
	 {:country_name=>"United States", :region_name=>"North Carolina", :count=>2},
	 {:country_name=>"Reserved", :region_name=>"", :count=>3},
	 {:country_name=>"United States", :region_name=>"", :count=>3},
	 {:country_name=>"United States", :region_name=>"Maryland", :count=>3},
	 {:country_name=>"United States", :region_name=>"New York", :count=>4},
	 {:country_name=>"United States", :region_name=>"Georgia", :count=>5},
	 {:country_name=>"United States", :region_name=>"California", :count=>6},
	 {:country_name=>"United States", :region_name=>"Oregon", :count=>17},
	 {:country_name=>"United States", :region_name=>"Florida", :count=>34}]
	 
Looks like a pretty big East/West Coast split!

What else can we do? We can look at timezones for support hours, look for places to try meatspace events (Florida looks good!), and even create a pretty map.



### Lessons Learned

Geolocation is not hard when you start with an IP address, especially if you can sacrifice so data and allow some ambiguity.

## Exercise 3: Grouping (Categorization)

### Problem
Our subscription base is growing like crazy! We also have a great new companion feature to our toolset. We believe we have at least three different sets of subscribers -- hobbyists, professionals, and superusers -- and want to target each of those groups differently to start marketing our latest product.

Question: How do we divide our users into our three sets? 



### Code

How many segments, a-priori? k = Sqrt(n/2). Is one rubric. Many more. 

### Results
### Lessons Learned

## Exercise 4: Similarity

### Problem
We just implemented social networking on the site! Users can follow, friend, and like each other just like Facebook. But to really get things moving in the right direction, we want to recommend people to follow who are like them. How do we do it?

Question: 

### Code
### Results
### Lessons Learned

## Bonus Round
### Data Warehousing - Star Schema for Users
### Tools of the trade: Julia, Python, R, Octave, etc
### Using R with Ruby
### Frontiers



## The proposal

### Abstract:

Your Rails app is full of data that can (and should!) be turned into useful information with some simple machine learning techniqiues. We'll look at basic techniques that are both immediately applicable and the foundation for more advanced analysis -- starting with your Users table.

We will cover the basics of assigning users to categories, segmenting users by behavior, and simple recommendation algorithms. Come as a Rails dev, leave a data scientist.

### Details:

My goal is to show Rails developers they can use machine learning techniques without leaving the comfort of Ruby and Rails. Nearly all Rails applications have a Users table of some kind and we'll use that common starting point to demonstrate a few simple yet powerful techniques for machine learning.

Our first task is to assign users to male and female categories based on their first name. We'll use a library for this but look at the techniques that make it work. Participants should be able to immediately apply this to their own apps.

Our next technique will let us cluster users using a hierarchical technique. This technique can be used to segment users based on specific attributes, such as overlapping beliefs, values, or behaviors.

Finally we'll look at using K-means to recommend similar content (eg blog posts) to users based on their reading behavior.

I'm going to do as much of this as possible with real data from users in the audience -- assuming we can get solid enough internet access that they can fill out a form.

As a bonus, we can talk about integrating R into Rails applications if there's any interest.

## TL;DR

Every app has a users table.

### Gender

* Gender can be assigned based on first-name. Based on census/reporting data, normalized by country.
* Five options: male, female, mostly_male/female, unknown (andy)
* Original c code and data written 5y ago by [citation needed]
* YMMV!!!!! Hello Kim Stanley Robinson. John Paul is "andy"

### Location

* Plenty of FOSS options, also paid. Can use http://freegeoip.net locally!
* Store it when they login, track over time. Not a dw talk tho

### Grouping/Hiearchy

* Many ways to generate groups, but generally you map to N-dimensional space and calculate a distance between each group (cluster). Merge the closest and repeat.

#### Simple cluster
* Finding your rockstar users by badges

#### More complex clustering
* breaking it out by points
* differences between algorithms

#### Stop conditions
* by group count
* by difference



