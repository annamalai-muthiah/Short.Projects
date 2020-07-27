---
title: "Web Scrapping of Event Listings"
author: "Annamalai Muthiah"
date: "July 15, 2020"
output: 
  html_document: 
    keep_md: yes
---


## 

**Purpose**: If you are interested in planning science, technology or career related online events and you are looking at a few dates (Jan 9, 16 and 23, 2021 in this example) to choose from, one thing you might want to consider doing is to scrap data (event listings) from websites such as eventbrite, Meetup and Charlottesville Business Innovation Council (CBIC) calendar, clean the data and summarize the information to then check what other concurrent and related events are happening on those dates so as to avoid conflicts.

**Acknowledgement**: The document I used to learn of how to scrap web with R:  https://www.freecodecamp.org/news/an-introduction-to-web-scraping-using-r-40284110c848/

**Step 1**. Scrapping data from Eventbrite (Dates: Jan 9, 16 or 23, 2021)<br />
**Step 2**. Copying data from Meetup (Dates: Jan 9, 16 or 23, 2021)<br />
**Step 3**. CBIC calendar (shorturl.at/lACQ0) had no event for Jan 2021 weekends - so no data<br />
**Step 4**. Combine 1, 2 and 3 and remove "virtual game nights" from the listings (there were too many of these and were not science, technology or career related) and filter only for Jan 9, 16 or 23<br /> 
**Step 5**. Filter for key words career, job, tech, code, cville, charlottesville, computer, silicon or science in event names and provide a final tally of events for each date


