---
title: "Web Scraping with rvest"
author: "Kamil Klosek"
date: "3/27/2021"
output:
  html_document:
    keep_md: true
---



## Introduction

In this module, we are going to learn how to harness HTML source code for web scraping using the rvest pacakge. Therefore, knowledge in HTML/XML and CSS are highly useful for this module. Our goals are:  
  
1. Estimate size of the workload
2. Automatize data collection process
3. Archive raw data
4. Extract and format data

Alternatively, some web services allow us to download data using APIs. This will be the content of the next module.

## Legal side of web scraping

There are at least two points to consider before engaging in web scraping:

1. Make sure that it is permissible to download specific content 
- Read /robots.txt file
- Read Terms of Agreement if you extract data from a password-protected site
2. Minimize your web traffic
- Before considering web scraping, check whether the website provides a database or API (use those first!)
- Implement reasonable request delays, i.e. Crawl-delays (might be mentioned in robots.txt)
- Consider scraping the website in off-peak hours (e.g. at night)
3. Identify yourself through a User Agent

Every website (should) have a robots.txt file, accessible through root-homepage-name/robots.txt in the URL-field in your browser, which specifies which directories are Allowed/Disallowed to scrape from.

Be mindful that web scraping creates traffic. If too many people engage in web scraping at the same time, you can inadvertently make it more difficult for other users to access the website. In worst case, you might crash the hosing server. If you are lucky, nothings happens, otherwise your IP address might be blocked (which would be unfortunate if you access websites from an institutional VPN), and in worst case you will get a letter from a lawyer demanding compensation for the damage you created. Therefore, behave in an appropriate and mindful manner when you engage in web scraping.