```r
###################################################Step 1: Scrapping Data from EventBrite ####################
# Loading the library
require(tidyverse) # for data wrangling
require(rvest) # for HTML parsing

# Step 1.1: Make a dataframe to store the data
Event.Brite.Listing = data.frame(Date=character(), Time=character(), Event.Name=character())

# Step 1.2: State dates of interest & total no.of web pages of listings for each date
Date.Numbers= c("9", "16", "23")
Tot.Pages= c(36,35,50)

# Step 1.3: Make a nested loop to extract for each date (outer loop), each page (middle loop) and within each page, all the listings (inner loop). Listings, after extraction, needed a bit of formating before storing in data frame. 
# Note from the author: There is perhaps a more efficient way than using loops. Question might arise why I did not contact eventbrite API. I tried but a reliable API was not available. Instead I used HTML mode examination approach. Though a bit unpredictable, it was successful
for (j in 1:length(Date.Numbers))
{
  No.of.pages = Tot.Pages[j]
  Day = Date.Numbers[j]  
  
  for (k in 1:No.of.pages)
  { 
    # construct the URL of each page
    URL.portion.1='https://www.eventbrite.com/d/online/all-events/?page='
    URL.portion.2='&start_date=2021-01-'
    URL.portion.3='&end_date=2021-01-'
    
    # extracting data from a certain HTML node that carries the listing data
    Page.URL = str_c(URL.portion.1,as.character(k),URL.portion.2,Day,URL.portion.3,Day)
    Listings.on.a.Page = read_html(url <- Page.URL) %>%   html_nodes('.search-event-card-wrapper') %>% html_text() %>% unique()
    No.of.items.on.page=length(Listings.on.a.Page)
    
    #Listings on a page are like blobs and need text manipulation
    for (i in 1:No.of.items.on.page)
    {     
     # Split the string on ")". First piece contains event time. Split it further on "(" and then on year to extract date and time of event separately
      Time.Blob = strsplit(Listings.on.a.Page[i], ")", fixed = TRUE) [[1]][1]
      Event.Date.and.Time = str_trim(strsplit(Time.Blob, "(", fixed = TRUE)[[1]][1]) 
      Event.Date = paste(str_trim(strsplit(Event.Date.and.Time, "2021", fixed=TRUE)[[1]][1]), "2021", sep=" ")
      Event.Time = str_trim(strsplit(Event.Date.and.Time, "2021", fixed = TRUE)[[1]][2])
      
     # Split the string on ")". Second piece contains event name. Use the first word of event name to use as key phrase to parse the blob and use the last piece to construct the event name fully after removing the phrase "to your collection" from the last piece
      Event.Blob = strsplit(Listings.on.a.Page[i], ")", fixed = TRUE)[[1]][2]
      First.of.Event.Words = strsplit(Event.Blob, " ")[[1]][1]
      No.of.Event.References = length(strsplit(Listings.on.a.Page[i], First.of.Event.Words, fixed = TRUE)[[1]])
      Rest.of.Event.Words = strsplit(Listings.on.a.Page[i], First.of.Event.Words, fixed = TRUE)[[1]][No.of.Event.References]
      Event.Name = paste(First.of.Event.Words, Rest.of.Event.Words, sep = "")
      Str.to.be.Removed = "to your collection."
      Event.Name.Proper = gsub(Str.to.be.Removed, "", Event.Name)
      
      # Storing the three pieces of data in the data frame in a cumulative manner
      Event.Brite.Listing = rbind(Event.Brite.Listing, c(Event.Date, Event.Time, Event.Name.Proper))
    }
    # print(k) - to know the progress rate on the middle loop, the longest loop
  }
}

# Step 1.4: Adding column names to the data frame and removing redundancy in data frame through unique() function
colnames(Event.Brite.Listing) = c("Event.Date", "Event.Time", "Event.Name")
Event.Brite.Listing = unique(Event.Brite.Listing)
###############################################End of Step 1: Scrapping Data from EventBrite ####################

############################################### Step 2: Copying data from Meetup ################
# Author's note: I had trouble with the meetupr package from github (installed using remotes::install_github("rladies/meetupr")) that was capable of communicating with Meetup's API. I tried using it but it threw up many errors. I tried solving those error but it was too challenging. Therefore, I gave up.
# Author's note: read_html approach used above with eventbrite involves inspecting html nodes and selecting the correct one. It was not easy to find the nodes to extract the Meetup event details. So I copied the data from the Meetup website, pasted it on to a csv file, read it into R, then formatted the data to extract the event_name, event_date and event_time

# Step 2.1: Data frame to store Meetup listings
Meetup.Listing = data.frame(Date=character(), Time=character(), Event.Name=character())

# Step 2.2: Stating the dates of interest and fragements of addresses of the csv files that contain Meetup event details
Date.Numbers= c("9", "16", "23")
Address.part.1 = "C:/Users/Annamalai/Desktop/Meetup.2021.Jan."
Address.part.2=".csv"

# Step 2.3: Reading in each date's Meetup file data, extracting events' details such as time, date and event name and then storing in the data frame above
for (i in 1:length(Date.Numbers)) {
  
  # Generating file address  
  Folder.Address = str_c(Address.part.1,Date.Numbers[i],Address.part.2)
  # Reading in the file
  meetup.extraction = read_csv(Folder.Address, col_names = FALSE)
  # Number of event blobs in the file
  no.of.elements.in.extraction= dim(meetup.extraction)[1]
  # Creating event date in "Sat, Jan X, 2021" format
  Event.Date = paste("Sat", paste("Jan", Date.Numbers[i], sep = " "), "2021", sep = ", ")
  # Extracting events' times from event blobs - present in every 4th element starting with the first element
  Event.Time = meetup.extraction[seq(1,no.of.elements.in.extraction,4),1]
  # Extracting events' names from event blobs - present in every 4th element starting with the third element
  Event.Name = meetup.extraction[seq(3,no.of.elements.in.extraction,4),1] 
  # Creating a cumulative data frame
  Meetup.Listing = rbind(Meetup.Listing, data.frame(Event.Date, Event.Time, Event.Name))
}

# Step 2.4: Creating column names to the data frame and cutting out redundant rows
colnames(Meetup.Listing) = c("Event.Date", "Event.Time", "Event.Name")
Meetup.Listing = unique(Meetup.Listing)
################################################################ End of Step 2:Copying data from Meetup #################

# Step 3:Due to absence of data from CBIC Calendar, there was not much to do 

################################# Step 4: Combining event listings from Meetup and eventbrite and cleaning up data 

# Step 4.1: Combining the data frames from eventbrite and Meetup and clean-up
Winter.Workshop.2021.Combined.Listing = 
  rbind(cbind(Event.Brite.Listing, Source = "Event.Brite"), 
        cbind(Meetup.Listing, Source = "MeetUp")) %>% 
  # Cleaning up data - filtering only for the dates of interest. %>% operator connects these operations 
  dplyr::filter(Event.Date == "Sat, Jan 9, 2021"|Event.Date == "Sat, Jan 16, 2021"|Event.Date == "Sat, Jan 23, 2021") %>% 
  # There was gratuitous amounts of online game nights on the calendars that have no relevance to our present task. grepl matches the "search word" and slice selects those rows that match
  slice(which(grepl("game night", Event.Name, ignore.case = TRUE) != TRUE))

# Step 4.2: This step summarises the data as number of events counted by date and data source (meet-up or event brite)  
Preliminary.Summary.of.listings = Winter.Workshop.2021.Combined.Listing %>% group_by(Event.Date, Source) %>% dplyr::summarise(no.of.events = n()) %>% spread(Source, no.of.events) # spread function shows the same summary (number of events) with date as row names and source as column names
################################End of Step 4: Combining event listings from meetup and eventbrite and cleaning up data

print(Preliminary.Summary.of.listings)
```

```
  # A tibble: 3 x 3
  # Groups:   Event.Date [3]
    Event.Date        Event.Brite MeetUp
    <chr>                   <int>  <int>
  1 Sat, Jan 16, 2021         347     47
  2 Sat, Jan 23, 2021         574     29
  3 Sat, Jan 9, 2021          386     40
```

```r
######################################Step 5: Selecting events of interests and final summary #####################
Events.of.Interest =
Winter.Workshop.2021.Combined.Listing %>% slice(which(grepl('career|job|tech|code|cville|charlottesville|computer|silicon|science', Event.Name, ignore.case = TRUE) == TRUE)) %>% # searching by key words 
unique() # Ensuring events are unique

print(Events.of.Interest)
```

```
              Event.Date   Event.Time
  25    Sat, Jan 9, 2021  5:00 PM PST
  71    Sat, Jan 9, 2021  1:00 PM PST
  105   Sat, Jan 9, 2021  5:00 PM GMT
  508   Sat, Jan 9, 2021  7:00 AM MST
  533   Sat, Jan 9, 2021  1:00 PM GMT
  639   Sat, Jan 9, 2021 11:30 PM CST
  642   Sat, Jan 9, 2021  2:00 AM EST
  656   Sat, Jan 9, 2021  7:00 AM GMT
  694   Sat, Jan 9, 2021  7:00 PM CET
  736  Sat, Jan 16, 2021  5:00 PM PST
  754  Sat, Jan 16, 2021  9:00 AM CST
  783  Sat, Jan 16, 2021  1:00 PM PST
  819  Sat, Jan 16, 2021  5:00 PM GMT
  1205 Sat, Jan 16, 2021  7:00 AM MST
  1230 Sat, Jan 16, 2021  1:00 PM GMT
  1335 Sat, Jan 16, 2021 11:30 PM CST
  1338 Sat, Jan 16, 2021  2:00 AM EST
  1388 Sat, Jan 16, 2021  7:00 PM CET
  1428 Sat, Jan 23, 2021  5:00 PM PST
  1499 Sat, Jan 23, 2021  5:00 PM GMT
  981  Sat, Jan 23, 2021     10:00 AM
                                                                         Event.Name
  25              Career pathways /counseling and referrals to Silicon Valley jobs 
  71                                    FREE LIVE-Webinar Financial Career Seminar 
  105         Career Head Start Masterclass - IT & Consulting & Project Management 
  508                                             Career Event for Business Majors 
  533          Java Basics in 3 hours.  Code the Hangman Game.  Virtual Classroom. 
  639  Artificial Intelligence | Online Instructor Training at Vepsun Technologies 
  642                                  Science NBE Package (For independent study) 
  656  Business & Professional > Sales & Marketing / Career / Startups & Small Bus 
  694   Der Affiliate Code - Ralf Schmitz - einzigartige Erfolgsgranate  (Klicken) 
  736             Career pathways /counseling and referrals to Silicon Valley jobs 
  754                                         CODE + BREWS Virtual: Third Saturday 
  783                                   FREE LIVE-Webinar Financial Career Seminar 
  819         Career Head Start Masterclass - IT & Consulting & Project Management 
  1205                                            Career Event for Business Majors 
  1230         Java Basics in 3 hours.  Code the Hangman Game.  Virtual Classroom. 
  1335 Artificial Intelligence | Online Instructor Training at Vepsun Technologies 
  1338                                 Science NBE Package (For independent study) 
  1388  Der Affiliate Code - Ralf Schmitz - einzigartige Erfolgsgranate  (Klicken) 
  1428            Career pathways /counseling and referrals to Silicon Valley jobs 
  1499        Career Head Start Masterclass - IT & Consulting & Project Management 
  981                                 LeetCode: online : Questions on google groups
            Source
  25   Event.Brite
  71   Event.Brite
  105  Event.Brite
  508  Event.Brite
  533  Event.Brite
  639  Event.Brite
  642  Event.Brite
  656  Event.Brite
  694  Event.Brite
  736  Event.Brite
  754  Event.Brite
  783  Event.Brite
  819  Event.Brite
  1205 Event.Brite
  1230 Event.Brite
  1335 Event.Brite
  1338 Event.Brite
  1388 Event.Brite
  1428 Event.Brite
  1499 Event.Brite
  981       MeetUp
```

```r
# Final summary of events of interest
Events.of.Interest %>% group_by(Event.Date) %>% dplyr::summarise(no.of.events = n()) %>% arrange(no.of.events)
```

```
  # A tibble: 3 x 2
    Event.Date        no.of.events
    <chr>                    <int>
  1 Sat, Jan 23, 2021            3
  2 Sat, Jan 16, 2021            9
  3 Sat, Jan 9, 2021             9
```

```r
######################################End of Step 5: Selecting events of interests and final summary #####################
```

Based on final summary of events of interest, the date with the least number of events is perhaps the best one. You should still comb through the list of events on that date to conclusively decide if it is really the best date. In this example, Jan 23, 2021 seems like the best date to hold a science, technology or career related online event. 