If you web scrape inside the European Union, consider reading the [directive 2019/790](https://eur-lex.europa.eu/eli/dir/2019/790/oj) on copyright and related rights.

## HTML/XML syntax

HTML stand for **H**yper **T**ext **M**arkup **L**anguage and is an offshoot of XML which stands for **Ex**tensible **M**arkup **L**anguage. HTML forms the backbone of the majority of sites on the web, therefore it is crucial to understand its fundamental structure from the perspective of a web scraper.

In markup language, plain text is categorized using tags with **<>** angle brackets. For instance, in HTML the tag `<title>` tells your browser to display *Title Name* as title: `<title>Title Name</title>`. Together the combination of tags and text is referred to as *element*. While the syntax is the same, the meaning of tags in HTML are strictly defined by the [World Wide Web Consortium (W3C)](https://www.w3schools.com/html/), whereas XML allows the user to create own tags. Another important feature for us are **attributes**. Attributes can be attached to tags, e.g. `<a href="URL">Text</a>`. In this example, we assign a URL to text embedded in the `<a></a>` tag. 
  
Another important feature of HTML/XML is their tree-structure. Such tree-structured text is also called **Document Object Model (DOM)**. Tags like `<body></body>` can include multiple tags, e.g. `<title>`, `<p>`, or `<ul>` to name a few. This nested form allows us web scrapers to pinpoint at those tags which as closely as possible define the information we want to extract. For instance, in a structure such as `<body><p>Text<ul>Text</ul></p></body>` we might want to extract information provided by an unstructured list (ul) by singling out **ul** as our target tag. Another way of thinking about tags in a tree-structure is to view them as nodes that can split into several additional nodes.

Lastly, HTML source code can contain **CSS (Cascading Style Sheets)** which is used to format the layout of websites. Website developers define tags with CSS information, e.g. `p.highlight` where `p` stands for the paragraph tag `<p>` and `highlight` represents a specific layout style provided in curly brackets: `p.highlight {color : green; font-size:150%}`. The source code can then contain the CSS information using the class attribute: `<p class = "highlight">Text</p>`. One can also define the CSS style directly within a tag, but this becomes tedious and is less readable, especially when many different styles are used.

## XPATH

**XPATH** stands for **X**ML **P**ath **L**anguage and is a web technology that allows the user to extract information from HTML/XML documents. Since their structure is hierarchical, we can move alongside tags in the tree from the highest-order tags (e.g. `/html/body`) to lower-order tags (e.g. `/div/p/ul`). While we can use absolute paths (`/html/body/div/p/ul`), we might also use relative paths using `//` (e.g. `//p/ul`). This means that we extract information from all `<ul>` which are a node below all `<p>`. We however disregard those `<ul>` which are not connected with a `<p>`.

XPATH uses some characters with special meaning:

- The star * performs as a wildcard operator. This means that inserting * into a path such as `/body/div/*/ul` leads to identifying information that starts at body, moves along to div and then takes all tags that contain an ul-tag into consideration.
- The vertical bar `|` allows us to select multiple paths in one query, e.g. `//p | //a`
- The `@` symbol allows us to select attributes within `<>` tags, e.g. `p[@class=...]`


XPATH also the ability to create very precise paths by specifying unique relationships between nodes. For instance, the relative path `/p/i` provides us with information from the `<i>` node with a preceding `<p>` node, but what if there are multiple p-i nodes across branches and we are only interested in a particular branch? For this, we can harness the hierarchical tree-structure in XPATH which allows us to distinguish branches. The following table summarizes which node relations can be used and can also be found [here](https://www.w3schools.com/xml/xpath_axes.asp).

| AxisName           | Result                                                                                                                       | 
| :----------------- | :--------------------------------------------------------------------------------------------------------------------------- | 
| ancestor           | Selects all ancestors (parent, grandparent, etc.) of the current node                                                        |
| ancestor-or-self   | Selects all ancestors (parent, grandparent, etc.) of the current node and the current node itself                            |
| attribute          | Selects all attributes of the current node                                                                                   | 
| child              | Selects all children of the current node                                                                                     |
| descendant         | Selects all descendants (children, grandchildren, etc.) of the current node                                                  |
| descendant-or-self | Selects all descendants (children, grandchildren, etc.) of the current node and the current node itself                      |
| following          | Selects everything in the document after the closing tag of the current node                                                 |
| following-sibling  | Selects all siblings after the current node                                                                                  |
| namespace          | Selects all namespace nodes of the current node                                                                              |
| parent             | Selects the parent of the current node                                                                                       |
| preceding          | Selects all nodes that appear before the current node in the document, except ancestors, attribute nodes and namespace nodes |
| preceding-sibling  | Selects all siblings before the current node                                                                                 |
| self               | Selects the current node                                                                                                     |


For instance, the expression `//p/ancestor::a` selects all those `<p>` nodes which have somewhere in their branch a lower-order `<a>` node. The expression `//div/parent::title` selects those `<div>` nodes with `<title>` as their parent.
  
Lastly, we can use **predicates** in brackets `[]` to create numeric or textual conditional TRUE/FALSE statements. The following expressions are a non-exhaustive list of the probably most useful predicates:

| Predicate                  | Effect                                                                                    |
| :------------------------- | :---------------------------------------------------------------------------------------- |
| `/div/h1[1]`               | Select the first H1 heading that is part of a `<div>` branch                              |
| `/div/h1[last()]`          | Select the last H1 heading that is part of a `<div>` branch                               |
| `/div/h1[position() >= 2]` | Select the second and following H1 headings that are part of a `<div>` branch             |
| `/div/h1[@href]`           | Select those H1 headings which contain a `href` attribute and are part of a `<div>` branch |


## How to identify the information I need in the HTML source code

Now since we know how HTML/XML code is structured, and we have an idea how to access branches, we need to identify those elements that contain the information we are looking for. However, there is unfortunately no water-proof way but there are tools and ways how to facilitate the process.

The most useful tool is [SelectorGadget](https://chrome.google.com/webstore/detail/selectorgadget/mhjhnkcfbdhnjickkkdbjoemdmbfginb) which is available as Chrome extension. Selector gadget allows you to point-and-click at elements of a website that contain your desired information. Make sure that only information is highlighted in green/yellow that contains what you want to extract. In best circumstances, you will obtain a tag or a combination of tags that we can use later as nodes to separate irrelevant from relevant information.

Should you fail to obtain some straightforward combination of tags, the second option is to use **XPATH**. You can use SelectorGadget to obtain the XPATH or you right-click on your desired information and select *inspect* (on Chrome). Evaluate the highlighted source code and see whether you can find a pattern that connects the data you are looking for with a combination of tags. For instance, you might be interested in obtaining the titles of articles and you discover that those titles you are searching for all use class="media_tag2" within an `<a>` tag. With this information at hand, you can use XPATH to directly access all text with `<a class="media_tag2">`.

## Using HTML for web scraping in R

In R, we typically engage in the following process:

1. Download the HTML source code
2. Parse the source code to create a DOM (rvest package does it automatically)
3. Harness the tree-structure of your DOM to identify elements with relevant information
4. Use R functions to extract information
5. Create data objects such as lists or data frames suitable for data analysis

## Setting-up your user agent

When you communicate with web servers, it is good practise to insert your name and email address so that the webmaster knows that you are a person and not a bot. Do not provide credentials of someone else other than you! This is called spoofing and illegal. 


```r
library(httr)
agent <- user_agent(agent = "your email address; your project; additional information")
set_config(agent)
```

## Example 1: Extracting text from one website

Here, we are going to extract the main text of a website. For our purposes, we use the website https://fivethirtyeight.com/politics/ 
First of all, we have to check the robots.txt file: https://fivethirtyeight.com/robots.txt
  
We get the following output: 
`User-agent: *`
`Disallow: /wp-admin/`
`Allow: /wp-admin/admin-ajax.php`  
  
This means that:

1. A user agent with * refers to everybody who wants to scrape data from the website
2. We are forbidden to scrape data from the path  https://fivethirtyeight.com/wp-admin/*
3. The third condition is not relevant for our purposes
4. What is missing here is a Crawl-delay argument specifing how much time should pass between each request. It is always prudent to have 10 seconds delay.

This already looks promising. The website administrator allows us to web scrape this particular website with the one aforementioned exception. Let us now have a look at a particular article: https://fivethirtyeight.com/features/why-georgias-new-voting-law-is-such-a-big-deal/  

The article consists of a headline, text, and much more information. What we can do now is to use SelectorGadget in Chrome and click on the first paragraph starting with "From November 2020 to January 2021....". What you can see is that each paragraph is highlighted with a yellow background except the the first paragraph which is green. Selector gadget also shows you that all those paragraphs share the `<p>` tag. That is important information because it means that we can find all text using the `<p>` tag. However, there is one (or more) problems. SelectorGadget also highlights the date, author, an additional attribute below the author name, the source of the picture, and a photo plus link in the middle of the text. They are highlighted because they, too, share the `<p>` tag. Hence, we have to deselect the unwanted information by clicking at the date and the link "Related". SelectorGadget now provides us with the term ".single-post-content > p" that we can use to identify excacly the main text of the article.  

Now it is time to use the power of the rvest package. Load the rvest package and the tidyverse into R using the `library()` function:


```r
library(rvest)
library(tidyverse)
library(httr)
```

We create now a variable url, where we store the link to our article. We then use the `read_html()` function to download the HTML source code. Under the hood, the function already parses the source code into a DOM object with tree-structure. We store the DOM in the object page.


```r
url <- "https://fivethirtyeight.com/features/why-georgias-new-voting-law-is-such-a-big-deal/"
page <- read_html(x = url)
```

In this step, it is time to extract the information about the text using the `html_elements()` and `html_text()` functions.


```r
text <- 
  page %>% 
  html_elements(".single-post-content > p") %>% 
  html_text(trim = TRUE) # Remove whitespace at the beginning and end of a string.
```

You should find in your R environment the object `text` which contains eight different strings each representing one paragraph from the original text. If we wanted to, we could paste it together with the `toString()` function.


```r
text <- toString(text)
substr(x = text, start = 1, stop = 199) # show the first sentence
```

```
## [1] "From November 2020 to January 2021, the story of the state of Georgia was pro-Democratic: Democratic candidates for president and the U.S. Senate all won. But more importantly, it was pro-democratic."
```

So, we learned in this example:

1. How to read a robots.txt file
2. How to use SelectorGadget to find a combination of CSS and tags that uniquely identifies what we want
3. How to create a DOM with `read_html()`
4. How to extract information from the DOM with `html_elements()` and `html_text()`


## Example 2: Create a dataset of recently published articles

This example will be very comprehensive and feature many aspects that are relevant for web scraping. We will make use of the fact that the URL of the FiveThirtyEight (Politics) website displays the page number. For instance `https://fivethirtyeight.com/politics/features/page/3/` will bring us to page 3. Hence, we can iterate over different pages and extract information about articles from each page.

In this task, we are going to create a dataset listing all published articles available from page one until page ten with the name of the author(s), publication date, title, link, and category. First of all, we have to define the appropriate URL.


```r
url <- "https://fivethirtyeight.com/politics/features/page/"
```

In the following step, we create an empty list within which we store our temporary data.


```r
dfs <- list()
```

Further, let us create a subfolder in our project where we would like to store raw HTML source code. We do this because when we run an analysis, it would be tedious as well as time- and resource inefficient to constantly scrape data from websites we have already scraped.


```r
data.directory <- "./source_code/"
if (!dir.exists(data.directory)) dir.create(data.directory)
```

Now the long and tedious part begins. We have to conduct the following steps:

1. Create a for-loop that loops over pages 1 to 10
2. Create for each loop a URL with the appropriate page number
3. Pull out from each site information on title, author, date, link, and category
4. Store the data from one page as data frame in our list. Each list element is a data frame, so in the end we will have 10 elements
5. Include a time-delay of 10 seconds plus variation to respect server capacities
6. Execute the for-loop
7. Combine all data frames into one using the function `rbindlist()` from the `data.table` package

**Caution**: Retrieving data on authors is not straightforward, as each author name is saved as a separate element within the source code. Simply retrieving author names gives us a vector with more authors than there are articles. Hence, to obtain author names we retrieve information on each post and then use regular expressions to extract author details.

**Caution**: The first article on the first page is somewhat differently embedded in the HTML source code than all following articles. Hence, separate code is written to retrive information on date and author names.

**Caution**: Unfortunately, there can be in very seldom cases missing information for categories. Here, I include a loop with sapply that loops over the lowest common node that each article must have. Then, I executed another loop to validate that the desired node was indeed available. If not, I imputed an `NA`.

Before we start executing program that loops through the different pages, I create a custom function to clean the data that we extract for authors. It removes unnecessary text and produces as output a tibble.


```r
CleanAuthorText <- function(x, firstPage = i){
  
  # Different extraction rule for the first page
  if(firstPage == 1){
    RegEx <- "\\tBy .*" # First article on the first page does not end with \\n
  } else {
    RegEx <- "\\tBy .*?\\n" 
  }
  
  x <- 
    x %>% 
    str_extract(pattern = RegEx) %>% # Extract author names
    str_remove(pattern = "\\tBy ") %>% # Remove unnecessary text
    str_remove(pattern = "\\n") %>% # Remove unnecessary text
    str_split(pattern = ", | and ", simplify = TRUE) %>% # Split author names in separate columns
    as_tibble() %>% # Create a tibble
    na_if(y = "") # Remove empty strings with NAs
  return(x)
}
```

I create a second custom function to take the tibble with author names and to collapse it into a vector.


```r
CreateAuthorVectorFromTable <- function(x){
    if(ncol(x) == 1) { # If there is only one author, just pull the column as vector
    x <- 
      x %>% 
      pull(1)
    } else { #If there are multiple authors, then collapse rows into strings
      x <- 
        apply(x, 1, paste, collapse = ", ") %>%  # For-loop for each row
        str_remove_all(pattern = ", NA") # Remove unnecessary text
    }
  return(x)
}
```

I create a third custom function which loops over the elements we extract from the nodes and checks if they are empty. In such a case, the function assigns an `NA` into the vector. We need this function for extracting information on categories. Be aware that we define here two input variables, x and the selector.


```r
CorrectMissingElements <- function(x,selector) {
  x <- 
    sapply(x, # Loop over each element separately
                     function(y){ # Apply the following function
                       y %>%  # Take an element
                         html_nodes(selector) %>% # Extract from the element the node you need
                         html_text(trim = TRUE) # Convert to text
                       }
                     ) %>% # Pass the resulting element to the next function
    sapply(function(y) # Apply the function to the retrieved element
      ifelse(length(y) == 0, NA, y)) # Check if the element is empty. If yes, insert an NA
  
  attributes(x) <- NULL # Remove attributes from the final vector
  return(x)
}
```



Let us now implement the web extraction process!


```r
for (i in 1:10) {
  
  # Receive information on current iteration
  print(paste("Iteration number:",i))
  
  # Create the final URL
  final.url <- paste0(url,i)

  # Parse HTML code
  page <- read_html(final.url)
  
  # Create a file name to save raw HTML source code of each page
  file.name <- paste0("./source_code/",Sys.Date(),"_page_",i,".html")
  
  # Download file if it does not exist yet
  if(!file.exists(file.name)) {
    
    # Download page
    foo <- GET(final.url)
    
    # Extract HTML source code as text
    to.archive <- content(foo, as = "text")
    
    # Archive website
    write(to.archive, file.name)
    
  }
  
  # Retrieve titles
  titles <- 
    page %>% 
    html_elements(css = ".article-title a") %>% 
    html_text(trim = TRUE)

  # Retrieve dates
  dates <- 
    page %>% 
    html_elements(xpath = "//div[@class='post-info']/p/time") %>% 
    html_text(trim = TRUE)
  
  # Retrieve links
  links <- 
    page %>% 
    html_elements(xpath = "//h2[@class='article-title entry-title']/a") %>% 
    html_attr("href")
  
  # Retrieve authors
  authors <- 
    page %>% 
    html_elements(css = ".tease-meta-content") %>% # Must use ".tease-meta-content" instead of ".fn" to retain information that authors belong to one element in the HTML source code
    html_text() %>% 
    CleanAuthorText(firstPage = i) %>% 
    CreateAuthorVectorFromTable()

  # Retrieve categories
  
  ## Retrieve node that all articles share
  category <- 
  page %>% 
  html_elements(xpath = "//div[starts-with(@id,'post-')]") %>% 
  CorrectMissingElements(selector = ".post-info .term")
  

  ####################################
  # Retrieve details from the first article on the first page
  # For details, see explanations in the code above
  
  if(i == 1) {
    
    # First author
    
    authors1 <- 
    page %>% 
    html_element(css = ".vcard") %>% 
    html_text()  %>% 
    CleanAuthorText(firstPage = i) %>% 
    CreateAuthorVectorFromTable()
    
    authors <- c(authors1,authors)
      
    # Retrieve date for the first post
      
    dates1 <- 
      page %>% 
      html_elements(xpath = "//time[@class='updated visually-hidden']") %>% 
      html_text(trim = TRUE) %>% 
      str_remove(pattern = ",.*") %>% 
      str_c(", ", format(Sys.Date(), "%Y"))
    
    dates <- c(dates1,dates)
    
    # Update empty category
    
    category <- c(NA,category)
  } 
  
  ####################################
  dfs[[i]] <- data.frame(title = titles, author = authors, date = dates, category = category, link = links)
  
  Sys.sleep(time = 12 + rnorm(n = 1, mean = 0, sd = 2))
  
}
```

```
## [1] "Iteration number: 1"
## [1] "Iteration number: 2"
## [1] "Iteration number: 3"
## [1] "Iteration number: 4"
## [1] "Iteration number: 5"
## [1] "Iteration number: 6"
## [1] "Iteration number: 7"
## [1] "Iteration number: 8"
## [1] "Iteration number: 9"
## [1] "Iteration number: 10"
```

```r
dat <- as.data.frame(data.table::rbindlist(dfs)) %>% 
  mutate(author = ifelse(test = author == "NA", yes = "A FiveThirtyEight Chat", no = author))  %>% # Replace NAs in our dataset by A FiveThirtyEight Chat
  filter(category != "Politics Podcast" | is.na(category)) # Remove the Politics Podcast
```

We did it! We created our dataset containing information about published articles on the first ten pages FiveThirtyEight website. We can now have a look how the dataset looks like:


```r
head(dat, )
```

```
##                                                                                          title
## 1                                       Why Being ???Anti-Media??? Is Now Part Of The GOP Identity
## 2                            Americans Oppose Many Voting Restrictions ??? But Not Voter ID Laws
## 3      Why The Recent Violence Against Asian Americans May Solidify Their Support Of Democrats
## 4                                    Why Democrats Weren???t Going To Reverse The Result In Iowa
## 5 Police Misconduct Trials Are Rare. Instead, Cities Pay Millions To Settle Misconduct Claims.
## 6                            Why Joe Manchin Is So Willing And Able To Block His Party???s Goals
##                       author          date                      category
## 1            Meredith Conroy April 5, 2021                          <NA>
## 2           Nathaniel Rakich  Apr. 2, 2021                 Voting Rights
## 3             Michael Tesler  Apr. 1, 2021 Asian American Discrimination
## 4           Geoffrey Skelley Mar. 31, 2021                      Congress
## 5 Galen Druke, Laura Bronner Mar. 31, 2021               Police Violence
## 6            Perry Bacon Jr. Mar. 31, 2021                      Congress
##                                                                                                                            link
## 1                                    https://fivethirtyeight.com/features/why-being-anti-media-is-now-part-of-the-gop-identity/
## 2                         https://fivethirtyeight.com/features/americans-oppose-many-voting-restrictions-but-not-voter-id-laws/
## 3 https://fivethirtyeight.com/features/why-the-recent-violence-against-asian-americans-may-solidify-their-support-of-democrats/
## 4                                https://fivethirtyeight.com/features/why-democrats-werent-going-to-reverse-the-result-in-iowa/
## 5 https://fivethirtyeight.com/videos/police-misconduct-trials-are-rare-instead-cities-pay-millions-to-settle-misconduct-claims/
## 6                        https://fivethirtyeight.com/features/why-joe-manchin-is-so-willing-and-able-to-block-his-partys-goals/
```

Theoretically, we can now make some brief descriptive analysis. For instance, let us have a look at who were the top five contributors to the blog in the recent weeks and in which categories those articles were placed.

Authors:


```r
authors <- 
  dat$author %>% 
  str_split(pattern = ", ", simplify = TRUE) %>% 
  na_if(y = "") %>% 
  table() %>% 
  sort(decreasing = TRUE) %>% 
  as_tibble() %>% 
  rename(author = ".") %>% 
  filter(n > 3) 

ggplot(data = authors, mapping = aes(x = reorder(author, n), y = n, fill = n)) + 
  geom_col() +
  scale_fill_continuous(low = "grey", high = "black") +
  xlab(label = "") + 
  ylab(label = "") +
  coord_flip() + 
  theme_minimal()
```

![](web_scraping_with_rvest_files/figure-html/Most active author-1.png)<!-- -->

Categories:


```r
categories <- 
  dat$category %>% 
  table(useNA = NULL) %>% 
  sort(decreasing = TRUE) %>% 
  as_tibble() %>% 
  rename(category = ".") %>% 
  filter(n > 1) 

ggplot(data = categories, mapping = aes(x = reorder(category, n), y = n, fill = n)) + 
  geom_col() +
  scale_fill_continuous(low = "grey", high = "black") +
  xlab(label = "") + 
  ylab(label = "") +
  coord_flip() + 
  theme_minimal()
```

![](web_scraping_with_rvest_files/figure-html/Most frequent categories-1.png)<!-- -->

That's it! You have learned in the above example:

- How to create a directory for your raw HTML source code
- How to save raw HTML source code on your hard drive
- How to use a for-loop to iterate through different pages
- How to implement a Crawl-delay
- How to use XPATH and CSS selectors
- How to combine data frames from different pages into one data frame
- How important it is to pay attention to details such as the HTML code of the first entry of a website


## Credits

The book *Automated Data Collection with R: A Practical Guide to Web Scraping and Text Mining* by Simon Munzert, Christian Rubba, Peter Meibner (2015) was of great help to understand websites from the perspective of web scraping.  
The author of this module also benefited immensely from the course *Big Data Collection and Management in R* provided by Dr. Matt W. Loftis at the ECPR Methods School in 2021.

